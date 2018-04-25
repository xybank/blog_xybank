---
title: 基于练手项目的网络框架搭建
date: 2018-04-25 16:00:00
tags: [Android,rxjava,retrofit,okhttp]
categories: 彭连
description: 基于Rxjava+okhttp+retrofit搭建的网络请求框架，封装了练手项目中的各个表对应的基本网络请求
---

## 开发背景
本框架是基于Rxjava2+retrofit2+okhttp3，可以实现线程之间的快速切换；处理数据简洁易懂，易于进行元素间的变换；可以简单处理大量的嵌套异步回调等。但是使用Retrofit+RxJava+OkHttp完成一次网络请求还是需要写很多代码的，所以肯定是需要再次封装的。该框架的目标是使开发者不用花费精力是理解网络请求的内部实现以及uri等配置，只需要调用对应的业务方法即可完成网络请求，当然更高目标是架构清晰，易于扩展，耦合度低。

## 框架搭建以及介绍
### 1、添加依赖
   app gradle配置：
```
implementation "com.squareup.retrofit2:retrofit:$rootProject.retrofitVersion"
    implementation "com.squareup.retrofit2:converter-gson:$rootProject.retrofitVersion"
    implementation "com.jakewharton.retrofit:retrofit2-rxjava2-adapter:$rootProject.rxjavaAdapter"
    implementation "com.squareup.okhttp3:logging-interceptor:$rootProject.okhttploggingVersion"
    implementation "com.squareup.okhttp3:okhttp:$rootProject.okhttpVersion"
    implementation "io.reactivex.rxjava2:rxjava:$rootProject.rxjavaVersion"
    implementation "io.reactivex.rxjava2:rxandroid:$rootProject.rxandroidVersion"
implementation "com.jakewharton.rxbinding2:rxbinding:$rootProject.rxbindingVersion"
```
Project gradle配置
```
ext {
    retrofitVersion = '2.2.0'
    okhttploggingVersion = '3.4.1'
    okhttpVersion = '3.4.1'
    rxjavaVersion = '2.1.0'
    rxandroidVersion = '2.0.1'
    rxbindingVersion = '2.0.0'
    rxjavaAdapter = '1.0.0'
}
```

### 2、架构展示
![](img/pl/project_structure.png)
### 3、关键模块解析
**RxHelper**：是一个Rxjava的辅助类，主要提供线程切换等方法。
```
public static <T> ObservableTransformer<T, T> rxSchedulerHelper() {//compose处理线程
        return new ObservableTransformer<T, T>() {

            @Override
            public ObservableSource<T> apply(Observable<T> upstream) {
                return upstream.subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread());
            }
        };
    }
    
    
```
**RetrofitCreateHelper**：是框架的核心类，包含okhttp以及retrofit配置以及Api接口的反射调用等。采用gson实现json字符串与实体类之间的自动转化
```
private static final int TIMEOUT_READ = 20;
    private static final int TIMEOUT_CONNECTION = 10;

    private static final HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor()
            .setLevel(HttpLoggingInterceptor.Level.BODY);
    private static CacheInterceptor cacheInterceptor = new CacheInterceptor();
    private static HeaderInterceptor headerInterceptor = new HeaderInterceptor();

    private static OkHttpClient okHttpClient = new OkHttpClient.Builder()
            //SSL证书
            .sslSocketFactory(TrustManager.getUnsafeOkHttpClient())
            .hostnameVerifier(org.apache.http.conn.ssl.SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER)
            .addInterceptor(headerInterceptor)
            .addInterceptor(interceptor)   //打印日志
            .addNetworkInterceptor(cacheInterceptor)  //设置Cache拦截器
            .addInterceptor(cacheInterceptor)
            .cache(HttpCache.getCache())
            //time out
            .connectTimeout(TIMEOUT_CONNECTION, TimeUnit.SECONDS)
            .readTimeout(TIMEOUT_READ, TimeUnit.SECONDS)
            .writeTimeout(TIMEOUT_READ, TimeUnit.SECONDS)
            //失败重连
            .retryOnConnectionFailure(true)
            .build();

    public static <T> T createApi(Class<T> clazz, String url) {
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(url)
                .client(okHttpClient)
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        return retrofit.create(clazz);
    }
```

**RxManager**：提供Rxjava中订阅者的注册订阅与取消订阅
```
public class RxManager {
    private CompositeDisposable mCompositeDisposable = new CompositeDisposable();// 管理订阅者者

    public void register(Disposable d) {
        mCompositeDisposable.add(d);
    }

    public void unSubscribe() {
        mCompositeDisposable.dispose();// 取消订阅
    }
}
```

**Okhttp包**：主要就是定义一些okhttp的拦截器，包含缓存拦截器和https认证等，其中由于采用的是bomb的后台，需要添加一个header拦截器，自定义一些header。okhttp采用的是责任链拦截器的模式，采用的是类似U型的调用，后一个拦截器执行失败则结束，成功则将结果返回给前一个拦截器，以此类推，直至首个拦截器，则完成一次完整的网络请求。
```
public class TrustManager {

    public static SSLSocketFactory getUnsafeOkHttpClient() {
        try {
            // Create a trust manager that does not validate certificate chains
            final X509TrustManager[] trustAllCerts = new X509TrustManager[]{new X509TrustManager() {
                @Override
                public void checkClientTrusted(
                        X509Certificate[] chain,
                        String authType) throws CertificateException {
                }

                @Override
                public void checkServerTrusted(
                        X509Certificate[] chain,
                        String authType) throws CertificateException {
                }

                @Override
                public X509Certificate[] getAcceptedIssuers() {
                    return new X509Certificate[0];
                }
            }};

            // Install the all-trusting trust manager
            final SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, trustAllCerts,
                    new java.security.SecureRandom());
            // Create an ssl socket factory with our all-trusting manager
            final SSLSocketFactory sslSocketFactory = sslContext
                    .getSocketFactory();


            return sslSocketFactory;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
}
```
**Api包**：定义表对应的api接口类，接口类中定义各个业务方法
```
public interface UserApi{
    String HOST = "https://api.bmob.cn/1/classes/";
    String TABLE_URL = "_User";

    @GET(TABLE_URL + "/{objectId}")
    Observable<UserBean> getData(@Path("objectId") String objectId);

    @GET(TABLE_URL)
    Observable<ListObjectBean<UserBean>> getListData();

    @POST(TABLE_URL)
    Observable<UserBean> postData(@Body UserBean userBean);

    @PUT(TABLE_URL + "/{objectId}")
    Observable<UserBean> putData(@Path("objectId") String objectId, @Body UserBean userBean);

    @DELETE(TABLE_URL + "/{objectId}")
    Observable<UserBean> deleteData(@Path("objectId") String objectId);
}
```
**Bean包**：定义与表字段对应的实体类，其中ListObjectBean是专门处理list数据的，由于采用gson自动转化，所以需要在List<T>包裹一层；BaseBean是实体基类，定义一些公共属性。
```
public class ListObjectBean<T> implements Serializable{
    private List<T> results;

    public List<T> getResults() {
        return results;
    }

    public void setResults(List<T> results) {
        this.results = results;
    }
}
```
## 使用说明
基于的是bomb的云服务器后台，bomb后台提供的restful接口方式是基于表形式的，后台提供的服务接口有限，目前本框架暂时只支持以下五种基于表形式的请求接口，有其它需求后期再考虑扩展。（已下以rentout_info表为例，由于只是测试使用，如果是mvp架构，实际使用可以将Observable的获取放入model层）

### 1、Get方式（获取单条数据）

   入参 :   objectId（表objectId）
   
   回调 :   成功回调 ---> RentOutInfoBean实体=>与表字段对应的实体类
            失败回调---> Throwable异常对象
            
   举例 :   
   ```
   rxManager.register(RetrofitCreateHelper.createApi(RentOutApi.class, RentOutApi.HOST).getData(objectId)
                        .compose(RxHelper.<RentOutInfoBean>rxSchedulerHelper()).subscribe(new Consumer<RentOutInfoBean>() {
                            @Override
                            public void accept(RentOutInfoBean result) throws Exception {
                                if (result != null) {
                                    StringBuilder sb = new StringBuilder();
                                    sb.append(result.getObjectId() + ":" + result.getCity() + ":" + result.getContent() + "\n");
                                    tvResult.setText(sb.toString());
                                }
                            }
                        }, new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable throwable) throws Exception {
                                ToastUtil.showLong(MainActivity.this, throwable.toString());
                                tvResult.setText(throwable.toString());
                            }
                        }));
   ```
### 2、Get方式（获取多条数据）

   入参 :   无
   
   回调 :   成功回调 --->ListObjectBean<RentOutInfoBean>=>含有List<RentOutInfoBean>
   失败回调---> Throwable异常对象
            
   举例 :  
   ```
    rxManager.register(RetrofitCreateHelper.createApi(RentOutApi.class, RentOutApi.HOST)
                        .getListData().compose(RxHelper.<ListObjectBean<RentOutInfoBean>>rxSchedulerHelper())
                        .subscribe(new Consumer<ListObjectBean<RentOutInfoBean>>() {
                            @Override
                            public void accept(ListObjectBean<RentOutInfoBean> result) throws Exception {
                                if (result != null) {
                                    List<RentOutInfoBean> rentOutInfoBeans = result.getResults();
                                    if (rentOutInfoBeans != null && rentOutInfoBeans.size() > 0) {
                                        StringBuilder sb = new StringBuilder();
                                        for (RentOutInfoBean rentOutInfoBean : rentOutInfoBeans) {
                                            sb.append(rentOutInfoBean.getObjectId() + ":" + rentOutInfoBean.getCity() + ":" + rentOutInfoBean.getContent() + "\n");
                                        }
                                        tvResult.setText(sb.toString());
                                    }
                                }
                            }
                        }, new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable throwable) throws Exception {
                                ToastUtil.showLong(MainActivity.this, throwable.toString());
                                tvResult.setText(throwable.toString());
                            }
                        }));
   ```
### 3、Post方式（添加单条数据）
入参 :   RentOutInfoBean实体

回调 :   成功回调 --->RentOutInfoBean实体=>只包含objectId和createdAt字段    失败回调---> Throwable异常对象
            
 举例 : 
 ```
  RentOutInfoBean testBean = new RentOutInfoBean();
                testBean.setLastTime("131241341");
                testBean.setContent("13413411314ssa3");
                testBean.setCity("哈尔滨");
                rxManager.register(RetrofitCreateHelper.createApi(RentOutApi.class, RentOutApi.HOST).postData(testBean)
                        .compose(RxHelper.<RentOutInfoBean>rxSchedulerHelper()).subscribe(new Consumer<RentOutInfoBean>() {
                            @Override
                            public void accept(RentOutInfoBean result) throws Exception {
                                if (result != null) {
                                    tvResult.setText(result.getObjectId());
                                }
                            }
                        }, new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable throwable) throws Exception {
                                ToastUtil.showLong(MainActivity.this, throwable.toString());
                                tvResult.setText(throwable.toString());
                            }
                        }));
 ```
 ### 4、Put方式（修改单条数据）
   入参 :   objectId  
   
   RentOutInfoBean实体=>包含要修改的字段
   回调 :   成功回调 --->RentOutInfoBean实体=>只包含updatedAt字段失败回调---> Throwable异常对象
   
   举例 : 
   ```
    final RentOutInfoBean testBean1 = new RentOutInfoBean();
                testBean1.setCity("上海");
                rxManager.register(RetrofitCreateHelper.createApi(RentOutApi.class, RentOutApi.HOST).putData(objectId, testBean1)
                        .compose(RxHelper.<RentOutInfoBean>rxSchedulerHelper()).subscribe(new Consumer<RentOutInfoBean>() {
                            @Override
                            public void accept(RentOutInfoBean result) throws Exception {
                                if (result != null) {
                                    tvResult.setText(result.getUpdatedAt());
                                }
                            }
                        }, new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable throwable) throws Exception {
                                ToastUtil.showLong(MainActivity.this, throwable.toString());
                                tvResult.setText(throwable.toString());
                            }
                        }));
   ```
### 5、Delete方式（删除单条数据）

   入参 :   objectId
   
   回调 :   成功回调 --->RentOutInfoBean实体=>只包含msg字段 值为”ok”失败回调---> Throwable异常对象
            
   举例 :   
   ```
   rxManager.register(RetrofitCreateHelper.createApi(RentOutApi.class, RentOutApi.HOST).deleteData(objectId)
                        .compose(RxHelper.<RentOutInfoBean>rxSchedulerHelper()).subscribe(new Consumer<RentOutInfoBean>() {
                            @Override
                            public void accept(RentOutInfoBean result) throws Exception {
                                if (result != null) {
                                    tvResult.setText(result.getMsg());
                                }
                            }
                        }, new Consumer<Throwable>() {
                            @Override
                            public void accept(Throwable throwable) throws Exception {
                                ToastUtil.showLong(MainActivity.this, throwable.toString());
                                tvResult.setText(throwable.toString());
                            }
                        }));
   ```
## 总结
由于目前还是处于第一版，再加上bomb后台的restful接口服务限制，只提供了一些基础通用的请求方式，还有许多需要补充与完善的地方，比如文件上传、模糊搜索等一些高级请求。 

参考链接：
https://blog.csdn.net/qiang_xi/article/details/53959437

Demo链接：
https://github.com/penglian/TestRestful
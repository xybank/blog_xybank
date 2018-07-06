---
title: CombineBitmap开源项目源码解析
date: 2018-7/6 12:00:00
tags: [Android,Bitmap,WeChat,DingDing]
categories: 彭连
description: 近来在github上看到一个比较火的自定义View，仿微信以及钉钉群组头像生成，便好好学习了一番
---

## 项目简述

啥也不说，先看一波效果图：

![](/img/pl/combine_bitmap_effect.png)

别看这只是一个小小的view，由于涉及Bitmap，所以对性能的处理也是必不可少，还得压缩组合bitmap展现出好的视觉效果；再者还得考虑网络加载还是Resource加载，以及缓存处理等，实现起来还是得费不少功夫的。
### 结构简述
项目结构如下：
![](/img/pl/combine_bitmap_caterlog.png)
cache包下主要是内存缓存（LruCache）与磁盘缓存（DiskLruCache）的两个辅助类

handle包没啥东西，只是定了一个接口以及Handle的子类

layout包包含了根据特定的规则对bitmap进行组合的接口以及两个实现类DingLayoutManager(仿钉钉)以及WeChatLayoutManager(仿微信)

helper包包含的东西就比较杂，不过根据类名也比较好辨认，比如BitMapLoader(bitmap加载类)、Builder(builder的实体构造模式)、ThreadPool(线程池)还有一些CompressHelper(压缩)等

最外层的CombineBitmap一看就知道是入口类。
具体的调用方式如下：
```
private void loadWechatBitmap(ImageView imageView, int count) {
        CombineBitmap.init(MainActivity.this)
                .setLayoutManager(new WechatLayoutManager())
                .setSize(180)
                .setGap(3)
                .setGapColor(Color.parseColor("#E8E8E8"))
                .setUrls(getUrls(count))
                .setImageView(imageView)
                .build();
    }
```
### 设计思路
从调用方式可知，通过builder的方式构造实体，根据实体中url属性是否为空来判断bitmap加载来源。如果url为空，则是从Resource加载，获取到bitmap后先进行采样率压缩然后根据特定的规则对bitmap数组进行组合并返回一个最终的bitmap，执行imageView.setBitmap（bitmap）就ok了。反之，采用的是三级缓存加载，加载规则是先从内存中加载，有则直接返回，没有就从磁盘加载，有则返回并且将其加入到内存缓存中，没有的话就从网络加载（HttpUrlConnecttion），加载完将其存入磁盘缓存中，并且从缓存中读取bitmap，这样就将其存入内存缓存了。获取到bitmap之后，操作就跟resource一样了，就是压缩组合了。当然其中还有运用到线程池、队列管理等。

## 关键模块分析
### Builder实体类构建：
如下图：
![](/img/pl/combine_bitmap_builder.png)
builder实体相当于一个配置类，是与调用者直接关联的，这个类的设计是贯穿于整个项目结构的，通过这个配置类使用者就能了解这个框架或者开源库的大致功能以及使用，通过配置属性就能实现对应的功能。如上图，定义的属性都是项目设计过程中必不可少的并且让用户自定义的（都有注释我就不一一解释了）。

### 缓存模块
主要是LruCache以及DiskLruCache，项目中LruCache采用的是v4包中，LruCacheHelper只是对其做了初始化配置以及get与add方法；磁盘缓存采用的是‘com.jakewharton:disklrucache’的依赖库，DiskCacheHelper只是对其进行初始化配置等。

### Bitmap组合模块
定义了一个接口以及两个实现类：
DingLayoutManager类：
```
   /**
     * 
     * @param size  最终生成bitmap的尺寸
     * @param subSize  单个bitmap的尺寸
     * @param gap 每个小bitmap之间的距离
     * @param gapColor  间距的颜色
     * @param bitmaps  bitmap数组
     * @return
     */  
  public Bitmap combineBitmap(int size, int subSize, int gap, int gapColor, Bitmap[] bitmaps) {
        Bitmap result = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(result);
        if (gapColor == 0) {
            gapColor = Color.WHITE;
        }
        canvas.drawColor(gapColor);

        int count = bitmaps.length;
        Bitmap subBitmap;

        int[][] dxy = {{0, 0}, {1, 0}, {1, 1}, {0, 1}};

        for (int i = 0; i < count; i++) {
            if (bitmaps[i] == null) {
                continue;
            }
            subBitmap = Bitmap.createScaledBitmap(bitmaps[i], size, size, true);
            if (count == 2 || (count == 3 && i == 0)) {
                subBitmap = Bitmap.createBitmap(subBitmap, (size + gap) / 4, 0, (size - gap) / 2, size);
            } else if ((count == 3 && (i == 1 || i == 2)) || count == 4) {
                subBitmap = Bitmap.createBitmap(subBitmap, (size + gap) / 4, (size + gap) / 4, (size - gap) / 2, (size - gap) / 2);
            }

            int dx = dxy[i][0];
            int dy = dxy[i][1];

            canvas.drawBitmap(subBitmap, dx * (size + gap) / 2.0f, dy * (size + gap) / 2.0f, null);
        }
        return result;
    }
```

WeChatLayoutManager类:
```
   /**
     * 
     * @param size  最终生成bitmap的尺寸
     * @param subSize  单个bitmap的尺寸
     * @param gap 每个小bitmap之间的距离
     * @param gapColor  间距的颜色
     * @param bitmaps  bitmap数组
     * @return
     */
 public Bitmap combineBitmap(int size, int subSize, int gap, int gapColor, Bitmap[] bitmaps) {
        Bitmap result = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(result);
        if (gapColor == 0) {
            gapColor = Color.WHITE;
        }
        canvas.drawColor(gapColor);

        int count = bitmaps.length;
        Bitmap subBitmap;

        for (int i = 0; i < count; i++) {
            if (bitmaps[i] == null) {
                continue;
            }
            subBitmap = Bitmap.createScaledBitmap(bitmaps[i], subSize, subSize, true);

            float x = 0;
            float y = 0;

            if (count == 2) {
                x = gap + i * (subSize + gap);
                y = (size - subSize) / 2.0f;
            } else if (count == 3) {
                if (i == 0) {
                    x = (size - subSize) / 2.0f;
                    y = gap;
                } else {
                    x = gap + (i - 1) * (subSize + gap);
                    y = subSize + 2 * gap;
                }
            } else if (count == 4) {
                x = gap + (i % 2) * (subSize + gap);
                if (i < 2) {
                    y = gap;
                } else {
                    y = subSize + 2 * gap;
                }
            } else if (count == 5) {
                if (i == 0) {
                    x = y = (size - 2 * subSize - gap) / 2.0f;
                } else if (i == 1) {
                    x = (size + gap) / 2.0f;
                    y = (size - 2 * subSize - gap) / 2.0f;
                } else if (i > 1) {
                    x = gap + (i - 2) * (subSize + gap);
                    y = (size + gap) / 2.0f;
                }
            } else if (count == 6) {
                x = gap + (i % 3) * (subSize + gap);
                if (i < 3) {
                    y = (size - 2 * subSize - gap) / 2.0f;
                } else {
                    y = (size + gap) / 2.0f;
                }
            } else if (count == 7) {
                if (i == 0) {
                    x = (size - subSize) / 2.0f;
                    y = gap;
                } else if (i < 4) {
                    x = gap + (i - 1) * (subSize + gap);
                    y = subSize + 2 * gap;
                } else {
                    x = gap + (i - 4) * (subSize + gap);
                    y = gap + 2 * (subSize + gap);
                }
            } else if (count == 8) {
                if (i == 0) {
                    x = (size - 2 * subSize - gap) / 2.0f;
                    y = gap;
                } else if (i == 1) {
                    x = (size + gap) / 2.0f;
                    y = gap;
                } else if (i < 5) {
                    x = gap + (i - 2) * (subSize + gap);
                    y = subSize + 2 * gap;
                } else {
                    x = gap + (i - 5) * (subSize + gap);
                    y = gap + 2 * (subSize + gap);
                }
            } else if (count == 9) {
                x = gap + (i % 3) * (subSize + gap);
                if (i < 3) {
                    y = gap;
                } else if (i < 6) {
                    y = subSize + 2 * gap;
                } else {
                    y = gap + 2 * (subSize + gap);
                }
            }

            canvas.drawBitmap(subBitmap, x, y, null);
        }
        return result;
    }
```
这是这个开源库最关键的部分，就是组合的规则，相信是作者花了不少时间研究的，具体的逻辑可以直接看代码，不是很难。

### 压缩
压缩采用的是采样率压缩，可以先获取宽高，计算采样率，在进行采样率压缩，这样就可以减少内存消耗.
```
public Bitmap compressDescriptor(FileDescriptor fd, int reqWidth, int reqHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeFileDescriptor(fd, null, options);
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeFileDescriptor(fd, null, options);
    }
```
### BitMapLoader
这是三级缓存加载类，采用的规则就是先从内存加载，再从磁盘最后是网络，是这个开源库中逻辑最复杂最缜密的地方，还采用了线程池以及自定义队列来管理任务。
```
public class BitmapLoader {
    private static String TAG = BitmapLoader.class.getSimpleName();
    private static final int BUFFER_SIZE = 8 * 1024;

    private LruCacheHelper lruCacheHelper;
    private DiskLruCache mDiskLruCache;
    private CompressHelper compressHelper;

    private volatile static BitmapLoader manager;

    public static BitmapLoader getInstance(Context context) {
        if (manager == null) {
            synchronized (BitmapLoader.class) {
                if (manager == null) {
                    manager = new BitmapLoader(context);
                }
            }
        }
        return manager;
    }

    // 存储线程池中的任务
    private Map<String, Runnable> doingTasks;
    // 存储暂时不能进入线程池的任务
    private Map<String, List<Runnable>> undoTasks;


    private BitmapLoader(Context context) {
        mDiskLruCache = new DiskLruCacheHelper(context).mDiskLruCache;
        lruCacheHelper = new LruCacheHelper();
        compressHelper = CompressHelper.getInstance();

        doingTasks = new HashMap<>();
        undoTasks = new HashMap<>();
    }


    public void asyncLoad(final int index, final String url, final int reqWidth, final int reqHeight, final Handler handler) {
        Runnable task = new Runnable() {
            @Override
            public void run() {
                Bitmap bitmap = loadBitmap(url, reqWidth, reqHeight);
                if (bitmap != null) {
                    handler.obtainMessage(1, index, -1, bitmap).sendToTarget();
                } else {
                    handler.obtainMessage(2, index, -1, null).sendToTarget();
                }
            }
        };

        if (collectUndoTasks(url, task)) {
            return;
        }

        ThreadPool.getInstance().getThreadPoolExecutor().execute(task);
    }

    private Bitmap loadBitmap(String url, int reqWidth, int reqHeight) {

        // 尝试从内存缓存中读取
        String key = Utils.hashKeyFormUrl(url);
        Bitmap bitmap = lruCacheHelper.getBitmapFromMemCache(key);
        if (bitmap != null) {
            Log.e(TAG, "load from memory:" + url);
            return bitmap;
        }

        try {
            // 尝试从磁盘缓存中读取
            bitmap = loadBitmapFromDiskCache(url, reqWidth, reqHeight);
            if (bitmap != null) {
                Log.e(TAG, "load from disk:" + url);
                return bitmap;
            }
            // 尝试下载
            bitmap = loadBitmapFromHttp(url, reqWidth, reqHeight);
            if (bitmap != null) {
                Log.e(TAG, "load from http:" + url);
                return bitmap;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

    private Bitmap loadBitmapFromHttp(String url, int reqWidth, int reqHeight)
            throws IOException {
        String key = Utils.hashKeyFormUrl(url);
        //判断当前mDiskLruCache是否已经在操作key，为null说明没有，不为null则直接返回
        DiskLruCache.Editor editor = mDiskLruCache.edit(key);
        if (editor != null) {
            OutputStream outputStream = editor.newOutputStream(0);
            if (downloadUrlToStream(url, outputStream)) {
                editor.commit();
            } else {
                editor.abort();
            }
            mDiskLruCache.flush();

            executeUndoTasks(url);
        }
        return loadBitmapFromDiskCache(url, reqWidth, reqHeight);
    }

    private boolean downloadUrlToStream(String urlString,
                                        OutputStream outputStream) {
        HttpURLConnection urlConnection = null;
        BufferedOutputStream out = null;
        BufferedInputStream in = null;

        try {
            final URL url = new URL(urlString);
            urlConnection = (HttpURLConnection) url.openConnection();
            in = new BufferedInputStream(urlConnection.getInputStream(), BUFFER_SIZE);
            out = new BufferedOutputStream(outputStream, BUFFER_SIZE);

            int b;
            while ((b = in.read()) != -1) {
                out.write(b);
            }
            return true;
        } catch (IOException e) {
            Log.e(TAG, "downloadBitmap failed." + e);
        } finally {
            if (urlConnection != null) {
                urlConnection.disconnect();
            }
            Utils.close(out);
            Utils.close(in);
        }
        return false;
    }

    private Bitmap loadBitmapFromDiskCache(String url, int reqWidth,
                                           int reqHeight) throws IOException {
        Bitmap bitmap = null;
        String key = Utils.hashKeyFormUrl(url);
        DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);
        if (snapShot != null) {
            FileInputStream fileInputStream = (FileInputStream) snapShot.getInputStream(0);
            FileDescriptor fileDescriptor = fileInputStream.getFD();
            bitmap = compressHelper.compressDescriptor(fileDescriptor, reqWidth, reqHeight);
            if (bitmap != null) {
                lruCacheHelper.addBitmapToMemoryCache(key, bitmap);
            }
        }

        return bitmap;
    }

    private boolean collectUndoTasks(String url, Runnable task) {
        String key = Utils.hashKeyFormUrl(url);

        if (lruCacheHelper.getBitmapFromMemCache(key) != null) {
            return false;
        }

        DiskLruCache.Snapshot snapShot = null;
        try {
            snapShot = mDiskLruCache.get(key);
        } catch (IOException e) {
            e.printStackTrace();
        }

        if (snapShot != null) {
            return false;
        }

        // 如果当前url下载操作过程的磁盘缓存的Editor未结束，又来了一个新的url，则不能正常生成新Editor
        // 则将新url对应的任务先保存起来
        if (doingTasks.containsKey(key)) {
            if (undoTasks.containsKey(key))  {
                List<Runnable> tasks = undoTasks.get(key);
                tasks.add(task);
                undoTasks.put(key, tasks);
            } else {
                List<Runnable> tasks = new ArrayList<>();
                tasks.add(task);
                undoTasks.put(key, tasks);
            }
            return true;
        }

        doingTasks.put(key, task);
        return false;
    }

    private void executeUndoTasks(String url) {
        String key = Utils.hashKeyFormUrl(url);
        // 检查undoTasks中是否有要执行的任务
        if (undoTasks.containsKey(key)) {
            for (Runnable task : undoTasks.get(key)) {
                ThreadPool.getInstance().getThreadPoolExecutor().execute(task);
            }
            undoTasks.remove(key);
        }
        // 从doingTasks中移除已经执行完的任务
        doingTasks.remove(key);
    }
}
```
asyncLoad为入口方法，loadBimtap（）方法放入线程中执行，获取规则就是如上所述，先从内存再从磁盘再从网络，最后利用线程池去执行runnable。

这里重点说下两个map队列，分别是存储doingTask(正在执行的任务)以及undoTask(暂未执行的任务)。collectUndoTasks（）在执行runnable之前执行，如果从内存或者磁盘中根据key（url经过md5转化成特定位数的数）获取到对应的对象，直接返回false，即执行runnable;否则，判断doingTask是否包含对应的key,包含的话就将其存入undoTask，返回true不执行runnable，不包含就将其存入doingTask中，返回false去执行runnable。

在执行完网络加载操作loadBitmapFromHttp（key）方法，需要再次检测两个队列，undoTask如果包含key，就去执行对应的task,执行完从队列中移除掉key;还有就是把doingTask中的key给移除掉。

队列可以很好的管理各个对象的状态，方便各个地方获取使用，其add与remove操作得把握控制得当，随意的添加或者移除会产生事倍功半的效果。



## 总结
以上是我个人对于此次CombineBitmap开源库的学习总结，这个库耦合性低，涉及的知识面也广，实现效果也不错，建议大家去fork学习使用。

CombineBitmap项目地址：https://github.com/Othershe/CombineBitmap

---
title: 浅谈Serializable与Parcelable
date: 2018-06-06 12:00:00
tags: [Android,Serializable,Parcelable]
categories: 彭连
description: 简要介绍序列化以及反序列化，以及两种实现方式Serializable和Parcelable还有二者的区别
---

## 序列化与反序列化的由来
### 序列化
序列化 (Serialization)将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。即对象转换为字节序列。
### 反序列化
顾名思义，就是从存储区上反序列化对象的状态，重新*创建*对象。即字节序列重新转换为对象。
### 两者的由来
在日常java编程中，网络传输以及文件存储等操作是必不可少的。序列化能保证网络传输以及文件存储中对象的一致性与持久性，不会发生错乱；反序列化能重新创建一个相同的对象，所以说序列化与反序列化也是java编程中必不可少的。
## 如何实现序列化
### Serializable接口
*Serializable*是java提供的一个序列化的接口，它是一个空接口，为对象提供标准的序列化与反序列化操作。实体定义如下
```
public class FlyPig implements Serializable {  
    private static final long serialVersionUID = 1L;  
    private static String AGE = "269";  
    private String name;  
    private String color;  
    ...
}
```
*Serializable*来实现序列化非常简单，只需要实现该接口并且定义一个*serialVersionUID*就行，其余的工作都交给系统自动化进行了。如何进行对象的序列化与反序列化也非常简单，只需要*ObjectOutputStream*与*ObjectInputStream*即可轻松实现，如下
```
//序列化
  FlyPig flyPig = new FlyPig("black","naruto","0000");  
  ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("d:/flyPig.txt")));  
  oos.writeObject(flyPig);  
  oos.close();  
//反序列化
  ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("d:/flyPig.txt")));  
        FlyPig person = (FlyPig) ois.readObject();
```
上面提到要指定*serialVersionUID*（通常制定为1L即可，系统会自动生成hash值），不指定能否进行序列化呢？答案是yes。*serialVersionUID*只是一个标识，用来辅助序列化与反序列化过程的，原则上序列化后的*serialVersionUID*只有和当前类的*serialVersionUID*相同才能被反序列化。原理就是序列化是将该字段写入文件或者其它中介，在反序列时先比对该字段，相同则可以反序列化成功，不同则抛异常*InvalidClassException*。

不指定*serialVersionUID*，当类新增或者删除属性时，系统会重新计算该类的hash并赋给*serialVersionUID*，这时候就会不一致导致反序列化失败，程序出现crash。手动指定就可以避免这种情况。当然如果类结构改变的话，尽管*serialVersionUID*验证通过，还是会序列化失败，因为类结构发生改变，无法从老版本的数据中还原出一个新的类结构对象
### Parcelable接口
*Parcelable*是Android新提供的序列化方式，相对于*Serializable*，它的实现稍微复杂一点。
```
public class ParcelableType implements Parcelable {  
    int age;  
    String name;  
    boolean isGood;  
    boolean complete;  
    private String[] ids;  
    private OrderInfoBean bean;  
    private ArrayList<OrderInfoBean> listBeans;  
  
    /** 
     * 默认构造方法 
     */  
    public ParcelableType() {  
        // TODO Auto-generated constructor stub  
    }  
  
    public ParcelableType(Parcel in) {  
        readFromParcel(in);  
    }  
  
    /*** 
     * 默认实现 
     */  
    @Override  
    public int describeContents() {  
        // TODO Auto-generated method stub  
        return 0;  
    }  
  
    @Override  
    public void writeToParcel(Parcel dest, int flags) {  
        dest.writeInt(age);  
        dest.writeString(name);  
        dest.writeInt(isGood ? 1 : 0);  
        dest.writeInt(complete ? 1 : 0);  
        if (ids != null) {  
            dest.writeInt(ids.length);  
        } else {  
            dest.writeInt(0);  
        }  
        dest.writeStringArray(ids);  
        dest.writeParcelable(bean, flags);  
        dest.writeList(listBeans);  
  
    }  
  
    @SuppressWarnings("unchecked")  
    private void readFromParcel(Parcel in) {  
        age = in.readInt();  
        name = in.readString();  
        isGood = (in.readInt() == 1) ? true : false;  
        complete = (in.readInt() == 1) ? true : false; 
        int length = in.readInt();  
        ids = new String[length];  
        in.readStringArray(ids);  
        bean = in.readParcelable(OrderInfoBean.class.getClassLoader());  
        listBeans = in.readArrayList(OrderInfoBean.class.getClassLoader());  
  
    }  
  
    public static final Parcelable.Creator<ParcelableType> CREATOR = new Parcelable.Creator<ParcelableType>() {  
        public ParcelableType createFromParcel(Parcel in) {  
            return new ParcelableType(in);  
        }  
  
        public ParcelableType[] newArray(int size) {  
            return new ParcelableType[size];  
        }  
    };  
 ....
}  
```
*Parcel*内部包装了可序列化的数据，可在Binder中自由的传递。从以上代码可以看出，序列化过程包含序列化、反序列化以及功能描述。序列化由*writeToParcel*方法完成，最终通过Parcel的一系列write方法完成；反序列化通过*CREATOR*来完成，其内部标识了如何创建序列化对象以及数组，并通过Parcel的一系列read方法来完成反序列化的过程；内容描述功能由*describeContents*方法完成，几乎所有的情况这个方法都返回0，只有当前对象存在文件描述符时，此方法返回1。
## 总结
对于Android开发来说，*Parcelable*与*Serializable*都能实现序列化并且都可用于Intent之间的数据传递，那么二者该如何选取呢？*Serializable*是java的序列化接口，其使用起来简单但是开销很大，序列化与反序列化过程需大量的IO操作。而*Parcelable*是Android的序列化方式，因此更适用于Android平台，它的缺点就是使用起来麻烦一点，但是效率更高，是Android推荐的序列化方法，我们应该首选*Parcelable*。*Parcelable*主要用在内存序列化上，通过*Parcelable*将对象序列化到存储设备或者将对象序列化后通过网络传输也都是可以的，但是这个过程会稍显复杂，因此在这两种情况建议大家使用*Serializable*。


参考书籍：《Android开发艺术探索》


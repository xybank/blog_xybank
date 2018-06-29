---
title: 对于获取设备标识的简述
date: 2018-06-29 12:00:00
tags: [Android,deviceId,SharePreference,File]
categories: 彭连
description: 介绍Android获取设备标识常见问题以及解决方案。
---

## 常见的几种方式
### Imei
IMEI 国际移动设备身份码 目前GSM/WCDMA/LTE手机终端需要使用IMEI号码，在单卡工程中一个手机号对应一个IMEI号，双卡手机则会对应两个IMEI号，一张是手机卡对应一个。
#### Imei获取
```
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>

TelephonyManager telephonyManager = (TelephonyManager)context.getSystemService(context.TELEPHONY_SERVICE);  
String imei = telephonyManager.getDeviceId();
```
#### Imei弊端
由以上可以看出使用IMEI来作为Android的设备唯一标识符存在一定的弊端，如果用户禁用掉相关权限，那么对于以上获取参数的代码。则会直接报错，不会得到我们想要的内容。

### Mac地址
Mac 指的就是我们设备网卡的唯一设别码，该码全球唯一，一般称为物理地址，硬件地址用来定义设备的位置
#### Mac地址获取
```
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/> 
private void getMacAddress(){
    String macAddress = null;
    StringBuffer buf = new StringBuffer();
    NetworkInterface networkInterface = null;
    try {
        networkInterface = NetworkInterface.getByName("eth1");
        if (networkInterface == null) {
            networkInterface = NetworkInterface.getByName("wlan0");
        }
        if (networkInterface == null) {
            return "02:00:00:00:00:02";
        }
        byte[] addr = networkInterface.getHardwareAddress();
        for (byte b : addr) {
            buf.append(String.format("%02X:", b));
        }
        if (buf.length() > 0) {
            buf.deleteCharAt(buf.length() - 1);
        }
            macAddress = buf.toString();
        } catch (SocketException e) {
            e.printStackTrace();
            return "02:00:00:00:00:02";
        }
        return macAddress;
}
```
#### Mac地址弊端
* 如果使用Mac地址最重要的一点就是手机必须具有上网功能

* 在Android6.0以后google为了运行时权限对geMacAddress();作出修改通过该方法得到的mac地址永远是一样的， 但是可以其他途径获取。

### Android Id
```
String ANDROID_ID = Settings.System.getString(getContentResolver(), Settings.System.ANDROID_ID); 
```
#### Android ID弊端
* 手机恢复出厂设置以后该值会发生变化

* 在国内Android定制的大环境下，有些设备是不会返回ANDROID_ID的

### Serial Number（设备序列号）
```
String SerialNumber = android.os.Build.SERIAL;
```
#### Serial Number弊端
获取序列号不需要权限，但是有一定的局限性，在有些手机上会出现垃圾数据，比如红米手机返回的就是连续的非随机数

## 解决方案
比较以上几种方法。如果只是考虑单独使用，那么在不同程度上在用户使用的情况下都会出现无法生成或者生成无效设备唯一标识符的情况。所以我在开发中采用混合使用的方式，同时结合SD卡以及sharepreference进行本地持久化处理。

### 设计简述
在开发中通过结合 device_id 、 MacAddress 以及 随机生成的 UUID 进行生成设备唯一标识符。 然后通过把生成的唯一标识符写到SD卡中一个隐藏目录中（为什么是写到影藏文件中呢，主要是避免用户看见然后手动删除）。同时会在相应App sharepreference 进行保存一份。

用户首先获取deviceId，获取到的话就直接返回，获取不到的话就从sharepreference中获取，获取到就返回，获取不到就从File中获取，获取到就返回并且保存到SharePreference中，获取不到就主动构建并且保存到隐藏文件与SharePreference中，方便下次调用

### 代码如下
```
            TelephonyManager tm = (TelephonyManager) mActivity
                    .getSystemService(Context.TELEPHONY_SERVICE);
            String myIMEI = tm.getDeviceId();
            // 对IMEI号进行加密
            if (StringUtils.isEmpty(myIMEI)) {
               //先从sp中获取，获取不到的话再从file中获取，获取到的话进行sp存储；file获取不到的话重新生成,并且做保存操作
               myIMEI=DeviceIdUtils.getDeviceId(mActivity);
            }
            
```

DeviceIdUtils类如下：
```
public class DeviceIdUtils {
    //保存文件的路径
    private static final String CACHE_IMAGE_DIR = "aray/cache/devices";
    //保存的文件 采用隐藏文件的形式进行保存
    private static final String DEVICES_FILE_NAME = ".DEVICES";

    /**
     * 获取设备唯一标识符
     *
     * @param context
     * @return
     */
    public static String getDeviceId(Context context) {
        String deviceId = SpUtils.getString(context, SpConstant.SP_DEVICES_ID, "");
        if (!StringUtils.isEmpty(deviceId)) {
            //生成deviceId并且保存到file以及sp
            return deviceId;
        }
        //读取保存的在sd卡中的唯一标识符
        deviceId = readDeviceID(context);
        //判断是否已经生成过,
        if (!StringUtils.isEmpty(deviceId)) {
            //保存到sp中
            SpUtils.putString(context, SpConstant.SP_DEVICES_ID, deviceId);
            return deviceId;
        }
        //用于生成最终的唯一标识符
        StringBuffer s = new StringBuffer();
        try {
            //获取设备的MACAddress地址 去掉中间相隔的冒号
            deviceId = getLocalMac(context);
            if(!StringUtils.isEmpty(deviceId)){
                deviceId = deviceId.replace(":", "");
                s.append(deviceId);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        //UUID生成随机字符串
        UUID uuid = UUID.randomUUID();
        deviceId = uuid.toString().replace("-", "");
        s.append(deviceId);

        //为了统一格式对设备的唯一标识进行md5加密 最终生成32位字符串
        String md5 = getMD5(s.toString(), false);
        if (s.length() > 0) {
            //持久化操作, 进行保存到SD卡中
            saveDeviceID(md5, context);
            //保存到sp中
            SpUtils.putString(context, SpConstant.SP_DEVICES_ID, md5);
        }
        return md5;
    }


    /**
     * 读取固定的文件中的内容,这里就是读取sd卡中保存的设备唯一标识符
     *
     * @param context
     * @return
     */
    public static String readDeviceID(Context context) {
        File file = getDevicesDir(context);
        StringBuffer buffer = new StringBuffer();
        try {
            FileInputStream fis = new FileInputStream(file);
            InputStreamReader isr = new InputStreamReader(fis, "UTF-8");
            Reader in = new BufferedReader(isr);
            int i;
            while ((i = in.read()) > -1) {
                buffer.append((char) i);
            }
            in.close();
            return buffer.toString();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 获取设备MAC 地址 由于 6.0 以后 WifiManager 得到的 MacAddress得到都是 相同的没有意义的内容
     * 所以采用以下方法获取Mac地址
     *
     * @param context
     * @return
     */
    private static String getLocalMac(Context context) {
        String macAddress = null;
        StringBuffer buf = new StringBuffer();
        NetworkInterface networkInterface = null;
        try {
            networkInterface = NetworkInterface.getByName("eth1");
            if (networkInterface == null) {
                networkInterface = NetworkInterface.getByName("wlan0");
            }
            if (networkInterface == null) {
                return "";
            }
            byte[] addr = networkInterface.getHardwareAddress();
            for (byte b : addr) {
                buf.append(String.format("%02X:", b));
            }
            if (buf.length() > 0) {
                buf.deleteCharAt(buf.length() - 1);
            }
            macAddress = buf.toString();
        } catch (SocketException e) {
            e.printStackTrace();
            return "";
        }
        return macAddress;
    }

    /**
     * 保存 内容到 SD卡中,  这里保存的就是 设备唯一标识符
     *
     * @param str
     * @param context
     */
    public static void saveDeviceID(String str, Context context) {
        File file = getDevicesDir(context);
        try {
            FileOutputStream fos = new FileOutputStream(file);
            Writer out = new OutputStreamWriter(fos, "UTF-8");
            out.write(str);
            out.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 对挺特定的 内容进行 md5 加密
     *
     * @param message   加密明文
     * @param upperCase 加密以后的字符串是是大写还是小写  true 大写  false 小写
     * @return
     */
    public static String getMD5(String message, boolean upperCase) {
        String md5str = "";
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] input = message.getBytes();
            byte[] buff = md.digest(input);
            md5str = bytesToHex(buff, upperCase);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return md5str;
    }


    public static String bytesToHex(byte[] bytes, boolean upperCase) {
        StringBuffer md5str = new StringBuffer();
        int digital;
        for (int i = 0; i < bytes.length; i++) {
            digital = bytes[i];

            if (digital < 0) {
                digital += 256;
            }
            if (digital < 16) {
                md5str.append("0");
            }
            md5str.append(Integer.toHexString(digital));
        }
        if (upperCase) {
            return md5str.toString().toUpperCase();
        }
        return md5str.toString().toLowerCase();
    }

    /**
     * 统一处理设备唯一标识 保存的文件的地址
     *
     * @param context
     * @return
     */
    private static File getDevicesDir(Context context) {
        File mCropFile = null;
        if (Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
            File cropdir = new File(Environment.getExternalStorageDirectory(), CACHE_IMAGE_DIR);
            if (!cropdir.exists()) {
                cropdir.mkdirs();
            }
            mCropFile = new File(cropdir, DEVICES_FILE_NAME); // 用当前时间给取得的图片命名
        } else {
            File cropdir = new File(context.getFilesDir(), CACHE_IMAGE_DIR);
            if (!cropdir.exists()) {
                cropdir.mkdirs();
            }
            mCropFile = new File(cropdir, DEVICES_FILE_NAME);
        }
        return mCropFile;
    }
}
```


## 总结
设备标识获取的思路大致就是结合deviceId、SharePreference以及隐藏文件，优先级从高到低，具体使用可以结合实际场景做适当的修正。



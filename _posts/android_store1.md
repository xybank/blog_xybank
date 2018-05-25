---
title: Android_6.数据存储_1.文件存储
date:  2018-05-25 12:27:25
tags: [android,file]
categories: 张永枫
description: 本文是以Android Studi为开发工具，<<第一行代码Anroid第2版>> 为学习指导书籍的学习记录
---

[示例源码下载百度云地址](https://pan.baidu.com/s/1C-A73-CsGn493o-H_NBM1Q)

# 6.1 数据的持久化技术
数据持久代就是指那些内存中的瞬时数据保存到存储设备中，保证即使在手机或电脑关机的情况下，这些数据仍然不会丢失。
Android有三种方式实现数据持久化功能：
1. 文件存储；
2. SharePreferenced存储；
3. 数据库存储。

# 6.2 文件存储
文件存储是Android中最基本的一种数据存储方式，这种方式不对存储的内容进行任何的格式化处理，数据会原封不动地保存到文件中；
所以这种方法比较适合==一些简单的文本数据，如文本数据或二进制文件==。

## 6.2.1 将数据存储到文件中

### openFileOutout()方法
Context类中提供了一个openfileOutput()方法，用于将数据存储到指定的文件中，这个方法接收两个参数：

* 第一个参数：指定创建文件的名称(不包括文件的路径，文件默认会存储到/data/date/\<pageage name\>/file/目录下面
* 第二个参数：指定文件的操作模式有两种

    1. MODE_PRIVATE（默认模式,当保存的文件名一样时，写入的内容会覆盖原文件的内容）
    2. MODE_APPEND（当保存的文件名一样时，写入的内容会在原文件的内容进入追加。）

openfileOutput（）方法返回的是一个FileOutputStream对象，得到这个对象后就可以使用==Java流的方式==将数据写入到文件中如下java 示例：

```
    public void save(){
        String data = "Data to save";
        FileOutputStream out = null;
        BufferedWriter writer = null;
        try{
            out = openFileOutput("data", Context.MODE_PRIVATE);
            writer = new BufferedWriter(new OutputStreamWriter(out));
            writer.write(data);
        }catch (IOException e){
            e.printStackTrace();
        }finally{
            try{
                if (writer != null){
                    writer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private FileOutputStream openFileOutput(String data, int modePrivate) {

    }
```


新建一个“FilePersistenceTest”项目，
修改activity_main.xml文件的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/edit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Type something here"/>

    <Button
        android:id="@+id/button_1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="保存数据"
        />
</LinearLayout>
```

修改MainActivity类的代码如下：

```
package com.zyf.android.filepersistencetest;

import android.content.Context;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.text.TextUtils;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class MainActivity extends AppCompatActivity {
    //声明一个EditText对象
    private EditText edit;

    private Button button_obj;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        edit = (EditText) findViewById(R.id.edit);//获取实例
        button_obj = (Button) findViewById(R.id.button_1);
        button_obj.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.i("SecondActivity", "123131231231231");
                String inputText = edit.getText().toString();
                Log.i("SecondActivity", inputText);
                save(inputText);
            }
        });
    }
     
    //窗口销毁后响应OnDestroy();
    protected void onDestroy(){
        super.onDestroy();
    }

    //保存数据的方法
    public void save(String inputText){
        FileOutputStream out = null;
        BufferedWriter writer = null;
        try{
            out = openFileOutput("data2.txt", Context.MODE_PRIVATE);
            Log.i("SecondActivity", "out");

            writer = new BufferedWriter(new OutputStreamWriter(out));
            Log.i("SecondActivity", "writer");

            writer.write(inputText);//执行写入数据的方法
        }catch (IOException e){
            Log.i("SecondActivity", "IOException1");
            e.printStackTrace();
            //对异常进行处理

        }finally {
            try{
                if (writer != null){
                    writer.close();
                    Log.i("SecondActivity", "writer is close");
                }
            }catch (IOException e){
                Log.i("SecondActivity", "IOException1");
                e.printStackTrace();
            }
        }

    }
}

```

查看保存的文件
借助Android device Monitor 工具（点击AS,右下角的Android device Monitor）会打开一个列表。

打开data>data>项目的包名>files>保存文件的名字
![](https://upload-images.jianshu.io/upload_images/6841945-85bbe4977ee113af.jpg?imageMogr2/auto-orient/)

## 6.2.2 从文件中读取数据

使用openFileInput()方法，获取一个FileInputStream对象，再建一个InputStreamReader对象，接着构建BufferedReader对象，读取文字并存放到StringBuilder 对象中去。
修改MainActivity 代码如下；

```
package com.zyf.android.filepersistencetest;
import android.content.Context;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.text.TextUtils;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class MainActivity extends AppCompatActivity {
    //声明一个EditText对象
    private EditText edit;
    private Button button_obj;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        edit = (EditText) findViewById(R.id.edit);//获取实例

        String inputText = load();
        if (!TextUtils.isEmpty(inputText)){
            edit.setText(inputText);
            edit.setSelection(inputText.length());
            Toast.makeText(this,"Restoring succeeded",Toast.LENGTH_SHORT).show();
        }

        button_obj = (Button) findViewById(R.id.button_1);
        button_obj.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.i("SecondActivity", "123131231231231");
                String inputText = edit.getText().toString();
                Log.i("SecondActivity", inputText);
                save(inputText);
            }
        });
    }

    //加载文件的方法
    private String load() {
        FileInputStream in = null;
        BufferedReader reader = null;
        StringBuilder content = new StringBuilder();
        try {
            in = openFileInput("data2.txt");
            reader = new BufferedReader(new InputStreamReader(in));
            String line = "";
            while ((line = reader.readLine()) != null){
                content.append(line);
            }
        }catch (IOException e ){
            e.printStackTrace();
        }finally {
            if (reader != null){
                try{
                    reader.close();
                }catch (IOException e){
                    e.printStackTrace();
                }
            }
        }
        return content.toString();
    }

    //窗口销毁后响应OnDestroy();
    protected void onDestroy(){
        super.onDestroy();
    }

    //保存数据的方法
    public void save(String inputText){
        FileOutputStream out = null;
        BufferedWriter writer = null;
        try{
            out = openFileOutput("data2.txt", Context.MODE_PRIVATE);
            Log.i("SecondActivity", "out");

            writer = new BufferedWriter(new OutputStreamWriter(out));
            Log.i("SecondActivity", "writer");

            writer.write(inputText);//执行写入数据的方法
        }catch (IOException e){
            Log.i("SecondActivity", "IOException1");
            e.printStackTrace();
            //对异常进行处理

        }finally {
            try{
                if (writer != null){
                    writer.close();
                    Log.i("SecondActivity", "writer is close");
                }
            }catch (IOException e){
                Log.i("SecondActivity", "IOException1");
                e.printStackTrace();
            }
        }
    }
}

```

再运行代码就可以了



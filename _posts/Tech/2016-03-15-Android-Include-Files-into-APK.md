---
layout: post
title:  "Include configure files into Android APK"
date:   2016-03-15 16:00:00
tags: [android, assets, include file, tech]
categories: Tech
---

>  项目中遇到android APK需要参数配置文件，将其打包入assets

`Copy files into dir src/main/assets/`
{% highlight Java %}
/**
 *  从assets目录中复制需要的文件夹(不是全部的，除了建立的文件夹，还有些自动生成的文件夹)
 *  @param  oldPath  String  原文件夹路径  如：/aa
 *  @param  newPath  String  复制后文件夹路径  如：/bb/cc
 */
private void copyFilesFassets(String oldPath, String newPath) {
    try {
        String fileNames[] = getAssets().list(oldPath);//获取该目录下的所有文件
        File folder = new File(newPath);
        if (!folder.exists()) {
            folder.mkdirs();
        }
        for (String fileName : fileNames) {
            File newFile = new File(newPath + "/" + fileName);
            if(newFile.exists()) {
                continue;
            }
            InputStream is = getAssets().open(oldPath+ "/" + fileName);
            FileOutputStream fos = new FileOutputStream(newFile);
            byte[] buffer = new byte[1024];
            int byteCount=0;
            while((byteCount=is.read(buffer))!=-1) {//循环从输入流读取 buffer字节
                fos.write(buffer, 0, byteCount);//将读取的输入流写入到输出流
            }
            fos.flush();//刷新缓冲区
            is.close();
            fos.close();
            Log.i(LOG_TAG, "Written configure files: " + newFile);
        }
    } catch (Exception e) {
        Log.e(LOG_TAG, "Exception: copy assets ", e);
    }
}
{% endhighlight %}
之后底层C++库就可以利用newPath获取configuration files
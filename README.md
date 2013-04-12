me.fantouch.libs
================

**Android Library工程,包含若干模块,使Android开发更方便**  
**你的Android工程添加本Library依赖,即可使用**  
  
**含有以下模块**
* [CrashHandler崩溃处理模块](https://github.com/fantouch/me.fantouch.libs#crashhandler)  
* [ELog日志模块](https://github.com/fantouch/me.fantouch.libs#elog)  
* [UpdateHelper自动更新模块](https://github.com/fantouch/me.fantouch.libs#updatehelper)  

>本Library已包含 [afinal.jar](https://github.com/yangfuhai/afinal) 和 `android-support-v4.jar`,  
你的工程请注意不需要重复包含了.

================  

##CrashHandler崩溃处理模块
* 崩溃了  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/6e307ff6-15bc-40ea-a3de-c0ebb05733af.jpg?resizeSmall&width=832)  

* 崩溃报告自动存储在私有目录(/data/data/com.xxx)  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/6cd49d87-8abb-4fa9-9fe4-e33920ef7bb9.jpg?resizeSmall&width=832)  

* 服务器收到崩溃报告  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/214cb2a3-eb44-41c0-8527-6536c7c302e9.jpg?resizeSmall&width=832)  

###CrashHandler需要权限  
```xml  
<!-- 不依赖Activity的Context弹出Dialog -->
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
   
<!-- 检查是否wifi网络 (如果需要上传日志) -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- 使用网络上传日志 (如果需要上传日志) -->
   <uses-permission android:name="android.permission.INTERNET" />
```
###如何使用CrashHandler  
* 在自定义的MyApplication内注册CrashHandler   

```java
    public class MyApplication extends Application {
        @Override
        public void onCreate() {
            super.onCreate();
    
            /* 注册crashHandler,保存日志但不上传到服务器 */
            CrashHandler.getInstance().init(getApplicationContext(), null);
            
            /* 注册crashHandler,保存日志并自动上传到服务器 */
            CrashHandler.getInstance().init(getApplicationContext(), SendService.class);
            
            // 根据你与服务器的交互协议,实现 SendService,
            // 见下文 CrashHandler如何能自动上传日志到服务器? {@link https://github.com/fantouch/me.fantouch.libs/edit/master/README.md#crashhandler-3}
            
        }
    }
```  

* 在 `AndroidManifest.xml` 内注册你的MyApplication   

```xml 
<application
  android:name="me.fantouch.libs.crash.MyApplication"
  … >
    <activity> … </activity>
</application>  
```

####CrashHandler如何能自动上传日志到服务器?
* 请根据你与服务器的交互协议,实现 `SendService`, 示例:

>推荐使用[FinalHttp](https://github.com/yangfuhai/afinal)(me.fantouch.libs已包含FinalHttp)

```java
    public class SendService extends AbsSendReportsService {
        @Override
        public void sendZipReportsToServer(File reportsZip, NotificationHelper notificationHelper) {
            
            AjaxParams params = new AjaxParams();
            try {
                params.put("reportsZip", reportsZip);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
            
            FinalHttp fh = new FinalHttp();
            fh.post("http://server.com/upload.php", params, new AjaxCallBack<String>() {
                
                @Override
                public void onSuccess(String t) {
                    stopSelf();// 无论成功还是失败,都应该停止服务
                }
                
                @Override
                public void onFailure(Throwable t, String strMsg) {
                    stopSelf();// 无论成功还是失败,都应该停止服务
            }
            });
            
        } 
    }
```


================  

##ELog日志模块
* `ELog.d("Hello~~");`  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/39fdd19e-c607-4ad9-b80b-d169f5a979d7.png?resizeSmall&width=832)  

* Eclipse Logcat输出  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/e8f2016e-e8e8-46e1-8b23-3a21442fa75b.jpg?resizeSmall&width=832)  
 * 简单,你只需关心要Log的 **内容**
 * 自动用类名填充 **TAG**
 * 自动Log **方法名**
 * 自动Log **文件名**, **行号**
 * Eclipse的LogCat里面双击, **转跳到Java文件** 相应行   
 (过段时间再来调试,有没有感觉看不懂log,找不到log语句在哪里?)
 * 可以把 **日志保存** 到文件
 * 可以 **上传日志** 到服务器  
 
###ELog需要权限  

```xml  
<!-- 检查是否wifi网络  (如果需要上传日志)-->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- 使用网络上传日志  (如果需要上传日志)-->
   <uses-permission android:name="android.permission.INTERNET" />
```
###如何使用ELog  

* 开启ELog  
 
>建议在`Application`里面执行,或者做一个菜单让用户在需要的时候自行开启  
如果不启用,ELog将什么都不做,不会耗费系统资源  

```java
ELog.setEnableLogcat(true);// 启用Logcat输出
```
* 使用    

```java
ELog.d("Hello~~");
```

####ELog如何保存日志?  
```java
// 启用保存日志功能
// 日志文件在/data/data/com.xxx
ELog.setEnableLogToFile(true, getApplicationContext());
```
####ELog如何上传日志到服务器?  
* 启用保存日志功能
* 然后  

```java
// SendService extends AbsSendReportsService
ELog.sendReportFiles(getApplicationContext(), SendService.class);
```

* 注意,你需要根据你与服务器的协议,实现[SendService](https://github.com/fantouch/me.fantouch.libs/edit/master/README.md#-3)


================  

##UpdateHelper自动更新模块
* 发现新版本  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/a7ccd1f6-fdaf-4b40-a2d0-bcbdff2b3a41.png?resizeSmall&width=832)  

* 后台下载  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/62c5e4a8-4836-4a04-9912-e58c02210c86.png?resizeSmall&width=832)  

* 下载完成  
![](https://www.evernote.com/shard/s25/sh/4d01bbd4-c5df-4d90-a617-29e5ead4bfc2/e18af5ee47804638bcf9c4251b9639a9/res/90d142ba-0d85-4371-b8f6-e418fac3e806.png?resizeSmall&width=832)  

###UpdateHelper需要权限  

```xml  
   <!-- 使用网络 -->
   <uses-permission android:name="android.permission.INTERNET" />
   
   <!-- 更新apk文件会下载到sd卡 /mnt/sdcard/com.xxx -->  
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

###如何使用UpdateHelper
```java
// 需要实现 parser 和 normalUpdateListener
 new UpdateHelper(this, parser, normalUpdateListener).check("http://192.168.1.100/checkUpdate.php");
```
* 根据与服务器的交互协议,实现 `parser`.示例:  

```java
        /** 解析服务器信息,并把信息存入UpdateInfoBean */
        AbsUpdateInfoParser parser = new AbsUpdateInfoParser() {
            @Override
            public UpdateInfoBean parse(String info) {
                
                // 解析服务器信息 info
                // ...
                
                UpdateInfoBean infoBean = new UpdateInfoBean();
                
                infoBean.setVersionCode("10");
                infoBean.setVersionName("beta1");
                infoBean.setWhatsNew("修复了xx bug");
                infoBean.setDownUrl("http://server.com/xx_beta1.apk");
                
                return infoBean;
            }
        };
```

* 根据你的业务逻辑,实现 `normalUpdateListener`.示例:   

```java
        NormalUpdateListener normalUpdateListener = new NormalUpdateListener() {
            @Override
            public void onCheckStart() {
                // 可以在这提示用户正在检查更新
            }

            @Override
            public void onDownloadStart() {
                // 如果用户选择下载更新,这个方法会被调用
            }

            @Override
            public void onCheckFinish() {
                // 检查完成(没有可用更新,或者检查更新中途出现异常)
                // 例如你是在程序Loading界面进行检查更新,那么现在可以跳过Loading进入程序首页了
            }

            @Override
            public void onCancel() {
                // 用户不想更新,选择了"下次再说"
                // 例如你是在程序Loading界面进行检查更新,那么现在可以跳过Loading进入程序首页了
            }
        };
```
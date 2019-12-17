# FT Mobile SDK Android

## 安装
- 在项目的根目录的 build.gradle 文件中添加 FT SDK 的的远程仓库地址

``` groovy

repositories {
    //...
    maven {
        url 'https://mvnrepo.jiagouyun.com/repository/maven-releases'
    }

}
```

- 在项目的主模块的 build.gradle 文件中添加 FT SDK 的依赖

``` groovy
dependencies {
    implementation 'com.cloudcare.ft.mobile.sdk.traker.agent:ft-sdk:1.0.0'
}
```

关于最新的版本号，请参考[更新文档](https://gitlab.jiagouyun.com/cma/ft-sdk-android/blob/master/README.md)
    
## 配置

1. 在程序的入口 Application 中添加关于 FT SDK 的初始化配置安装代码。
关于配置项的说明

参数|类型|含义|是否必须
:--:|:--:|:--:|:--:
metricsUrl|String|FT-GateWay metrics 写入地址|是
enableRequestSigning|boolean|配置是否需要进行请求签名|是
akId|String|access key ID|enableRequestSigning 为 true 时，必须要填
akSecret|String|access key Secret|enableRequestSigning 为 true 时，必须要填
useOAID|boolean|是否使用 oaid 字段[了解 OAID](#1关于oaid)|否
isDebug|boolean|是否需要显示日志|否

示例代码
```java
public class DemoApplication extends Application {
    private String accesskey_id = "xxxx";
    private String accessKey_secret = "xxxxx";

    public DemoApplication() {
        super();
    }

    @Override
    public void onCreate() {
        super.onCreate();

        FTSDKConfig ftSDKConfig = new FTSDKConfig("http://xxxxx",
                true,
                accesskey_id,``
                accessKey_secret);
        ftSDKConfig.setUseOAID(true);
        FTSdk.install(ftSDKConfig);
    }
}
```

2. 关于权限的配置
FT SDK 用到了系统的两个权限，分别为 READ_PHONE_STATE、WRITE_EXTERNAL_STORAGE
权限使用说明

名称|使用原因
:--:|:--:
READ_PHONE_STATE|用于获取手机的设备信息，便于精准分析数据信息
WRITE_EXTERNAL_STORAGE|用户存储缓存数据

关于如何申请动态权限，具体详情参考[Android Devloper](https://developer.android.google.cn/training/permissions/requesting?hl=en)

## 方法
1、FT SDK 公开了2个埋点方法，用户通过这三个方法可以主动在需要的地方实现埋点，然后将数据上传到服务端。

- 方法一：

```java
/*** 主动埋点
 * @param event 埋点事件名称
 * @param tags 埋点数据
 * @param values 埋点数据
 */
 public void track(String event, JSONObject tags, JSONObject values)
```

- 方法二：

```java
/**
 * 主动埋点
 * @param event 埋点事件名称
 * @param values 埋点数据
 */
 public void trackValues(String event, JSONObject values)
```

2、方法使用示例

```java
public void clickText(View view) {
    try {
        JSONObject tags = new JSONObject();
        JSONObject values = new JSONObject();
        tags.put("view","text");
        values.put("userName","admin");
        FTTrack.getInstance().track("ClickView",tags,values);
    } catch (JSONException e) {
        e.printStackTrace();
    }
}

@Override
public boolean onOptionsItemSelected(@NonNull MenuItem item) {
    try {
        JSONObject values = new JSONObject();
        values.put("userName","MenuItem");
        FTTrack.getInstance().trackValues("ClickView",values);
    } catch (JSONException e) {
        e.printStackTrace();
    }
    return super.onOptionsItemSelected(item);
}
```


## 常见问题
#### 1.关于OAID
- 介绍

 在 Android 10 版本中，非系统应用将不能获取到系统的 IMEI、MAC 等信息。面对该问题移动安全联盟联合国内的手机厂商推出了
补充设备标准体系方案，选择用 OAID 字段作为IMEI等系统信息的替代字段。OAID 字段是由中国信通院联合华为、小米、OPPO、
VIVO 等厂商共同推出的设备识别字段，具有一定的权威性。
关于 OAID 可移步参考[移动安全联盟](http://www.msa-alliance.cn/col.jsp?id=120)

- 使用

 使用方式和资源下载可参考[移动安全联盟的集成文档](http://www.msa-alliance.cn/col.jsp?id=120)

 示例：

1. 下载好资源文件后，将 miit_mdid_x.x.x.arr 拷贝到项目的 libs 目录下，并设置依赖，其中 x.x.x 代表版本号
[获取最新版本](http://www.msa-alliance.cn/col.jsp?id=120)

    ![Alt](http://zhuyun-static-files-production.oss-cn-hangzhou.aliyuncs.com/helps/markdown-screentshot/ft-sdk-android/use_learn_1.png#pic_center)

2. 将下载的资源中的 supplierconfig.json 文件拷贝到主项目的 assets 目录下，并修改里面对应的内容，特别是需要设置 appid 的部分。
需要设置 appid 的部分需要去对应厂商的应用商店里注册自己的 app。

    ![Alt](http://zhuyun-static-files-production.oss-cn-hangzhou.aliyuncs.com/helps/markdown-screentshot/ft-sdk-android/use_learn_2.png#pic_center)

    ![Alt](http://zhuyun-static-files-production.oss-cn-hangzhou.aliyuncs.com/helps/markdown-screentshot/ft-sdk-android/use_learn_3.png#pic_center)

3. 设置依赖
``` groovy
implementation files('libs/miit_mdid_x.x.x.arr')
```

4. 混淆设置
```
 -keep class com.bun.miitmdid.core.**{*;}
```

5. 设置 gradle 编译选项，这块可以根据自己的对平台的选择进行合理的配置
``` groovy
ndk {
    abiFilters 'armeabi-v7a','x86','arm64-v8a','x86_64','armeabi'
}
packagingOptions {
    doNotStrip "*/armeabi-v7a/*.so"
    doNotStrip "*/x86/*.so"
    doNotStrip "*/arm64-v8a/*.so"
    doNotStrip “*/x86_64/*.so"
    doNotStrip "armeabi.so"
}
```

6. 以上步骤配置完成后，在配置 FT SDK 时调用 FTSDKConfig 的 setUseOAID(true) 方法即可

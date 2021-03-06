## Android SDK 集成文档

[查看DEMO](https://github.com/AppAdhoc/android-abtestdemo)

[ChangeLog](https://github.com/AppAdhoc/AdhocSDK-Android/blob/master/changelog.md)
[Android SDK API 参考](http://www.appadhoc.com/android/reference/)

<h3 id="sdk"> 下载SDK </h3>

更新时间：2017/3/23

[adhoc-min.jar&nbsp;&nbsp;2.3.6](https://www.appadhoc.com/downloads/android/adhoc-v2.3.6-min.jar)

### 导入SDK

将下载得到的 SDK JAR拖入到的AndroidStudio / Eclipse 工程根目录libs中(没有则新建)，右键Add as Library添加到库：（示意图，实际包名以最新版本为准）

![导入SDK](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/android1.png)

### 加入网络和SDCARD读写权限

在项目中找到项目配置文件 AndroidManifest.xml，加入网络访问权限和SDCARD读写权限：

```
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

<h3 id="init"> SDK初始化 </h3>

创建Appllication 类，并在OnCreate()中加入下面代码：

```
AdhocConfig adhocConfig = new AdhocConfig.Builder()
        //设置App上下文(必要参数)
        .context(this)
        //设置Appkey(必要参数)
        .appKey(key)
        .build();

AdhocTracker.init(adhocConfig);
```

需要在AndroidManifest.xml 里指定类名：

![SDK初始化](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/android2.png)

如果已经创建过Application类，跳过新建步骤，直接在onCreate加入上述代码即可。

其中，“appKey” 是在登录 AppAdhoc 后,创建“应用”之后获得的授权标识。

<p style="color:#aaa">注意：试验“应用”此时应该提前创建完毕。可在AppAdhoc控制台应用列表找到，如下图红线部分：</p>

![your_app_key](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/appkey.png)

<p style="color:#a00">请勿在SDK基础上进行自行封装，以免影响到试验逻辑，造成试验无法正常运行。如果确有自行封装的需求，请与客户经理联系，获取注意事项。</p>

<!-- init方法中，支持的全部配置如下（非必要）：
```
AdhocConfig adhocConfig = new AdhocConfig.Builder()
        //设置App上下文(必要参数)
        .context(this)
        //设置Appkey(必要参数)
        .appKey(key)
        //设置clientId,将<xxxx>替换为clientId
        .clientId("xxxx")
        //添加定向试验条件（自定义用户标签）
        .addCustom("sex", "male")
        .addCustom("age", "17")
        .addCustom("name", "20")
        //调用后,会自动上报崩溃次数统计
        .reportCrash()
        //调用后,优化指标只有在wifi网络下才会上报数据(可能会造成官网数据延时显示)
        .reportWifi()
        //调用后,会自动统计App的session时长
        .reportDuration()
        //调用后自动统计session
        .reportSession()
        //设置session最大间隔时间为10分钟(单位:毫秒)
        .intervalSessionDuration(60 * 1000)
        .build();

AdhocTracker.init(adhocConfig);
``` -->

<h3 id="flag"> 编程模式：根据“试验变量”展示相应内容</h3>

试验变量的值决定了展示的内容或程序的逻辑。试验变量在编程模式试验中创建，可视化试验无需此步骤。  
<p style="color:#aaa">注意：试验变量值应由PM或相关A/B Testing需求制定人员在后台提前录入完毕，如下图“版本管理”红线部分：</p>

![your_app_key](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/flag.png)

在调用SDK之前，记得引用头文件：

```
import com.adhoc.adhocsdk.AdhocTracker;
```

获取变量的值，请在使用到该变量处添加以下代码：

```
//获取Boolean类型的试验变量isNewHomePage的值，第二个参数为设定默认值，在网络异常时将按照该默认值执行
if (AdhocTracker.getFlag("isNewHomePage", false)) {
    //跳转至新首页
} else {
    //跳转至新旧首页
}
```

其中，'isNewHomePage' 即是“试验变量“，应与上图中红线标识保持一致。  
<p style="color:#a00">请注意在用户访问到试验页面时，需要调用过所有变量才算作进入试验，否则将不会统计试验数据。</p>

<h3 id="stat"> 上报指标</h3>

指标用于量化试验结果的好坏，AppAdhoc 后台中的试验图表根据此数据生成。
在可视化试验中，您也可以通过此方法定义上报指标。

<p style="color:#aaa">注意：指标值应由PM或相关AB Test需求制定人员在后台提前录入完毕，如下图“优化指标”红线部分：</p>

![优化指标](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/stat.png)

比如在进入某一逻辑分支后，可以统计点击次数。将上图中的指标“clickTimes”传入函数track实现上报指标, 每次累加1：

```
AdhocTracker.track("clickTimes", 1);
```

### 启用预定义指标

AppAdhoc提供3个预定义指标：访问时长、会话数、崩溃数，只需要在init时添加对应配置，即可获取统计数据。

<p style="color:#aaa">注意：指标值应由PM或相关AB Test需求制定人员在后台选择添加：</p>

![](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/stats3.png)

在init方法中加入以下配置项：
```
AdhocConfig adhocConfig = new AdhocConfig.Builder()
        .context(this)
        .appKey(key)
        //调用后,会自动上报崩溃次数统计
        .reportCrash()
        //调用后,会自动统计App的session时长
        .reportDuration()
        //调用后自动统计session
        .reportSession()
        .build();

AdhocTracker.init(adhocConfig);
```

### 混淆相关

在proguard-rules.txt文件中加入：

```
-keep class com.adhoc.** {*;}
```

对于 SDK 2.0.5 及之后版本还需加入如下代码：

```
-keep class android.support.v4.**{*;}
```

<h3 id="debug"> 集成调试 </h3>

集成调试只是为验证SDK的集成是否成功（并不是真正开始试验！），详见[移动调试工具](./testTools.md)。

### 开始试验

恭喜，您完成了AppAdhoc AB Testing Web SDK的埋点集成工作，请通知PM或相关AB Test需求制定人员，点下开始试验按钮吧！

<p style="color:#aaa">注意：确保app_key, 试验变量字符串，指标字符串与后台截图处一一对应，否则可能出现异常或无试验数据情况。</p>

<h3 id="orientation"> 高级功能 自定义受众定向（需要联系管理员开启）</h3>

AppAdhoc Web SDK 会自动把浏览器名称、版本、语言等用户标签自动上传，开发者也可以根据需要给当前用户打上合适的自定义标签，进而实现将不符合条件的用户排除在此次试验之外。比如只想要女性用户，或30岁以下的用户参与试验等。

<p style="color:#aaa">注意：自定义受众定向条件应由PM或相关AB Test需求制定人员在后台提前录入完毕，如下图“受众定向”红线部分。</p>


在运行控制/右侧定向试验：

![受众定向](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/button.png)

选择分组，点击编辑用户群：

![受众定向](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/dialog.png)

即得到受众条件的key，在下图例子中，“sex”是key：

![受众定向](https://github.com/AppAdhoc/AdhocSDK-Android/raw/master/picture/setting1.png)

在AdhocTracker.init()方法中进行设置：

```
AdhocConfig adhocConfig = new AdhocConfig.Builder()
        //设置App上下文(必要参数)
        .context(this)
        //设置Appkey(必要参数)
        .appKey(key)
        //添加自定义用户标签
        .addCustom("sex", "male")
        .addCustom("age", "17")
        .addCustom("name", "20")
        .build();

AdhocTracker.init(adhocConfig);
```

API 参考

[android SDK API 参考](http://www.appadhoc.com/android/reference/)

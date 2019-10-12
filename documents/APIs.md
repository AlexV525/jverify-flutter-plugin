[Common API](#common-api)

- [setup](#setup)
- [setDebugMode](#setDebugMode)
- [checkVerifyEnable](#checkVerifyEnable)
- [getToken](#getToken)
- [verifyNumber](#verifyNumber)
- [loginAuth](#loginAuth)
- [setCustomAuthViewAllWidgets](#setCustomAuthViewAllWidgets)

#### setup

添加初始化方法，调用 setup 方法初始化 Jverify SDK

**注意：** 插件版本 >= 0.0.1 android 端仅支持在 gradle 中配置 appKey 和 channel。参数 appKey、channel 只对 IOS 生效。

```dart

Jverify jverify = new Jverify();
jverify.setup(
    appKey: "JPush 上注册的包名对应的 Appkey", 
    channel: "devloper-default"
); // 初始化sdk,  appKey 和 channel 只对ios设置有效
```

#### setDebugMode

设置是否开启debug模式。true则会打印更多的日志信息

```dart
Jverify jverify = new Jverify();
jverify.setDebugMode(true); // 打开调试模式
```


#### checkVerifyEnable

判断当前的手机网络环境是否可以使用认证。

```dart
Jverify jverify = new Jverify();
jverify.checkVerifyEnable().then((map){
      bool result = map["result"];
      if(result){
        // 当前网络环境支持认证
      }else{
        // 当前网络环境不支持认证
      }
    });
```

#### getToken

获取当前在线的sim卡所在运营商及token。如果获取成功代表可以用来验证手机号。获取失败则建议做短信验证
**说明：** 开发者可以通过SDK获取token接口的回调信息来选择验证方式，若成功获取到token则可以继续使用极光认证进行号码验证；若获取token失败，需要换用短信验证码等方式继续完成验证。


```dart
Jverify jverify = new Jverify();
jverify.getToken().then((map){
          int _code = map["code"]; // 返回码，2000代表获取成功，其他为失败，详见错误码描述
          String _token = map["content"]; // 成功时为token，可用于调用验证手机号接口。token有效期为1分钟，超过时效需要重新获取才能使用。失败时为失败信息
          String _operator = map["operator"]; // 成功时为对应运营商，CM代表中国移动，CU代表中国联通，CT代表中国电信。失败时可能为null
          ...
        });
```


####  loginAuth

调起一键登录授权页面，在用户授权后获取loginToken
**说明：** ios在拉起授权页面之前，必须先setCustomUI 。

```dart
///具体使用可以查看 example 样例

Jverify jverify = new Jverify();

/// 步骤 1：调用接口设置 UI
jverify.setCustomAuthorizationView();

/// 步骤 2： 添加 loginAuth 接口回调的监听 （如果想通过 loginAuth 接口异步返回获取接口数据，则忽略此步骤）
jverify.addLoginAuthCallBackListener((event){
  print("通过添加监听，获取到 loginAuth 接口返回数据，code=${event.code},message = ${event.message},operator = ${event.operator}");
});
/// 步骤 3：开始调用一键登录接口        
jverify.loginAuth(true).then((map){
  /// 步骤 4：获取 loginAuth 接口异步返回数据（如果是通过添加 JVLoginAuthCallBackListener 监听来获取返回数据，则忽略此步骤）
  print("通过接口异步返回，获取到 loginAuth 接口返回数据，code=$code,message = $content,operator = $operator");
});
```

#### setCustomAuthorizationView

修改授权页面主题，开发者可以通过 setCustomAuthorizationView 方法修改授权页面主题，需在 loginAuth 接口之前调用

```
/// 先配置好 JVUIConfig 和 JVCustomWidget 数组
jverify.setCustomAuthorizationView(false,uiConfig,widgets: widgetList);
```
**JVUIConfig 和 JVCustomWidget 配置如下：**

##### JVUIConfig
```dart
/// 自定义授权的 UI 界面
JVUIConfig uiConfig = JVUIConfig();
uiConfig.navColor = Colors.red.value;

uiConfig.navText = "登录";
uiConfig.navTextColor = Colors.blue.value;
uiConfig.navReturnImgPath = "return_bg";

uiConfig.logoHidden = false;
uiConfig.logoOffsetY = 10;
uiConfig.logoWidth = 90;
uiConfig.logoHeight = 90;
uiConfig.logoImgPath = "logo";

uiConfig.numFieldOffsetY = 120;
uiConfig.numberColor = Colors.blue.value;

uiConfig.sloganOffsetY = 150;
uiConfig.sloganTextColor = Colors.black.value;

uiConfig.logBtnOffsetY = 300;
uiConfig.logBtnText = "登录按钮";
uiConfig.logBtnTextColor = Colors.brown.value;
uiConfig.loginBtnNormalImage = "login_btn_normal";
uiConfig.loginBtnPressedImage = "login_btn_press";
uiConfig.loginBtnUnableImage = "login_btn_unable";

uiConfig.checkedImgPath = "check_image";
uiConfig.uncheckedImgPath = "uncheck_image";
uiConfig.privacyOffsetY = 80;

uiConfig.clauseName = "协议1";
uiConfig.clauseUrl = "http://www.baidu.com";
uiConfig.clauseBaseColor = Colors.black.value;

uiConfig.clauseNameTwo = "协议二";
uiConfig.clauseUrlTwo = "http://www.hao123.com";
uiConfig.clauseColor = Colors.red.value;


uiConfig.privacyState = true;

Jverify jverify = new Jverify();

```

#### JVCustomWidget 自定义控件

```
 /// 添加自定义的 控件 到授权界面
List<JVCustomWidget>widgetList = [];

final String text_widgetId = "jv_add_custom_text";// 标识控件 id
JVCustomWidget textWidget = JVCustomWidget(text_widgetId, JVCustomWidgetType.textView);
textWidget.title = "新加 text view 控件";
textWidget.left = 20;
textWidget.top = 360 ;
textWidget.width = 200;
textWidget.height  = 40;
textWidget.backgroundColor = Colors.yellow.value;
textWidget.isShowUnderline = true;
textWidget.textAlignment = JVTextAlignmentType.center;
textWidget.isClickEnable = true;

// 添加点击事件监听
jverify.addClikWidgetEventListener(text_widgetId, (eventId) {
  print("receive listener - click widget event :$eventId");
  if (text_widgetId == eventId) {
    print("receive listener - 点击【新加 text】");
  }
});
widgetList.add(textWidget);

final String btn_widgetId = "jv_add_custom_button";// 标识控件 id
JVCustomWidget buttonWidget = JVCustomWidget(btn_widgetId, JVCustomWidgetType.button);
buttonWidget.title = "新加 button 控件";
buttonWidget.left = 100;
buttonWidget.top = 400;
buttonWidget.width = 150;
buttonWidget.height  = 40;
buttonWidget.isShowUnderline = true;
buttonWidget.backgroundColor = Colors.brown.value;
//buttonWidget.btnNormalImageName = "";
//buttonWidget.btnPressedImageName = "";
//buttonWidget.textAlignment = JVTextAlignmentType.left;

// 添加点击事件监听
jverify.addClikWidgetEventListener(btn_widgetId, (eventId) {
  print("receive listener - click widget event :$eventId");
  if (btn_widgetId == eventId) {
    print("receive listener - 点击【新加 button】");
  }
});
widgetList.add(buttonWidget);

```


|参数名	|参数类型	|说明|
|:----:|:-----:|:-----:|
|navColor	|int	|设置导航栏颜色|
|navText	|String	|设置导航栏标题文字|
|navTextColor	|int	|设置导航栏标题文字颜色|
|navReturnImgPath	|String	|设置导航栏返回按钮图标|
|logoWidth	|int	|设置logo宽度（单位：dp）|
|logoHeight	|int	|设置logo高度（单位：dp）|
|logoHidden	|boolean	|隐藏logo|
|logoOffsetY	|int	|设置logo相对于标题栏下边缘y偏移|
|logoImgPath	|String	|设置logo图片|
|numberColor	|int	|设置手机号码字体颜色|
|numFieldOffsetY	|int	|设置号码栏相对于标题栏下边缘y偏移|
|logBtnText	|String|	设置登录按钮文字|
|logBtnTextColor	|int	|设置登录按钮文字颜色|
|logBtnBackgroundPath	|String	|设置授权登录按钮图片(android)|
|loginBtnNormalImage  |String   |设置授权登录按钮正常状态图片(ios)|
|loginBtnPressedImage  |String   |设置授权登录按钮按下状态图片(ios)|
|loginBtnUnableImage   |String   |设置授权登录按钮不可用状态图片(ios)|
|logBtnOffsetY	|int	|设置登录按钮相对于标题栏下边缘y偏移|
|clauseName	 |String	|设置开发者隐私条款1名称|
|clauseUrl   |String	|设置开发者隐私条款1的URL|
|clauseNameTwo	|String	|设置开发者隐私条款2名称|
|clauseUrlTwo   |String	|设置开发者隐私条款2的URL|
|clauseBaseColor	|int	|设置隐私条款名称颜色(基础文字颜色)|
|clauseColor	|int	|设置隐私条款名称颜色(协议文字颜色)|
|privacyOffsetY	|int	|设置隐私条款相对于授权页面底部下边缘y偏移|
|checkedImgPath	|String	|设置复选框选中时图片|
|uncheckedImgPath	|String	|设置复选框未选中时图片|
|sloganTextColor	|int	|设置移动slogan文字颜色|
|sloganOffsetY	|int	|设置slogan相对于标题栏下边缘y偏移|

**说明：** 关于图片存放路径问题，android项目将图片存放至drawable文件夹下，可使用图片选择器的文件名,例如：btn_login.xml,入参为"btn_login"。
ios项目存放在 Assets.xcassets。

#### 错误码

|code	|message	|备注|
|:-----:|:----:|:-----:|
|1000	|verify consistent	|手机号验证一致|
|1001	|verify not consistent	|手机号验证不一致|
|1002	|unknown result	|未知结果|
|1003	|token expired	|token失效|
|1004	|sdk verify has been closed|	SDK发起认证未开启|
|1005	|AppKey 不存在	|请到官网检查 Appkey 对应的应用是否已被删除|
|1006	|frequency of verifying single number is beyond the maximum limit	|同一号码自然日内认证消耗超过限制|
|1007	|beyond daily frequency limit	|appKey自然日认证消耗超过限制|
|1008	|AppKey 非法	|请到官网检查此应用详情中的 Appkey，确认无误|
|1009	|当前的 Appkey 下没有创建 iOS 应用；|你所使用的 JCore 版本过低	请到官网检查此应用的应用详情；更新应用中集成的 JCore 至最新。|
|1010	|verify interval is less than the minimum limit	|同一号码连续两次提交认证间隔过短|
|1011	|appSign invalid	|应用签名错误，检查签名与Portal设置的是否一致|
|2000	|token request success	|获取token 成功|
|2001	|fetch token error	|获取token失败|
|2002	|sdk init failed	|sdk初始化失败|
|2003	|netwrok not reachable	|网络连接不通|
|2004	|get uid failed	|极光服务注册失败|
|2005	|request timeout	|请求超时|
|2006	|fetch config failed	|获取配置失败|
|2007	|内容为异常信息	|验证遇到代码异常|
|2008	|Token requesting, please wait	|正在获取token中，稍后再试v
|2009	|verifying, please try again later	|正在认证中，稍后再试|
|2010	|don't have READ_PHONE_STATE permission	|未开启读取手机状态权限|
|2011	|内容为异常信息	|获取配置时代码异常|
|2012	|内容为异常信息	|获取token时代码异常|
|2013	|内容为具体错误原因	|网络发生异常|
|2014	|internal error while requesting token	|请求token时发生内部错误|
|2015	|rsa encode failed	|rsa加密失败|
|2016	|network type not supported	|当前网络环境不支持认证|
|4001	|parameter invalid	|参数错误。请检查参数，比如是否手机号格式不对|
|4009	|	|解密rsa失败|
|4018	|	|没有足够的余额|
|4031	|	|不是认证用户|
|4032	|	|获取不到用户配置|
|4033	|appkey is not support login	|不是一键登录用户|
|5000	|bad server	|服务器未知错误|
|6000	|内容为token	|获取loginToken成功|
|6001	|fetch loginToken failed	|获取loginToken失败|
|6002	|fetch loginToken canceled	|用户取消获取loginToken|
|6003	|UI 资源加载异常	|未正常添加sdk所需的资源文件|
|-994	|   |网络连接超时	|
|-996	|   |网络连接断开	|
|-997	|  注册失败/登录失败	|（一般是由于没有网络造成的）如果确保设备网络正常，还是一直遇到此问题，则还有另外一个原因：JPush 服务器端拒绝注册。而这个的原因一般是：你当前 App 的 Android 包名以及 AppKey，与你在 Portal 上注册的应用的 Android 包名与 AppKey 不相同。|


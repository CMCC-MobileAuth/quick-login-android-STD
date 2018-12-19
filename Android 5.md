# 1. 开发环境配置

sdk技术问题沟通QQ群：609994083</br>

**注意事项：**

1. **认证取号服务必须打开蜂窝数据流量并且手机操作系统给予应用蜂窝数据权限才能使用**
2. **取号请求过程需要消耗用户少量数据流量（国外漫游时可能会产生额外的费用）**
3. **认证取号服务目前支持中国移动2/3/4G和中国电信4G**

## 1.1. 接入流程

**1.申请appid和appkey**

根据《开发者接入流程文档》，前往中国移动开发者社区（dev.10086.cn)，按照文档要求创建开发者账号并申请appid和appkey，并填写应用的包名和包签名。

**2.申请能力**

应用创建完成后，在能力配置页面上，勾选应用需要接入的能力类型，如一键登录，并配置应用的服务器出口IP地址。（如果在服务端需要用非对称加密方法对一些重要信息进行加密处理，请在能力配置页面填写RSA加密的公钥）

## 1.2. 开发流程

**第一步：下载SDK及相关文档**

请在开发者群或官网下载最新的SDK包

**第二步：搭建开发环境**

jar包集成方式：

1. 在Eclipse/AS中建立你的工程。 
2. 将`*.jar`拷贝到工程的libs目录下，如没有该目录，可新建。
3. 将sdk所需要的资源文件（anim, drawable, drawable-xxhdpi, layout, values文件）从demo工程res-umc目录下的文件添加到项目工程中

aar包集成方式：

1. 在Eclipse/AS中建立你的工程。 
2. 将`*.jar`拷贝到工程的libs目录下，如没有该目录，可新建。


**第三步：开始使用移动认证SDK**

**[1] AndroidManifest.xml设置**

添加必要的权限支持: 

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
```

权限说明：

| 权限                 | 说明                                       |
| -------------------- | ------------------------------------------ |
| INTERNET             | 允许应用程序联网，用于访问网关和认证服务器 |
| READ_PHONE_STATE     | 获取imsi用于判断双卡和换卡                 |
| ACCESS_WIFI_STATE    | 允许程序访问WiFi网络状态信息               |
| ACCESS_NETWORK_STATE | 获取网络状态，判断是否数据、wifi等         |
| CHANGE_NETWORK_STATE | 允许程序改变网络连接状态                   |

**[2] 配置授权登录activity**

开发者根据需要配置横竖屏方向：`android:screenOrientation`
示列代码为`unspecified`（默认值由系统选择显示方向）

```java
<activity
    android:name="com.cmic.sso.sdk.activity.OAuthActivity"
    android:configChanges="orientation|keyboardHidden|screenSize"
    android:screenOrientation="unspecified"
    android:launchMode="singleTop">
</activity>
<!-- required -->
<activity
    android:name="com.cmic.sso.sdk.activity.BufferActivity"
    android:configChanges="orientation|keyboardHidden|screenSize"
    android:screenOrientation="unspecified"
    android:launchMode="singleTop">
</activity>
<!-- required -->
<activity
    android:name="com.cmic.sso.sdk.activity.LoginAuthActivity"
    android:configChanges="orientation|keyboardHidden|screenSize"
    android:screenOrientation="unspecified"
    android:launchMode="singleTop">
</activity>
```

通过以上两个步骤，工程就已经配置完成了。接下来就可以在代码里使用统一认证的SDK进行开发了

**[3] 创建一个AuthnHelper实例**

`AuthnHelper`是SDK的功能入口，所有的接口调用都得通过AuthnHelper进行调用。因此，调用SDK，首先需要创建一个AuthnHelper实例

**方法原型：**

```java
public static AuthnHelper getInstance(Context context)
```

**参数说明：**

| 参数    | 类型    | 说明                                               |
| ------- | ------- | -------------------------------------------------- |
| context | Context | 调用者的上下文环境，其中activity中this即可以代表。 |

**示例代码：**

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }
```

**[4] 实现回调**

所有的SDK接口调用，都会传入一个回调，用于接收SDK返回的调用结果。结果以`JsonObject`的形式传递，`TokenListener`的实现示例代码如下：

```java
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        if (jObj != null) {
            mResultString = jObj.toString();
            mHandler.sendEmptyMessage(RESULT);
            if (jObj.has("token")) {
                mtoken = jObj.optString("token");
            }
        }
    }
};
```

**[5] 混淆策略**

请避免混淆一键登录SDK，在Proguard混淆文件中增加以下配置：

```java
-dontwarn class com.cmic.sso.sdk.**
-keep class com.cmic.sso.sdk.**{*;}
```

<div STYLE="page-break-after: always;"></div>

# 2. 一键登录功能

## 2.1. 准备工作

在中国移动开发者社区进行以下操作：

1. 获得appid和appkey；
2. 勾选一键登录能力；
3. 配置应用服务器的出口ip地址
4. 配置公钥（如果使用RSA加密方式）

## 2.2. 使用流程说明

![](image/login.png)

## 2.3. 取号请求

本方法用于发起取号请求，SDK完成网络判断、蜂窝数据网络切换等操作并缓存凭证scrip。

**取号方法原型：**

```java
public void getPhoneInfo(final String appId, 
                         final String appKey, 
                         final long expiresIn, 
                         final TokenListener listener,
                         final int requestCode)
```

**参数说明：**

| 参数        | 类型          | 说明                                                         |
| :---------- | :------------ | :----------------------------------------------------------- |
| appId       | String        | 应用的AppID                                                  |
| appkey      | String        | 应用密钥                                                     |
| expiresIn   | long          | 设置超时时间，单位ms，设置范围2000-8000                      |
| listener    | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |
| requestCode | int           | 请求标识码。与响应参数中的SDKRequestCode呼应，SDKRequestCode=用户传的requestCode，如果开发者没有传requestCode，那么SDKRequestCode=-1 |

**响应参数**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段           | 类型    | 含义                                                         |
| -------------- | ------- | ------------------------------------------------------------ |
| resultCode     | int     | 接口返回码，“103000”为成功。具体返回码见5.1 SDK返回码        |
| desc           | boolean | 成功标识，true为成功。                                       |
| SDKRequestCode | int     | 响应标识码。与请求参数中的requestCode呼应，SDKRequestCode=用户传的requestCode，如果开发者没有传requestCode，那么SDKRequestCode=-1 |

**示例代码：**

```java
/***
判断和获取READ_PHONE_STATE权限逻辑
***/   

//创建AuthnHelper实例
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }

//实现取号回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};

//调用取号方法
mAuthnHelper.getPhoneInfo(Constant.APP_ID, Constant.APP_KEY, 8000, mListener);
```

## 2.4. 使用短信验证码（可选）

SDK提供短信验证码作为网关取号的补充功能，短验功能只有在网关取号失败时才能使用。

**注意：**

1. 目前短信验证码只支持移动和电信手机号码
2. 无网络时，不提供短验服务
3. 未获取`READ_PHONE_STATE`授权时，不提供短验服务

**短信验证码开关原型：**

```java
public void SMSAuthOn(boolean on) 
```

**参数说明：**

| 字段 | 类型    | 含义                                                       |
| ---- | ------- | ---------------------------------------------------------- |
| on   | boolean | true:允许使用短信验证码</br>false（默认）:不使用短信验证码 |

**短信验证使用场景（SMSAuthOn打开前提）：**

一、开发者调用2.5中的loginAuth授权请求方法失败时，自动跳转到短信验证码页面；

二、开发者在授权页面切换，其中

“切换账号”按钮隐藏时，无法在授权页面跳转到短验页面；

“切换账号”按钮显示时，

1. 当SMSAuthOn设置为打开时，点击“切换账号”，跳转到SDK短验
2. 当SMSAuthOn设置为关闭时，点击“切换账号”，SDK将返回200060返回码，应用根据该返回码自行实现页面的跳转。

![](image/sms_logic.png)

## 2.5. 授权请求

应用调用本方法时，SDK将拉起用户授权页面，用户确认授权后，SDK将返回token给应用客户端。

**授权请求方法原型**

```java
public void loginAuth(final String appId, 
                      final String appKey, 
                      final TokenListener listener
                      final int requestCode)
```

**请求参数**

| 参数        | 类型          | 说明                                                         |
| :---------- | :------------ | :----------------------------------------------------------- |
| appId       | String        | 应用的AppID                                                  |
| appkey      | String        | 应用密钥                                                     |
| listener    | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |
| requestCode | int           | 请求标识码。与响应参数中的SDKRequestCode呼应，SDKRequestCode=用户传的requestCode，如果开发者没有传requestCode，那么SDKRequestCode=-1 |

**响应参数**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段           | 类型   | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| resultCode     | Int    | 接口返回码，“103000”为成功。具体响应码见4.1 SDK返回码        |
| resultDesc     | String | 失败时返回：返回错误码说明                                   |
| authType       | String | 认证类型：</br>0:其他；</br>1:WiFi下网关鉴权；</br>2:网关鉴权；</br>3:短信上行鉴权；</br>7:短信验证码登录 |
| authTypeDec    | String | 认证类型描述，对应authType                                   |
| token          | String | 成功时返回：临时凭证，token有效期2min，一次有效；同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |
| openId         | String | 成功时返回：用户身份唯一标识                                 |
| SDKRequestCode | int    | 响应标识码。与请求参数中的requestCode呼应，SDKRequestCode=用户传的requestCode，如果开发者没有传requestCode，那么SDKRequestCode=-1 |

**示例代码**

```java
/***
判断和获取READ_PHONE_STATE权限逻辑
***/   

//创建AuthnHelper实例
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }

//实现取号回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};

//调用一键登录方法
mAuthnHelper.loginAuth(Constant.APP_ID, Constant.APP_KEY, mListener);
```

## 2.6. 授权页面设计

为了确保用户在登录过程中将手机号码信息授权给开发者使用的知情权，一键登录需要开发者提供授权页登录页面供用户授权确认。开发者在调用授权登录方法前，必须弹出授权页，明确告知用户当前操作会将用户的本机号码信息传递给应用。

### 2.6.1. 页面规范细则

![](image/authPage.png)

![](image/SMSPage.png)



**注意：开发者不得通过任何技术手段，将授权页面的隐私栏、品牌露出内容隐藏、覆盖，对于接入移动认证SDK并上线的应用，我方会对上线的应用授权页面做审查，如果有出现未按要求设计授权页面，将关闭应用的认证取号服务。**

### 2.6.2. 修改授权页面主题

开发者可以通过`setAuthThemeConfig`方法修改授权页面主题

**方法原型：**

```java
public void setAuthThemeConfig(authThemeConfig authThemeConfig)
```

**参数说明**

| 参数            | 类型            | 说明                                                         |
| :-------------- | :-------------- | :----------------------------------------------------------- |
| authThemeConfig | authThemeConfig | 主题配置对象，开发者在authThemeConfig.java类中调用对应的方法配置授权页中对应的元素 |

**authThemeConfig.java配置元素说明：**

*授权页导航栏：*

| 方法                | 说明                   |
| ------------------- | ---------------------- |
| setNavColor         | 设置导航栏颜色         |
| setNavText          | 设置导航栏标题文字     |
| setNavTextColor     | 设置导航栏标题文字颜色 |
| setNavReturnImgPath | 设置导航栏返回按钮图标 |

*授权页logo（logo的层级在次底层，仅次于自定义控件）*

| 方法             | 说明                            |
| ---------------- | ------------------------------- |
| setLogoImgPath   | 设置logo图片                    |
| setLogoWidthDip  | 设置logo宽度                    |
| setLogoHeightDip | 设置logo高度                    |
| setLogoOffsetY   | 设置logo相对于标题栏下边缘y偏移 |
| setLogoHidden    | 隐藏logo                        |

*授权页号码栏：*

| 方法               | 说明                              |
| ------------------ | --------------------------------- |
| setNumberColor     | 设置手机号码字体颜色              |
| setNumFieldOffsetY | 设置号码栏相对于标题栏下边缘y偏移 |

*授权页登录按钮：*

| 方法               | 说明                                |
| ------------------ | ----------------------------------- |
| setLogBtnText      | 设置登录按钮文字                    |
| setLogBtnTextColor | 设置登录按钮文字颜色                |
| setLogBtnImgPath   | 设置授权登录按钮图片                |
| setLogBtnOffsetY   | 设置登录按钮相对于标题栏下边缘y偏移 |

*授权页隐私栏：*

| 方法                | 说明                                             |
| ------------------- | ------------------------------------------------ |
| setClauseOne        | 设置开发者隐私条款1名称和URL(名称，url)          |
| setClauseTwo        | 设置开发者隐私条款2名称和URL(名称，url)          |
| setClauseColor      | 设置隐私条款名称颜色(基础文字颜色，协议文字颜色) |
| setUncheckedImgPath | 设置复选框未选中时图片                           |
| setCheckedImgPath   | 设置复选框选中时图片                             |
| setPrivacyOffsetY   | 设置隐私条款相对于授权页面底部下边缘y偏移        |

*授权页slogan：*

| 方法               | 说明                              |
| ------------------ | --------------------------------- |
| setSloganTextColor | 设置移动slogan文字颜色            |
| setSloganOffsetY   | 设置slogan相对于标题栏下边缘y偏移 |

*短验页：*

| 方法                  | 说明                       |
| --------------------- | -------------------------- |
| setSmsNavText         | 设置短验页的导航栏标题文字 |
| setSmsLogBtnText      | 设置短验页的按钮文字       |
| setSmsLogBtnColor     | 设置短验页的按钮颜色       |
| setSmsLogBtnTextColor | 设置短验页的按钮文字颜色   |

### 2.6.3. 开发者自定义控件

授权页面允许开发者在授权页面titlebar和body添加自定义的控件

注意：自定义的控件不允许覆盖SDK默认的UI

**控件布局方法原型：**

```java
public AuthnHelper addAuthRegistViewConfig(String viewId, 
                        AuthRegisterViewConfig mAuthRegisterViewConfig)
```

**参数说明**

| 参数                    | 类型                   | 说明                                                         |
| :---------------------- | :--------------------- | :----------------------------------------------------------- |
| viewId                  | String                 | 开发者自定义控件名称，如果需要自行通过该控件finish授权页面，名称后缀必须添加umcskd_authority_finish，如title_button_umcskd_authority_finish |
| mAuthRegisterViewConfig | AuthRegisterViewConfig | 配置开发者自定义控件的控件来源、位置和处理逻辑等             |

初始化AuthRegisterViewConfig类时需要先调静态内部类Builder()里面的3个方法：

**setView：**开发者传入自定义的控件，开发者需要提前设置好控件的布局属性，SDK只支持RelativeLayout布局

*setView原型：*

```java
public Builder setView(View view) 
```



**setRootViewId：**设置控件的位置，目前SDK授权页允许在2个位置插入开发者控件

1. RootViewId.ROOT_VIEW_ID_TITLE_BAR，标题栏
2. RootViewId.ROOT_VIEW_ID_BODY，授权页空白处

*setRootViewId原型：*

```java
public Builder setRootViewId(int rootViewId) 
```



**setCustomInterface：**设置控件事件        

*setCustomInterface原型：*                                     

```java
public Builder setCustomInterface(CustomInterface customInterface)
```

**示例1（会finish授权页）：**

```java
//开发者自定义控件，mTitleBtn
private Button myTitleBtn = new Button(getContext());
mAuthnHelper.addAuthRegistViewConfig("title_button_umcskd_authority_finish", new AuthRegisterViewConfig.Builder()
        .setView(myTitleBtn)
        .setRootViewId(AuthRegisterViewConfig.RootViewId.ROOT_VIEW_ID_TITLE_BAR)
        .setCustomInterface(new CustomInterface() {
            @Override
            public void onClick(Context context) {
                Toast.makeText(context, "动态注册的按钮", Toast.LENGTH_SHORT).show();
            }
        })
        .build()
);
```

**示例2：**

```java
//开发者自定义控件，myThirdLoginView
private View myThirdLoginView = new LinearLayout(getContext());
mAuthnHelper.addAuthRegistViewConfig("layout_third_login", new AuthRegisterViewConfig.Builder()
        .setView(myThirdLoginView)
        .setRootViewId(AuthRegisterViewConfig.RootViewId.ROOT_VIEW_ID_BODY)
        .setCustomInterface(new CustomInterface() {
            @Override
            public void onClick(Context context) {
                Toast.makeText(context, "动态注册的按钮", Toast.LENGTH_SHORT).show();
            }
        })
        .build()
);
```

### 2.6.4. finish授权页

SDK允许开发者在授权页面通过自定义控件finish授权页面

具体实现方法：

1. 在调用addAuthRegistViewConfig方法时，viewId参数的命名必须包含"umcskd_authority_finish"如"title_button_umcskd_authority_finish"
2. 开发者在处理完自定义的控件事件后，SDK将自动finish授权页面

## 2.7. 获取手机号码（服务端）

开发者获取token后，需要将token传递到应用服务器，由应用服务器发起获取用户手机号接口的调用。

调用本接口，必须保证：

1. token在有效期内。（2分钟）
2. token还未使用过。
3. 应用服务器出口IP地址在开发者社区中配置正确。
4. 如果使用RSA加密，确保应用的公钥在开发者社区正确填写。

**接口说明：**

请求地址：https://www.cmpassport.com/unisdk/rsapi/loginTokenValidate

协议： HTTPS 

请求方法： POST+json,Content-type设置为application/json

**参数说明：**

1、json形式的报文交互必须是标准的json格式

2、发送时请设置content type为 application/json

3、参数类型都是String

**请求参数**

| 参数                | 是否必填 | 说明                                                         |
| :------------------ | :------: | :----------------------------------------------------------- |
| version             |    是    | 填2.0                                                        |
| msgid               |    是    | 标识请求的随机数即可(1-36位)                                 |
| systemtime          |    是    | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| strictcheck         |    是    | 暂时填写"0"，填写“1”时，将对服务器IP白名单进行强校验（后续将强制要求IP强校验） |
| appid               |    是    | 业务在统一认证申请的应用id                                   |
| expandparams        |    否    | 扩展参数                                                     |
| token               |    是    | 需要解析的凭证值。                                           |
| sign                |    是    | 当**encryptionalgorithm≠"RSA"**时，sign = MD5(appid + version + msgid + systemtime + strictcheck + token + appkey)（注：“+”号为合并意思，不包含在被加密的字符串中），输出32位大写字母；</br>当**encryptionalgorithm="RSA"**，业务端RSA私钥签名（appid+token）, 服务端使用业务端提供的公钥验证签名（公钥可以在开发者社区配置）。 |
| encryptionalgorithm |    否    | 推荐使用。开发者如果需要使用非对称加密算法时，填写“RSA”。（当该值不设置为“RSA”时，执行MD5签名校验） |

**响应参数**

| 参数         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| inresponseto | 对应的请求消息中的msgid                                      |
| systemtime   | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultcode   | 返回码                                                       |
| msisdn       | 表示手机号码，如果加密方式为RSA，应用需要用私钥进行解密      |

## 2.8. 本机号码校验（服务端）

开发者获取token后，需要将token传递到应用服务器，由应用服务器发起本机号码校验接口的调用。

调用本接口，必须保证：

1. token在有效期内（2分钟）
2. token还未使用过
3. 应用服务器出口IP地址在开发者社区中配置正确。

对于本机号码校验，需要注意：

1. 本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。
2. 签订合同后，将不在提供每天免费的测试次数。

**接口说明：**

请求地址： https://www.cmpassport.com/openapi/rs/tokenValidate

协议： HTTPS

请求方法： POST+json,Content-type设置为application/json

**参数说明：**

1、json形式的报文交互必须是标准的json格式

2、发送时请设置content type为 application/json

3、参数类型都是String

**请求参数**

| 参数          | 层级  | 是否必填                     | 说明                                                         |
| ------------- | ----- | ---------------------------- | ------------------------------------------------------------ |
| **header**    | **1** | 是                           |                                                              |
| version       | 2     | 是                           | 版本号,初始版本号1.0,有升级后续调整                          |
| msgId         | 2     | 是                           | 使用UUID标识请求的唯一性                                     |
| timestamp     | 2     | 是                           | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId         | 2     | 是                           | 应用ID                                                       |
| **body**      | **1** | 是                           |                                                              |
| openType      | 2     | 否，requestertype字段为0时是 | 运营商类型：</br>1:移动;</br>2:联通;</br>3:电信;</br>0:未知  |
| requesterType | 2     | 是                           | 请求方类型：</br>0:APP；</br>1:WAP                           |
| message       | 2     | 否                           | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams  | 2     | 否                           | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
| keyType       | 2     | 否                           | 手机号码加密方式：</br>0:默认phonenum采用sha256加密，sign采用HMACSHA256算法</br>1:RSA加密（暂未支持）</br>（注：keyType=1时，phonenum和sign均使用RSA，keyType不填或非1、0时按keyType=0处理） |
| phoneNum      | 2     | 是                           | 待校验的手机号码的64位sha256值，字母大写。（手机号码 + appKey + timestamp， “+”号为合并意思）（注：建议开发者对用户输入的手机号码的格式进行校验，增加校验通过的概率） |
| token         | 2     | 是                           | 身份标识，字符串形式的token                                  |
| sign          | 2     | 是                           | 签名，HMACSHA256( appId + msgId + phonNum + timestamp + token + version)，输出64位大写字母 （注：“+”号为合并意思，不包含在被加密的字符串中，appkey为秘钥, 参数名做自然排序（Java是用TreeMap进行的自然排序）） |

**响应参数**

| 参数         | 层级  | 说明                                                         |
| ------------ | ----- | :----------------------------------------------------------- |
| **header**   | **1** |                                                              |
| msgId        | 2     | 对应的请求消息中的msgid                                      |
| timestamp    | 2     | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId        | 2     | 应用ID                                                       |
| resultCode   | 2     | 平台返回码                                                   |
| **body**     | **1** |                                                              |
| resultDesc   | 2     | 平台返回码                                                   |
| message      | 2     | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| accessToken  | 2     | 使用短验辅助服务的凭证，当resultCode返回为001时，并且该appid在开发者社区配置了短验辅助功能时返回该参数。accessToken有效时间为5min，一次有效。 |
| expandParams | 2     | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |

**请求示例代码：**

```
{
    body =     {
        openType = 1;
        phoneNum =0A2050AC434A32DE684745C829B3DE570590683FAA1C9374016EF60390E6CE76;
        requesterType = 0;
        sign = 87FCAC97BCF4B0B0D741FE1A85E4DF9603FD301CB3D7100BFB5763CCF61A1488;
        token = STsid0000001517194515125yghlPllAetv4YXx0v6vW2grV1v0votvD;
    };
    header =     {
        appId = 3000******76;
        msgId = f11585580266414fbde9f755451fb7a7;
        timestamp = 20180129105523519;
        version = "1.0";
    };
}
```

<div STYLE="page-break-after: always;"></div>

<div STYLE="page-break-after: always;"></div>

## 2.9. 获取网络状态和运营商类型

### 2.9.1. 方法描述

本方法用于获取用户当前的网络环境和运营商

**原型**

```java
public JSONObject getNetworkType(Context context)
```

### 2.9.2. 参数说明

**请求参数**

| 参数    | 类型    | 说明       |
| ------- | ------- | ---------- |
| context | Context | 上下文对象 |

**响应参数**

参数JSONObject，含义如下：

| 参数         | 类型   | 说明                                                         |
| ------------ | ------ | ------------------------------------------------------------ |
| operatorType | String | 运营商类型：</br>1.移动流量；</br>2.联通流量；</br>3.电信流量 |
| networkType  | String | 网络类型：</br>0.未知；</br>1.流量；</br>2.wifi；</br>3.数据流量+wifi |

## 2.10. 删除临时取号凭证

### 2.10.1. 方法描述

开发者取号或者授权成功后，SDK将取号的一个临时凭证缓存在本地，缓存允许用户在未开启蜂窝网络时成功取号。开发者可以使用本方法删除该缓存凭证。

**原型**

```java
public void delScrip()
```



# 3. 服务端接口说明

## 3.1. 获取手机号码接口

业务平台或服务端携带用户授权成功后的token来调用认证服务端获取用户手机号码等信息。

### 3.1.1. 接口说明

**请求地址：**https://www.cmpassport.com/unisdk/rsapi/loginTokenValidate

**协议：** HTTPS 

**请求方法：** POST+json,Content-type设置为application/json

**注意：开发者需到开发者社区填写服务端出口IP地址后才能正常使用**

</br>

### 3.1.2. 参数说明

1、json形式的报文交互必须是标准的json格式

2、发送时请设置content type为 application/json

3、参数类型都是String

**请求参数**

| 参数                | 是否必填 | 说明                                                         |
| :------------------ | :------: | :----------------------------------------------------------- |
| version             |    是    | 填2.0                                                        |
| msgid               |    是    | 标识请求的随机数即可(1-36位)                                 |
| systemtime          |    是    | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| strictcheck         |    是    | 暂时填写"0"，填写“1”时，将对服务器IP白名单进行强校验（后续将强制要求IP强校验） |
| appid               |    是    | 业务在统一认证申请的应用id                                   |
| expandparams        |    否    | 扩展参数                                                     |
| token               |    是    | 需要解析的凭证值。                                           |
| sign                |    是    | 当**encryptionalgorithm≠"RSA"**时，sign = MD5(appid + version + msgid + systemtime + strictcheck + token + appkey)（注：“+”号为合并意思，不包含在被加密的字符串中），输出32位大写字母；</br>当**encryptionalgorithm="RSA"**，业务端RSA私钥签名(appid+token), 服务端使用业务端提供的公钥验证签名（公钥可以在开发者社区配置）。 |
| encryptionalgorithm |    否    | 推荐使用。开发者如果需要使用非对称加密算法时，填写“RSA”。（当该值不设置为“RSA”时，执行MD5签名校验） |

**响应参数**

| 参数         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| inresponseto | 对应的请求消息中的msgid                                      |
| systemtime   | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultcode   | 返回码                                                       |
| msisdn       | 表示手机号码，如果加密方式为RSA，应用需要用私钥进行解密      |

### 3.1.3. 示例

**请求示例**

```
{
    appid = 3000******76; 
    msgid = 335e06a28f064b999d6a25e403991e4c;
    sign = 213EF8D0CC71548945A83166575DFA68;
    strictcheck = 0;
    systemtime = 20180129112955435;
    token = STsid0000001517196594066OHmZvPMBwn2MkFxwvWkV12JixwuZuyDU;
    version = "2.0";
}
```

**响应示例**

```
{
    inresponseto = 335e06a28f064b999d6a25e403991e4c;
    msisdn = 14700000000;
    resultCode = 103000;
    systemtime = 20180129112955477;
}
```

## 3.2. 本机号码校验

开发者获取token后，需要将token传递到应用服务器，由应用服务器发起本机号码校验接口的调用。

调用本接口，必须保证：

1. token在有效期内（2分钟）
2. token还未使用过
3. 应用服务器出口IP地址在开发者社区中配置正确。

对于本机号码校验，需要注意：

1. 本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。
2. 签订合同后，将不在提供每天免费的测试次数。

### 3.2.1. 接口说明

请求地址： https://www.cmpassport.com/openapi/rs/tokenValidate

协议： HTTPS

请求方法： POST+json,Content-type设置为application/json

### 3.2.2. 参数说明

1、json形式的报文交互必须是标准的json格式

2、发送时请设置content type为 application/json

3、参数类型都是String

**请求参数**

| 参数          | 层级  | 约束                         | 说明                                                         |
| ------------- | ----- | ---------------------------- | ------------------------------------------------------------ |
| **header**    | **1** | 必选                         |                                                              |
| version       | 2     | 必选                         | 版本号,初始版本号1.0,有升级后续调整                          |
| msgId         | 2     | 必选                         | 使用UUID标识请求的唯一性                                     |
| timestamp     | 2     | 必选                         | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId         | 2     | 必选                         | 应用ID                                                       |
| **body**      | **1** | 必选                         |                                                              |
| openType      | 2     | 否，requestertype字段为0时是 | 运营商类型：</br>1:移动;</br>2:联通;</br>3:电信;</br>0:未知  |
| requesterType | 2     | 必选                         | 请求方类型：</br>0:APP；</br>1:WAP                           |
| message       | 2     | 否                           | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams  | 2     | 否                           | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
| keyType       | 2     | 否                           | 手机号码加密方式：</br>0:默认phonenum采用sha256加密，sign采用HMACSHA256算法</br>1:RSA加密（暂未支持）</br>（注：keyType=1时，phonenum和sign均使用RSA，keyType不填或非1、0时按keyType=0处理） |
| phoneNum      | 2     | 必选                         | 待校验的手机号码的64位sha256值，字母大写。（手机号码 + appKey + timestamp， “+”号为合并意思）（注：建议开发者对用户输入的手机号码的格式进行校验，增加校验通过的概率） |
| token         | 2     | 必选                         | 身份标识，字符串形式的token                                  |
| sign          | 2     | 必选                         | 签名，HMACSHA256(appId + msgId + phonNum + timestamp + token + version)，输出64位大写字母 （注：“+”号为合并意思，不包含在被加密的字符串中,appkey为秘钥, 参数名做自然排序（Java是用TreeMap进行的自然排序）） |

**响应参数**

| 参数         | 层级  | 约束 | 说明                                                         |
| ------------ | ----- | :--- | :----------------------------------------------------------- |
| **header**   | **1** | 必选 |                                                              |
| msgId        | 2     | 必选 | 对应的请求消息中的msgid                                      |
| timestamp    | 2     | 必选 | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId        | 2     | 必选 | 应用ID                                                       |
| resultCode   | 2     | 必选 | 平台返回码                                                   |
| **body**     | **1** | 必选 |                                                              |
| resultDesc   | 2     | 必选 | 平台返回码                                                   |
| message      | 2     | 否   | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| accessToken  | 2     | 否   | 使用短验辅助服务的凭证，当resultCode返回为001时，并且该appid在开发者社区配置了短验辅助功能时返回该参数。accessToken有效时间为5min，一次有效。 |
| expandParams | 2     | 否   | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |

### 3.2.3. 示例

```
{
    body =     {
        openType = 1;
        phoneNum =0A2050AC434A32DE684745C829B3DE570590683FAA1C9374016EF60390E6CE76;
        requesterType = 0;
        sign = 87FCAC97BCF4B0B0D741FE1A85E4DF9603FD301CB3D7100BFB5763CCF61A1488;
        token = STsid0000001517194515125yghlPllAetv4YXx0v6vW2grV1v0votvD;
    };
    header =     {
        appId = 3000******76;
        msgId = f11585580266414fbde9f755451fb7a7;
        timestamp = 20180129105523519;
        version = "1.0";
    };
}
```

<div STYLE="page-break-after: always;"></div>

# 4. 返回码说明

## 4.1. SDK返回码

| 返回码 | 返回码描述                                                   |
| ------ | ------------------------------------------------------------ |
| 103000 | 成功                                                         |
| 102102 | 网络异常                                                     |
| 102507 | 登录超时（授权页点登录按钮时）                               |
| 103101 | appkey错误                                                   |
| 103102 | 包签名错误                                                   |
| 103108 | 短信验证码错误                                               |
| 103109 | 短信验证码校验超时                                           |
| 103111 | 网关IP错误（运营商误判）                                     |
| 103119 | appid不存在                                                  |
| 103125 | 短验下发时，手机号填写格式错误                               |
| 103203 | 缓存失效                                                     |
| 103211 | 其他错误                                                     |
| 103901 | 短验下发次数已达上限（5次/min,10次/day）                     |
| 103902 | scrip失效                                                    |
| 103911 | token请求过于频繁，10分钟内获取token且未使用的数量不超过30个 |
| 105001 | 联通取号失败                                                 |
| 105002 | 移动取号失败                                                 |
| 105003 | 电信取号失败                                                 |
| 105021 | 已达当天取号限额                                             |
| 105302 | appid不在白名单                                              |
| 200005 | 用户未授权（READ_PHONE_STATE）                               |
| 200010 | 取号请求时获取imsi失败                                       |
| 200020 | 用户取消登录                                                 |
| 200021 | 数据解析异常                                                 |
| 200022 | 无网络状态                                                   |
| 200023 | 请求超时                                                     |
| 200024 | 数据网络切换失败                                             |
| 200025 | 未知错误一般出现在线程捕获异常，请配合异常打印分析           |
| 200026 | 输入参数错误                                                 |
| 200027 | 未开启数据网络                                               |
| 200038 | 电信重定向失败                                               |
| 200039 | 电信取号接口返回失败                                         |
| 200040 | UI资源加载异常                                               |
| 200048 | 授权请求时获取imsi失败                                       |
| 200050 | EOF异常                                                      |
| 200060 | 切换账号（未使用SDK短验时返回）                              |
|        |                                                              |

## 4.2. 获取手机号码接口返回码

| 返回码 | 返回码描述               |
| ------ | ------------------------ |
| 103000 | 返回成功                 |
| 103101 | 签名错误                 |
| 103113 | token内容错误            |
| 103119 | appid不存在              |
| 103133 | sourceid不合法           |
| 103211 | 其他错误                 |
| 103412 | 无效的请求               |
| 103414 | 参数校验异常             |
| 103811 | token为空                |
| 104201 | token失效或不存在        |
| 105018 | 用户权限不足             |
| 105019 | 应用未授权（开发者社区） |

## 4.3. 本机号码校验接口返回码

| 返回码 | 说明                       |
| ------ | -------------------------- |
| 000    | 是本机号码（纳入计费次数） |
| 001    | 非本机号码（纳入计费次数） |
| 002    | 取号失败                   |
| 003    | 调用内部token校验接口失败  |
| 004    | 加密手机号码错误           |
| 102    | 参数无效                   |
| 124    | 白名单校验失败             |
| 302    | sign校验失败               |
| 303    | 参数解析错误               |
| 606    | 验证Token失败              |
| 999    | 系统异常                   |
| 102315 | 次数已用完                 |


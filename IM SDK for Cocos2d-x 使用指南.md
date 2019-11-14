# IM SDK for Cocos2d-x 使用指南

## 导入IM SDK
### Android
1. 将`yim`复制到Cocos2d-x开发工具生成的游戏目录根目录下。

2. 对`proj.android/jni/Android.mk`文件进行如下修改，**仅需修改标注部分，其余部分无需处理。**

    ``` mk

    LOCAL_PATH := $(call my-dir)

    include $(CLEAR_VARS)

    $(call import-add-path,$(LOCAL_PATH)/../../cocos2d)
    $(call import-add-path,$(LOCAL_PATH)/../../cocos2d/external)
    $(call import-add-path,$(LOCAL_PATH)/../../cocos2d/cocos)

    LOCAL_MODULE := cocos2dcpp_shared

    LOCAL_MODULE_FILENAME := libcocos2dcpp

    LOCAL_SRC_FILES := hellocpp/main.cpp \
       ../../Classes/Helloworld.cpp

    LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../Classes  \
    # #########添加如下代码,添加头文件路径.注意上一行结尾处添加换行符\########
    $(LOCAL_PATH)/../../yim/include \
    # ###############################################################

    # _COCOS_HEADER_ANDROID_BEGIN
    # _COCOS_HEADER_ANDROID_END

    LOCAL_STATIC_LIBRARIES := cocos2dx_static
    # #############添加如下代码,链接动态库##############################
    LOCAL_SHARED_LIBRARIES := YIM
    # ###############################################################

    #_COCOS_LIB_ANDROID_BEGIN
    #_COCOS_LIB_ANDROID_END

    include $(BUILD_SHARED_LIBRARY)
    ####################### 包含YMSDK子配置 #########################
    include $(LOCAL_PATH)/../../yim/lib/android/Android.mk
    ################################################################

    $(call import-module,.)

    # _COCOS_LIB_IMPORT_ANDROID_BEGIN
    # _COCOS_LIB_IMPORT_ANDROID_END
    ```

3. 若不需要语音转文字功能，需要对`yim/lib/android/Android.mk`文件进行如下修改：

    ``` mk
    LOCAL_PATH := $(call my-dir)

    include $(CLEAR_VARS)

    LOCAL_MODULE := YIM
    LOCAL_SRC_FILES := $(TARGET_ARCH_ABI)/libyim.so \

    include $(PREBUILT_SHARED_LIBRARY)

    ############## 不使用翻译可以删除下面内容 ##############
    include $(CLEAR_VARS)

    LOCAL_MODULE := YIMM
    LOCAL_SRC_FILES := $(TARGET_ARCH_ABI)/libmsc.so \                       
                       
    include $(PREBUILT_SHARED_LIBRARY)
    
    
    include $(CLEAR_VARS)
    LOCAL_MODULE := YIMNLS
    LOCAL_SRC_FILES := $(TARGET_ARCH_ABI)/libnlscppsdk.so \                       
                       
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE := YIMUSC
    LOCAL_SRC_FILES := $(TARGET_ARCH_ABI)/libuscasr.so \
                       
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE := YIMU
    LOCAL_SRC_FILES := $(TARGET_ARCH_ABI)+/libuuid.so \
                                             
    include $(PREBUILT_SHARED_LIBRARY)    
    
    ####################################################
    ```

4. 将 `yim/lib/android/` 目录中如下文件复制到`proj.android/libs/`目录下。
>`yim.jar`  
>`Msc.jar`  
>`alisr.jar`  
>`usc.jar`  
>`Sunflower.jar`  
>`android-support-v4.jar`  


5. 修改`proj.android/AndroidManifest.xml`文件，确保声明了如下权限：

``` xml
  <!-- 添加到application节点内 -->
  <application xxxx>
      <receiver android:name="com.youme.im.NetworkStatusReceiver" android:label="NetworkConnection" >
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            </intent-filter>
      </receiver>
  </application>
  <!-- 添加到跟application平级 -->
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
  <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.READ_PHONE_STATE" />
  <uses-permission android:name="android.permission.RECORD_AUDIO" />
  <!-- 获取地理位置的权限，可选 -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

6. 打开Eclipse，导入上一步的Android工程，在项目中第一个启动的`AppActivity`中的`onCreate`中增加以下Java代码：
    `注意必须把游密的初始化加到第一行，否则可能出现部分机型加载so不正常`

    ``` java
    @Override
    protected void onCreate(Bundle bundle)
    {
       //下面的两个调用顺序不能错
        com.youme.im.IMEngine.init(this);
        super.onCreate(bundle);           //super.onCreate是默认已经有的，不用再复制
    }
    ```

7. 如果打包或生成APK时需要混淆，则需要在`proguard.cfg`文件中添加如下代码：

    ``` shell
    -keep class com.youme.**
    -keep class com.youme.**
    -keep class com.iflytek.**
    -keepattributes Signature
    ```

### iOS
1. 将`yim`复制到Cocos2d-x开发工具生成的游戏目录 **根目录下**  
2. 在`Build Settings -> Search Paths -> Header Search Paths`中添加头文件路径：`../yim/include/`  
3. 在`Build Settings -> Search Paths -> Framework Search Paths`中添加框架文件路径：`../yim/lib/ios/`  
4. 在`Build Settings -> Search Paths -> Library Search Paths`中添加库文件路径：`../yim/lib/ios/`  
5. 在`Build Phases  -> Link Binary With Libraries`添加如下依赖库：  
 >`libc++.1.tbd`  
 >`libsqlite3.0.tbd`  
 >`libresolv.9.tbd`  
 >`SystemConfiguration.framework`  
 >`libYouMeCommon.a`  
 >`libyim.a`  
 >`libz.tbd`  
 >`CoreTelephony.framework`  
 >`AVFoundation.framework`  
 >`AudioToolbox.framework`  
 >`CoreLocation.framework`  
 >`iflyMSC.framework`       //如果是带语音转文字的sdk需要添加。  
 >`AliyunNlsSdk.framework`  //如果是带语音转文字的sdk需要添加，该库是动态库，需要在`General -> Embedded Binaries`也进行添加。  
 >`USCModule.framework`     //如果是带语音转文字的sdk需要添加。  
 >
 
 

6. **需要为iOS10以上版本添加录音权限配置**
iOS 10 使用录音权限，需要在`info`新加`Privacy - Microphone Usage Description`键，值为字符串，比如“语音聊天需要录音权限”。
首次录音时会向用户申请权限。配置方式如图(选择Private-Microphone Usage Description)。
![iOS10录音权限配置](https://www.youme.im/doc/images/im_iOS_record_config.jpg)

7. **LBS添加获取地理位置权限**
若使用LBS，需要在`info`新加`Privacy - Location Usage Description`键，值为字符串，比如“查看附近的玩家需要获取地理位置权限”。首次使用定位时会向用户申请权限，配置方式如上图录音权限。

### Windows
1. 将`yim`复制到Cocos2d-x开发工具生成的游戏目录<b>根目录下</b>

2. 在项目 -> 属性 -> C/C++ -> 常规 -> 附加包含目录 中添加`..\yim\include`

3. 拷贝yim/lib/x86 目录下的` msc.dll， yim.dll， yim.lib， libusc.dll， jsoncpp-0.y.z.dll， libeay32.dll， nlscppsdk.dll， pthreadVC2.dll， ssleay32.dll`拷贝到执行目录(如果是64位机器，使用yim/lib/x64)

4. 在项目->属性->链接器->输入->附加依赖库 中添加 `yim.lib`（该库是release模式下的  如果运行的时候是debug模式请添加yim_Debug.lib,如果不存在yim_Debug.lib则代表yim.lib通用）

## 特别注意事项
* C++版本的回调通知都是通过子线程调用，所以在收到回调后，不能直接操作UI，需要回到主线程后操作UI
* 重连有可能失败，所以一般收到OnLogout的通知后需要使用者判断游戏是否还在线，如果游戏还在线，就重新调用 登录->进频道 接口
* 初始化接口整个运行期间，只能调用一次
* 不同设备或者不同的运行期，消息id可能重复
* 返回值为YIMErrorcode的接口，如果返回了非YIMErrorcode_Success表示接口调用失败，就不会再有回调通知（如果是有回调的接口）
* 请务必使用游密SDK提供的`android-support-v4.jar` ，这样可以兼容`targetSdkVersion="23"`及其以上版本的录音授权模式

## 基本接口使用时序图
![基本接口使用时序图](https://youme.im/doc/images/im_sdk_flow_client_cpp.png)


## 初始化
### 包含SDK头文件

``` c
#include "YIM.h"
```

### 数据类型
SDK 封装了Android、iOS、windows平台常用数据类型。开发者需要了解字符串和整数封装后的类型`XString`和`XUINT64`：
> `XString`类型对应Android、iOS的`std::string`类型与windows的`std::wstring`类型
> `XCHAR`类型对应Android、iOS的`char`类型与windows的`wchar_t`类型
> `XUINT64`类型对应标准C++的`unsigned long long`类型

### SDK对象
IM SDK 功能主要类分为：SDK管理器，消息发送管理器，频道管理，具体的含义如下：
>`YIMManager`:SDK管理器，负责初始化、登录、登出等功能
>`YIMMessageManager`：消息发送管理器，负责发送文本、自定义等不同类型消息
>`YIMLocationManager`：地理位置管理器
>`YIMChatRoomManager`：频道管理器，负责创建聊天频道
>`YIMUserProfileManager`：用户信息管理器，负责设置、获取用户信息
>`YIMFriendManager`：好友关系管理器，负责管理好友添加、删除
>`IYIMLoginCallback`：登录登出回调虚基类，负责处理登录、登出结果
>`IYIMMessageCallback`：消息回调虚基类，负责处理消息收发操作结果
>`IYIMChatRoomCallback`：聊天室管理回调虚基类，负责处理聊天室管理操作结果
>`IYIMDownloadCallback`：下载回调虚基类，负责处理下载完成操作结果
>`IYIMContactCallback`：最近私聊用户列表查询结果通知基类
>`IYIMLocationCallback`：LBS相关回调，负责位置相关处理
>`IYIMAudioPlayCallback`：语音播放回调基类
>`IYIMReconnectCallback`：重连回调基类
>`IYIMNoticeCallback`：公告回调基类
>`IYIMReconnectCallback`：重连回调基类
>`IYIMFriendCallback`：好友管理回调基类
>`IYIMUserProfileCallback`：用户信息管理回调基类

### 获取SDK管理器
在工程的`AppDelegate`里面的以下方法中，添加SDK初始化代码。初始化SDK前需要创建SDK管理器`YIMManager`，通过它获取聊天室、消息管理器。

- 原型：

  ``` c
      // 获取游密IM管理器实例，是单例，多次调用仍然是返回同一个对象指针
      public static YIMManager* CreateInstance ();
  ```
  
- 返回：
  `YIMManager`实例

- 示例：

  ``` c
    //本文档为方便演示接口使用示例。使用了一个类MyIM包装SDK管理器，以后就可以通过这个`MyIM::Inst()`引用IMSDK管理器。
    YIMManager * MyIM::Inst()
    {
        if(NULL == _instance)
        {
            _instance = YIMManager::CreateInstance();
        }
        return _instance;
    }
  ```

  `MyIM::Inst()`获得IMSDK管理器之后，就可以通过它获取聊天室，消息管理器。

- 示例：

  ``` cpp
      // 获取频道管理器
      MyIM::Inst()->GetChatRoomManager();
      // 获取消息发送管理器
      MyIM::Inst()->GetMessageManager();
      // 获取地理位置管理器
      MyIM::Inst()->GetLocationManager();
      // 获取用户信息管理器
      MyIM::Inst()->GetUserProfileManager();
      // 获取好友关系管理器
      MyIM::Inst()->GetFriendManager();
  ```

### 初始化SDK
获得SDK管理器之后，紧跟着要调用`YIMManager`的`Init`接口进行初始化。初始化的函数主要功能是初始化IM SDK引擎。初始化接口的输入参数`appKey`和`appSecurity`需要根据实际申请得到的值进行替换, `packageName`目前传入空字符串即可。

- 接口：

  ``` c
      YIMErrorcode Init(const XCHAR* appKey, const XCHAR* appSecurity, const XCHAR* packageName);
  ```
  
- 参数：
  `appKey`: 用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://account.youme.im/login)获取、查看
  `appSecurity`: 用户游戏产品的密钥，可以在[游密官网](https://account.youme.im/login)获取、查看
  `packageName`：游戏包名
  
- 返回：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)

- 示例：

  ``` cpp
      MyIM::Inst()->Init("YoumeAppkey"," YoumeAppSecurity", "");
  ```

### 设置监听
SDK内部所有的较长耗时的接口调用都会采用异步回调的方式返回结果，所以需要开发者实现:
>`IYIMLoginCallback`
>`IYIMMessageCallback`
>`IYIMChatRoomCallback`
>`IYIMDownloadCallback`
>`IYIMContactCallback`
>`IYIMAudioPlayCallback`
>`IYIMLocationCallback`
>`IYIMReconnectCallback`
>`IYIMNoticeCallback`
>`IYIMFriendCallback`
>`IYIMUserProfileCallback`

然后把它们的实现对象设置到SDK管理器里。如果不设定就收不到相应事件的通知。建议初始化和设置监听在一起完成。

- 接口原型：

  ``` cpp
      //登录和登出
      void SetLoginCallback(IYIMLoginCallback* pCallback);
      //设置消息回调
      void SetMessageCallback(IYIMMessageCallback* pCallback);
      //设置聊天室回调
      void SetChatRoomCallback(IYIMChatRoomCallback* pCallback);
      //设置下载回调
      void SetDownloadCallback(IYIMDownloadCallback* pCallback);
      //设置联系人回调
      void SetContactCallback(IYIMContactCallback* pCallback);
      //设置语音播放回调
      void SetAudioPlayCallback(IYIMAudioPlayCallback* pCallback);
      // 设置地理位置回调
      void SetLocationCallback(IYIMLocationCallback* pCallback);
      // 设置通知回调
	   void SetNoticeCallback(IYIMNoticeCallback* pCallback);
      // 设置重连回调
      void SetReconnectCallback(IYIMReconnectCallback* pCallback);
      // 设置好友回调
	   void SetFriendCallback(IYIMFriendCallback* pCallback);
	   //设置用户信息回调
     void SetUserProfileCallback(IYIMUserProfileCallback* pCallback);
  ```
  
- 示例：

  ``` cpp
      //本文档示例中使用class MyIM继承自这4个回调类。实现它们回调接口。
      class MyIM : public IYIMLoginCallback, IYIMMessageCallback, IYIMChatRoomCallback, IYIMDownloadCallback, IYIMContactCallback, IYIMAudioPlayCallback, IYIMLocationCallback, IYIMNoticeCallback, IYIMReconnectCallback, IYIMFriendCallback, IYIMUserProfileCallback
      {
         //建议初始化和设置监听在一起完成
         public: void Start ()
         {
            MyIM::Inst()->SetLoginCallback(this);
            MyIM::Inst()->SetMessageCallback(this);
            MyIM::Inst()->SetChatRoomCallback(this);
            MyIM::Inst()->SetDownloadCallback(this);
            MyIM::Inst()->SetContactCallback(this);
            MyIM::Inst()->SetAudioPlayCallback(this);
            MyIM::Inst()->SetLocationCallback(this);
            MyIM::Inst()->SetNoticeCallback(this);
            MyIM::Inst()->SetReconnectCallback(this);
            MyIM::Inst()->SetFriendCallback(this);
            MyIM::Inst()->SetUserProfileCallback(this);
            
            MyIM::Inst()->Init("YoumeAppkey"," YoumeAppSecurity", "");
         }

      }
   ```
   
### 重连回调
#### 开始重连回调
- 原型：

  ``` c
  class IYIMReconnectCallback
  {
    public:  
      //开始重连
    virtual void OnStartReconnect() {}
  };    
  ```
     
#### 重连结果回调
- 原型：

  ``` c
  class IYIMReconnectCallback
  {
    public: 
      //重连结果回调
    virtual void OnRecvReconnectResult(ReconnectResult result) {}
  };  
  ```
  
- 参数：
  `result`：重连结果，枚举类型，
            RECONNECTRESULT_SUCCESS=0 //重连成功
            RECONNECTRESULT_FAIL_AGAIN=1 //重连失败，再次重连
            RECONNECTRESULT_FAIL=2 //重连失败    
    
### 录音音量回调
音量值范围:0~1, 频率:1s ios 约2次，android 约8次

- 原型：

  ``` c
  class IYIMMessageCallback
  {
    public:
      //音量回调,音量值范围:0~1
    virtual void OnRecordVolumeChange(float volume) {}
  };  
  ```   
  
- 参数：
  `volume`：音量值。

## 用户管理 
### 用户登录
完成以上的步骤后就可以使用IM功能了，IM用户登录IM后台服务器后即可以正常收发消息。通过`YIMManager`的`Login`接口登录IM，需要用户提供用户名、密码。登录为异步过程，通过回调函数返回是否成功，成功后方能进行后续操作。
用户首次登录会自动注册，如果后台已存在此用户名，则会校验密码是否与第一次相同（用户名和密码的格式见下方相关参数说明）。

- 原型：

  ``` c      
      YIMErrorcode Login(const XCHAR* userID, const XCHAR* passwd ,const XCHAR* token);
  ```
  
- 参数说明：
  `userID `：用户ID，由调用者分配，不可为空字符串，只可由字母或数字或下划线组成，长度限制为255字节
  `passwd`：用户密码，不可为空字符串，如无特殊要求可以设置为固定字符串
  `token`：使用服务器token验证模式时使用该参数，否则使用空字符串：""，由restAPI获取token值

- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。
  
- 异步回调接口：
  
  ``` c
  class IYIMLoginCallback
  {
    public:
    virtual void OnLogin(YIMErrorcode errorcode, const XString& userID) {}
  };  
  ```  
  
- 回调参数：  
  `errorcode`：错误码  
  `userID`：用户ID  
  
- 示例：

  ``` c
      YIMErrorcode ymErrorcode = MyIM::Inst()->Login("123456", "123456", "" );
      if(YIMErrorcode_Success == ymErrorcode)
      {
         printf("call Login()  success");
      }
      else
      {
         printf("call Login fail,不会有回调通知，errorcode is %d",  ymErrorcode);
      }
  ```

### 用户登出
如用户主动退出或需要进行用户切换，则需要调用登出操作，通过`YIMManager`的`Logout`接口登出。

- 原型：

  ``` c      
      YIMErrorcode Logout();
  ```

- 参数：
  无。

- 返回：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。
  
- 异步回调接口：
  
  ``` c
  class IYIMLoginCallback
  {
    public:
    virtual void OnLogout(YIMErrorcode errorcode) {}
  };  
  ```  
  
- 回调参数：
  `errorcode`：错误码  

- 示例：

  ``` c
      //登出
      YIMErrorcode ymErrorcode = MyIM::Inst()->Logout();
      if(YIMErrorcode_Success == ymErrorcode)
      {
         printf("Logout()  Success");
      }
      else
      {
         printf("Logout()  fail，errorcode is %d",  ymErrorcode);
      }
  ```  
  
### 被用户踢出通知
同一个用户ID在多台设备上登录时，后登录的会把先登录的踢下线，收到OnKickOff()通知。

- 原型：
   
  ``` c
  class IYIMLoginCallback
  {
    public:
    virtual void OnKickOff() {}
  };  
  ```  
    
### 用户信息
#### 设置用户的详细信息
通过`YIMManager`的`SetUserInfo`接口设置用户信息后，在游密消息管理后台才能看到消息发送者的详细信息。

- 原型：

  ``` c      
      YIMErrorcode SetUserInfo(const XCHAR* userInfo);
  ```

- 参数：
  `userInfo`： 用户信息字符串， 格式为json {"nickname":"","server_area_id":"","server_area":"","location_id":"","location":"","level":"","vip_level":""} (可以添加其他字段)
  `nickName`：用户昵称
  `server_area_id`：游戏服ID
  `server_area`：游戏服名称
  `location_id`：游戏大区ID
  `location`：游戏大区名  
  `level`：角色等级
  `vip_level`：玩家VIP等级
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  
  
#### 获取指定用户的详细信息
可通过`YIMManager`的`GetUserInfo`接口获取指定用户的详细信息。

- 原型：

  ``` c        
      YIMErrorcode GetUserInfo(const XCHAR* userID);
  ```

- 参数：
  `userID`： 要查询的用户ID
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。

- 异步回调接口：

  ``` c
  class IYIMContactCallback
  {
    public:
      //用户信息字符串， 格式为json {"nickname":"","server_area_id":"","server_area":"","location_id":"","location":"","level":"","vip_level":""}
    virtual void OnGetUserInfo(YIMErrorcode errorcode, const XString& userID, const XString& userInfo) {}
  };  
  ```
    
- 参数：
  `errorcode`：错误码
  `userID`：用户ID
  `userInfo`：用户详细信息    

#### 查询用户在线状态
可通过`YIMManager`的`QueryUserStatus`接口查询用户的在线状态。

- 原型：

  ``` c      
      YIMErrorcode QueryUserStatus(const XCHAR* userID)
  ```

- 参数：
  `userID`：要查询的用户ID

- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。

- 异步回调接口：

  ``` c  
  class IYIMContactCallback
  {
    public:    
    virtual void OnQueryUserStatus(YIMErrorcode errorcode, const XString&  userID, YIMUserStatus status) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码，查询请求是否成功的通知 
  `userID`：查询的用户ID  
  `status`：登录状态，枚举类型，
            UserStatus.STATUS_ONLINE=0  //在线
            UserStatus.STATUS_OFFLINE=1 //离线  

## 频道管理 
### 加入频道
通过`YIMChatRoomManager`的`JoinChatRoom`接口加入聊天频道，如果频道不存在则后台自动创建。有了这个ID就可以收发频道消息。开发者要保证频道号全局唯一，以避免用户进入错误频道。结果需要在开发者实现`IYIMChatRoomCallback`的`OnJoinChatRoom`里处理。

- 原型：

  ``` c
      YIMErrorcode JoinChatRoom(const XCHAR* chatRoomID);
  ```
  
- 参数：
  `chatRoomID `：请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 异步回调接口：

  ``` c
  class IYIMChatRoomCallback
  {
    public:
    virtual void OnJoinChatRoom(YIMErrorcode errorcode, const XString&  strChatRoomID) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `strChatRoomID`：频道ID   
  
- 示例：

  ``` c
      YIMErrorcode ymErrorcode;
      ymErrorcode = MyIM::Inst()->GetChatRoomManager()->JoinChatRoom("123456");
      if( ymErrorcode != YIMErrorcode_Success )
      {
         printf("OnJoinChatRoom()  fail，errorcode is %d",  ymErrorcode);
      }

      #异步回调处理
      void MyIM::OnJoinChatRoom(YouMe.YIMErrorcode errorcode,
                               const XString&  strChatRoomID);
      {
         if( errorcode == YIMErrorcode_Success )
         {
            printf("join chat room success.");
         }
      }
  ```

### 离开频道
通过`YIMChatRoomManager`的`LeaveChatRoom`接口离开频道。

- 原型：

  ``` c
      YIMErrorcode LeaveChatRoom(const XCHAR* strChatRoomID);
  ```

- 参数：
  `strChatRoomID `：请求离开的频道ID
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  
  
- 异步回调接口：

  ``` c
  class IYIMChatRoomCallback
  {
    public:
    virtual void OnLeaveChatRoom(YIMErrorcode errorcode, const XString& chatRoomID) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `chatRoomID`：频道ID   
  
- 示例：

  ``` c
      YIMErrorcode ymErrorcode;
      ymErrorcode = MyIM::Inst()->GetChatRoomManager()->LeaveChatRoom("123456");
      if(ymErrorcode != YIMErrorcode_Success)
      {
         printf("LeaveChatRoom()  fail，errorcode is %d",  ymErrorcode);
      }

      #异步回调处理
      void MyIM::OnLeaveChatRoom(YIMErrorcode errorcode, const XString&  chatRoomID);
      {
         if( errorcode == YIMErrorcode_Success )
         {
            printf("leave chat room success.");
         }
      }
  ```
    
### 离开所有频道
通过`YIMChatRoomManager`的`LeaveAllChatRooms`接口离开所有频道。

- 原型：

  ``` c
      YIMErrorcode LeaveAllChatRooms();
  ```
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 异步回调接口：

  ``` c 
  class IYIMChatRoomCallback
  {
    public:   
    virtual void OnLeaveAllChatRooms(YIMErrorcode errorcode) {}
  };      
  ```
    
- 参数：
  `errorcode `：错误码
  
### 用户进出频道通知
此功能默认不开启，需要的请联系我们开启此服务。联系我们，可以通过专属游密支持群或者技术支持的大群。

小频道(小于100人)内，当其他用户进入频道，会收到`OnUserJoinChatRoom`通知

- 原型：

  ``` c
  class IYIMChatRoomCallback
  {
    public:
    virtual void OnUserJoinChatRoom(const XString& chatRoomID, const XString& userID) {}
  };  
  ```

- 参数：
  `chatRoomID `：频道ID
  `userID`：用户ID


小频道(小于100人)内，当其他用户退出频道，会收到`OnUserLeaveChatRoom`通知

- 原型：

  ``` c
  class IYIMChatRoomCallback
  {
    public:
    virtual void OnUserLeaveChatRoom(const XString& chatRoomID, const XString& userID) {}
  };  
  ```

- 参数：
  `chatRoomID `：频道ID
  `userID`：用户ID

### 获取房间成员数量
可通过`YIMChatRoomManager`的`GetRoomMemberCount`接口获取进入频道的用户数量。

- 原型：

  ``` c
      YIMErrorcode GetRoomMemberCount(const XCHAR* chatRoomID);
  ```
  
- 参数：
  `chatRoomID `：频道ID(已成功加入此频道才能获取该频道的人数)
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 异步回调接口：

  ``` c
  class IYIMChatRoomCallback
  {
    public:
    virtual void OnGetRoomMemberCount(YIMErrorcode errorcode, const XString& chatRoomID, unsigned int count) {}
  };  
  ```
    
- 回调参数：
  `errorcode `：错误码
  `chatRoomID`：频道ID
  `count`：频道中的成员数量

## 消息管理 
### 接收消息
通过`IYIMMessageCallback`的`OnRecvMessage`接口被动接收消息。需要开发者实现。

- 原型：

  ``` c
  class IYIMMessageCallback
  {
    public:
      //接收到用户发来的消息
    virtual void OnRecvMessage(std::shared_ptr<IYIMMessage> pMessage)  {}
  };  
  ```
  
- 参数：
  `pMessage`：消息
  
- 备注：  
  `IYIMMessage` 消息基类成员方法如下：
  `GetSenderID()`：消息发送者ID
  `GetReceiveID()`：消息接收者ID
  `GetChatType()`：聊天类型，私聊/频道聊天
  `GetMessageID()`：消息ID
  `GetMessageBody()->GetMessageType()`：消息类型，枚举类型，
                 MessageBodyType_Unknow = 0,         //未知类型
		            MessageBodyType_TXT = 1,            //文本消息
		            MessageBodyType_CustomMesssage = 2, //自定义消息
		            MessageBodyType_Emoji = 3,          //表情
		            MessageBodyType_Image = 4,          //图片
		            MessageBodyType_Voice = 5,          //语音
		            MessageBodyType_Video = 6,          //视频
		            MessageBodyType_File = 7,           //文件
		            MessageBodyType_Gift = 8            //礼物
  `GetCreateTime()`：消息发送时间 
  `GetDistance()`：若是附近频道的消息，此值是该消息用户与自己的地理位置距离
  `IsRead()`：消息是否已读
  
  `IYIMMessageBodyText`继承`IYIMMessageBodyBase`类，类成员如下：
  `GetMessageContent()`：文本消息内容
  `GetAttachParam()`： 发送文本附件信息
  
  `IYIMMessageBodyCustom`继承`IYIMMessageBodyBase`类，类成员如下：
  `GetCustomMessage()`：自定义消息内容
  
  `IYIMMessageBodyFile`继承`IYIMMessageBodyBase`类，类成员如下：
  `GetFileName()`：文件名
  `GetFileType()`：文件类型，枚举类型，
                  FileType_Other = 0   //其它
                  FileType_Audio = 1   //音频（语音）
                  FileType_Image = 2   //图片
                  FileType_Video = 3   //视频
  `GetFileSize()`：文件大小
  `GetFileExtension()`：文件扩展信息
  `GetExtraParam()`：附加信息
  `GetLocalPath()`：文件路径
  
  `IYIMMessageGift`继承`IYIMMessageBodyBase`类，类成员如下：
  `GetAnchor()`：主播ID
  `GetGiftID()`：礼物ID
  `GetGiftCount()`：礼物数量
  `GetExtraParam()`：附加参数
  
  `IYIMMessageBodyAudio`继承`IYIMMessageBodyBase`类，类成员如下：
  `GetText()`：若使用的是语音转文字录音，此值为语音识别的文本内容，否则是""
  `GetAudioTime()`：语音时长（单位：秒）
  `GetExtraParam()`：发送语音时的附加参数 
  `GetFileSize()`：语音文件大小（单位：字节）
  `GetLocalPath()`：语音文件本地存放路径 
  `IsPlayed()`：语音是否已播放，true-已播放，false-未播放 

- 示例：

  ``` c
      void MyIM::OnRecvMessage (std::shared_ptr<IYIMMessage> pMessage)
      {
        YIMMessageBodyType msgType = pMessage->GetMessageBody()->GetMessageType();
        if (msgType ==  YIMMessageBodyType::MessageBodyType_TXT)
        {
            // 文本消息
            IYIMMessageBodyText * pMsgText =  (IYIMMessageBodyText*)pMessage->GetMessageBody();
            printf("RecvMessage text is %s send is %s,  RevID is %s",
                    pMsgText->GetMessageContent(),
                    pMessage->GetReceiveID(),
                    pMessage->GetSenderID());
        }
        else if (msgType == YIMMessageBodyType::MessageBodyType_Voice)
        {
            // 语音消息
            IYIMMessageBodyAudio * pMsgVoice =  (IYIMMessageBodyAudio*)pMessage->GetMessageBody();
            printf("RecvMessage text is %s ,messageid is %llu ,param is %s ,audio time is %d",
                    pMsgVoice->GetText(),
                    pMessage->GetMessageID(),
                    pMsgVoice->GetExtraParam(),
                    pMsgVoice->GetAudioTime() );
            //下载语音文件示例
            MyIM::Inst()->GetMessageManager()->DownloadFile(pMessage->GetMessageID(), "/sdcard/audiocaches/123.wav");
        }else if(msgType == YIMMessageBodyType::MessageBodyType_Gift){
            // 礼物消息
            IYIMMessageGift *pMsgVoice =  (IYIMMessageGift*)pMessage->GetMessageBody();
        }
      }
 ```

### 发送文本消息
可通过`YIMMessageManager`的`SendTextMessage`接口发送文本消息。异步返回结果通过`IYIMMessageCallback`的`OnSendMessageStatus`接口返回。

- 原型：

  ``` c
      YIMErrorcode SendTextMessage(const XCHAR* receiverID, YIMChatType chatType, const XCHAR* text, const XCHAR* attachParam, XUINT64* requestID);
  ```
  
- 参数：
  `receiverID `：接收者ID（用户ID或者频道ID）
  `chatType`：聊天类型
  `text`：聊天内容
  `attachParam`：发送文本附加信息
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  
  
- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  { 
    public:
    virtual void OnSendMessageStatus(XUINT64 requestID, YIMErrorcode errorcode, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime) {}
  };  
  ```
  
- 回调参数：
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `sendTime`：消息发送时间
  `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言（errorcode为禁言才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它（errorcode为禁言才有效）
  `forbidEndTime`：若在频道被禁言，禁言结束时间（errorcode为禁言才有效）

- 示例：

  ``` c
      YIMErrorcode ymErrorcode;
      XUINT64 iRequestID=0;
      ymErrorcode = MyIM::Inst()->GetMessageManager()->SendTextMessage("123456",                                                    YIMChatType::ChatType_PrivateChat,                                                    "Msg content!",                                                    &iRequestID);
     if(ymErrorcode != YIMErrorcode_Success)
     {
        printf("SendTextMessage() call failed，errorcode is %d",  ymErrorcode);
     }
  ```
  
- 异步回调处理：

  ``` c
      void MyIM::OnSendMessageStatus(XUINT64 iRequestID, YIMErrorcode errorcode, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime)
      {
         printf("OnSendMessageStatus request:iRequestID errorcode is %llu, %d:",
            iRequestID,
            errorcode) ;
      }
  ```
    
### 群发文本消息
可通过`YIMMessageManager`的`MultiSendTextMessage`接口群发文本消息，**每次不要超过200个用户**。

- 原型：

  ``` c
      //群发文本消息
      YIMErrorcode MultiSendTextMessage( const std::vector<XString>& vecReceiver, const XCHAR* text)
  ```
  
- 参数：
  `vecReceiver`：接受消息的用户id列表
  `text`：文本消息内容
    
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    

### 发送礼物消息
可通过`YIMMessageManager`的`SendGift`接口给主播发送礼物消息，支持在游密主播后台查看礼物消息信息和统计信息。客户端还是通过`OnRecvMessage`接收消息。

- 原型：

  ``` c
      YIMErrorcode SendGift(const XCHAR* anchorID, const XCHAR* channel, int giftId, int giftCount, const char* extraParam, XUINT64* requestID)
  ```

- 参数：
  `anchorID`：游密后台设置的对应的主播游戏id
  `channel`：主播所进入的频道ID，通过`JoinChatRoom`进入此频道的用户可以接收到消息。
  `giftId`：礼物物品id，特别的是`0`表示只是留言
  `giftCount`：礼物物品数量
  `extraParam`：扩展内容json字符串，目前要求包含如下字段：    `{"nickname":"","server_area":"","location":"","score":"","level":"","vip_level":"","extra":""}`
  `requestID`：消息序列号
    
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。 
  
- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnSendMessageStatus(XUINT64 requestID, YIMErrorcode errorcode, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime) {}
  };  
  ```
  
- 回调参数：
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `sendTime`：消息发送时间
  `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言（errorcode为禁言才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它（errorcode为禁言才有效）
  `forbidEndTime`：若在频道被禁言，禁言结束时间（errorcode为禁言才有效）     

### 发送自定义消息
可通过`YIMMessageManager`的`SendCustomMessage`接口发送自定义消息。异步返回结果通过`IYIMMessageCallback`的`OnSendMessageStatus`接口返回。注意：自定义消息可能包含`\0`终止符，发送和接收时都需要注意。

- 原型：

  ``` c
      YIMErrorcode SendCustomMessage(const XCHAR* receiverID, YIMChatType chatType, const char* content, unsigned int size, XUINT64 *requestID);
  ```
  
- 参数：
  `receiverID`：接收者（用户ID或者频道ID）
  `chatType`：聊天类型，私聊/频道聊天
  `content`：自定义消息内容， 可以包含`\0`终止符。
  `size`：自定义消息内容长度
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 回调通知接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnSendMessageStatus(XUINT64 iRequestID, YIMErrorcode errorcode, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime) {}
  };  
  ```

- 回调参数：
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `sendTime`：消息发送时间
  `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言（errorcode为禁言才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它（errorcode为禁言才有效）
  `forbidEndTime`：若在频道被禁言，禁言结束时间（errorcode为禁言才有效） 

- 示例：

  ``` c
    char buffer[20] = {0};
    buffer[10] = 1;
    buffer[15] = 2;
    YIMErrorcode ymErrorcode;
    XUINT64 iRequestID=0;
    ymErrorcode = MyIM::Inst()->GetMessageManager()->SendCustomMessage("123456",                                                    YIMChatType::ChatType_PrivateChat,                                                    buffer,                                                    20,                                                    &iRequestID);
    if(ymErrorcode != YIMErrorcode_Success)
    {
        printf("SendTextMessage()  fail，errorcode is %d",  ymErrorcode);
    }
  ```
    
- 异步回调处理：

  ``` c
     void myIM::OnSendMessageStatus(XUINT64 iRequestID, YIMErrorcode errorcode, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime)
    {
        printf("OnSendMessageStatus request:iRequestID progress errorcode is %llu,  %d:",
            iRequestID,
            errorcode) ;
    }
  ```
  
### 发送文件消息
#### 发送文件
可通过`YIMMessageManager`的`SendFile`接口发送文件消息。

- 原型：

  ``` c
      YIMErrorcode SendFile(const XCHAR* receiverID, YIMChatType chatType, const XCHAR* filePath, XUINT64* requestID, const XCHAR* extraParam, YIMFileType fileType = FileType_Other);
  ```
  
- 参数：
  `receiverID`：接收方id
  `chatType`：聊天类型，私聊/频道。
  `filePath`：要发送的文件绝对路径，包括文件名
  `extraParam`：额外信息，格式自己定义，自己解析
  `fileType`：文件类型，枚举类型，
              FileType.FileType_Other=0 //其它
              FileType.FileType_Audio=1 //音频
              FileType.FileType_Image=2 //图片
              FileType.FileType_Video=3 //视频
  `requestID`：消息序列号  

- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 回调通知接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnSendMessageStatus(XUINT64 iRequestID, YIMErrorcode errorcode, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime) {}
  };  
  ```

- 回调参数：
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `sendTime`：发送时间
  `isForbidRoom`：是否房间被禁言，true房间被禁言，false玩家被禁言（errorcode为禁言才有效）
  `reasonType`：禁言原因，参见YIMForbidSpeakReason（errorcode为禁言才有效）
  `forbidEndTime`：禁言截止时间戳（errorcode为禁言才有效） 
  
#### 下载文件
接收消息接口参考[收消息](#RecvMessage)通过`GetMessageType()`分拣出文件消息类型：`MessageBodyType_File`，用`GetMessageID()`获得消息ID。然后调用`YIMMessageManager`的`DownloadFile`接口下载文件。

- 原型：

  ``` c
      //下载收到的语音消息文件
      YIMErrorcode DownloadFile(XUINT64 messageID, const XCHAR* savePath);
  ```
  
- 参数：
  `messageID`：消息ID
  `savePath`：文件保存路径（带文件名的全路径），比如"/sdcard/test.wav"如果目录不存在，SDK会自动创建。
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` c  
  class IYIMDownloadCallback
  {
    public:
    virtual void OnDownload( YIMErrorcode errorcode, std::shared_ptr<IYIMMessage> msg, const XString& savePath ) {}
  };
  ```
  
- 回调参数：
  `msg`：消息结构，`IYIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注
  `errorcode`：下载结果错误码
  `savePath`：保存路径，如果没有权限问题，SDK会尝试自动创建  

### 语音消息管理  
语音消息聊天简要流程：

-  调用`SendAudioMessage`方法就开始录音，`StopAudioMessage`表示结束录音并发送，`CancleAudioMessage`表示取消本次录音发送。
-  接收方接收语音消息通知后，调用方控制是否下载，调用下载接口就可以下载音频文件。
-  开发者调用播放接口播放wav音频文件。
    
#### 设置是否自动下载语音消息
可通过`YIMMessageManager`的`SetDownloadAudioMessageSwitch`接口设置是否自动下载语音消息。
SetDownloadAudioMessageSwitch()在初始化之后，启动语音之前调用;若设置了自动下载语音消息，不需再调用DownloadFile()接口，收到语音消息时会自动下载，自动下载完成也会收到DownloadFile()接口对应的OnDownload()回调。

- 原型：

  ``` c
      YIMErrorcode SetDownloadAudioMessageSwitch(bool download);
  ```

- 参数：
  `download `：true-自动下载语音消息，false-不自动下载语音消息(默认)
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   
  
#### 设置语音识别的语言
`SendAudioMessage`提供语音识别功能，将语音识别为文字，默认输入语音为普通话，识别文字为简体中文，可通过`YIMMessageManager`的`SetSpeechRecognizeLanguage`函数设置语音识别的语言；若需要识别为繁体中文，需要联系我们配置此服务。

- 原型：

  ``` c
      YIMErrorcode SetSpeechRecognizeLanguage(SpeechLanguage language);
  ```

- 参数：
  `language`：语言，枚举类型，               
              SPEECHLANG_MANDARIN=0 //普通话
              SPEECHLANG_YUEYU=1    //粤语
              SPEECHLANG_SICHUAN=2  //四川话（windows不支持）
              SPEECHLANG_HENAN=3    //河南话（只支持iOS）
              SPEECHLANG_ENGLISH=4  //英语
              SPEECHLANG_TRADITIONAL=5 //繁体中文
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   
    
#### 发送语音消息
##### 启动录音
可通过`YIMMessageManager`的`SendAudioMessage`接口发送带文字识别的语音消息，调用`StopAudioMessage`接口后自动停止录音并发送，或者调用`CancleAudioMessage`取消本次消息发送。
**注意：语音消息最大的时长是1分钟（超过1分钟就自动发送）**

- 原型：

  ``` c
      // 启动录音带文字识别），开始录制语音消息，支持语音转文本
      YIMErrorcode SendAudioMessage(const XCHAR* receiverID, YIMChatType chatType, XUINT64* requestID);
  ```

- 参数：
  `receiverID`：接收者ID，私聊传入userid，频道聊天传入roomid
  `chatType`：聊天类型
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
    
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 原型：

  ``` c  
      //可通过`YIMMessageManager`的`SendOnlyAudioMessage`接口发送不带文字识别的语音消息，调用`StopAudioMessage`接口后自动停止录音并发送，或者调用`CancleAudioMessage`取消本次消息发送。    
      YIMErrorcode SendOnlyAudioMessage(const XCHAR* receiverID, YIMChatType chatType, XUINT64* requestID);
  ```
    
- 参数：
  `receiverID`：接收者ID，私聊传入userid，频道聊天传入roomid
  `chatType`：聊天类型
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 结束录音并发送
可通过`YIMMessageManager`的`StopAudioMessage`结束录音并发送录音文件。

- 原型：

  ``` c      
      YIMErrorcode StopAudioMessage(const XCHAR* extraParam);
  ```
  
- 参数：
  `extraParam`：发送语音消息的附加参数，格式自定义，自己解析 （json、xml...），可为空字符串。
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义) 
  
- 异步回调接口：
  结束录音并发送接口对应两个回调接口，若语音成功发送出去能得到两个回调通知，语音发送失败则只会得到`发送语音结果回调`。  
  
   ``` c    
  class IYIMMessageCallback
  {
    public:  
    // 开始上传语音回调，（调用StopAudioMessage停止语音之后，成功发送语音消息之前），录音结束，开始发送录音的通知，这个时候已经可以拿到语音文件进行播放
    virtual void OnStartSendAudioMessage(XUINT64 requestID,YIMErrorcode errorcode,const XString& text, const XString& audioPath, unsigned int audioTime)  {}
  };  
  
    // 发送语音结果回调，自己的语音消息发送成功或者失败的通知
    virtual void OnSendAudioMessageStatus(XUINT64 requestID,YIMErrorcode errorcode,const XString& text,const XString& audioPath, unsigned int audioTime, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime) {}
  };  
  ```  
  
- 参数：
  `errorcode`：错误码，`YIMErrorcode_Success` 或者 `YIMErrorcode_PTT_ReachMaxDuration` 均表示成功。
  `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `text`：语音转文字识别的文本内容，如果没有用带语音转文字的接口，该字段为空字符串
  `audioPath`：录音生成的wav文件的本地完整路径，比如"/sdcard/xxx/xxx.wav"
  `audioTime`：录音时长（单位为秒）
  
  `sendTime`：语音发送时间
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言（errorcode为禁言才有效）
  `reasonType`：禁言原因，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它（errorcode为禁言才有效）
  `forbidEndTime`：禁言截止时间戳（errorcode为禁言才有效）

##### 取消本次录音
可通过`YIMMessageManager`的`CancleAudioMessage`接口取消语音。

- 原型：

  ``` c      
      YIMErrorcode CancleAudioMessage();
  ```
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义) 

#### 接收语音消息
接收消息接口参考[收消息](#RecvMessage)通过`GetMessageType()`分拣出语音消息类型：`MessageBodyType_Voice`，用`GetMessageID()`获得消息ID。然后调用`YIMMessageManager`的`DownloadFile`接口下载语音消息，然后调用方播放该文件。

- 原型：

  ``` c
      //下载收到的语音消息文件
      YIMErrorcode DownloadFile(XUINT64 messageID, const XCHAR* savePath);
  ```
  
- 参数：
  `messageID`：消息ID
  `savePath`：文件保存路径（带文件名的全路径），比如"/sdcard/test.wav"如果目录不存在，SDK会自动创建。
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` c  
  class IYIMDownloadCallback
  {
    public:
    virtual void OnDownload( YIMErrorcode errorcode, std::shared_ptr<IYIMMessage> msg, const XString& savePath ) {}
  };
  ```
  
- 回调参数：
  `msg`：消息结构，`IYIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注
  `errorcode`：下载结果错误码
  `savePath`：保存路径，如果没有权限问题，SDK会尝试自动创建

#### 语音播放
##### 播放语音
可通过`YIMManager`的`StartPlayAudio`接口播放语音文件，目前只支持WAV格式。

- 原型：

  ``` c
      YIMErrorcode StartPlayAudio(const XCHAR* path );
  ```

- 参数：
  `path`： 音频文件绝对路径
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` c
  class IYIMAudioPlayCallback
  {
    public:
    virtual void OnPlayCompletion(YIMErrorcode errorcode, const XString& path) {}
  };  
  ```
  
- 参数：
  `errorcode`：错误码，errorcode == YIMErrorcode_Success表示正常播放结束，errorcode == YIMErrorcode_PTT_CancelPlay表示播放到中途被打断。 
  `path`：被播放的音频文件地址。  

##### 停止语音播放
可通过`YIMManager`的`StopPlayAudio`接口停止播放语音文件。

- 原型：

  ``` c
      YIMErrorcode StopPlayAudio();
  ```
    
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)     

##### 设置语音播放音量
可通过`YIMManager`的`SetVolume`接口设置语音的播放音量。

- 原型：

  ``` c
      void SetVolume( float volume );
  ```

- 参数：
  `volume`：音量值，取值范围`0.0f` - `1.0f`，默认值为 `1.0f`。
    
##### 查询播放状态
可通过`YIMManager`的`IsPlaying`接口查询播放状态。

- 原型：

  ``` c
      bool IsPlaying();
  ```

- 返回值：
  是否正在播放,`true`正在播放，`false`没在播放
    
#### 录音缓存
##### 设置录音缓存目录
可通过`YIMManager`的`SetAudioCacheDir`接口设置录音的缓存目录，如果没有设置，SDK会在APP默认缓存路径下创建一个文件夹用于保存音频文件。
**该接口建议初始化之后立即调用** 。

- 原型：

  ``` c
      static void SetAudioCacheDir(const XCHAR* audioCacheDir);
  ```
  
- 参数：
  `audioCacheDir`：缓存录音文件的文件夹路径，如果目录不存在，SDK会自动创建。

##### 获取当前设置的录音缓存目录
可通过`YIMManager`的`GetAudioCachePath`接口获取当前设置的录音缓存目录。

- 原型：

  ``` c
      XString GetAudioCachePath();
  ```
  
- 返回值：
  返回当前设置的录音缓存目录的完整路径,std::string型

##### 清空录音缓存目录
可通过`YIMManager`的`ClearAudioCachePath`接口
清理语音缓存目录(注意清空语音缓存目录后历史记录中会无法读取到音频文件，调用清理历史记录接口也会自动删除对应的音频缓存文件)

- 原型：

  ``` c
      bool ClearAudioCachePath();
  ```
  
- 返回值：
  `true`：表示清理成功，`false`：表示清理操作失败。
    
#### 设置下载保存目录
可通过`YIMMessageManager`的`SetDownloadDir`接口设置下载的保存目录。下载保存目录是DownloadFile的默认下载目录，对下载语音消息和文件均适用；设置下载保存目录在初始化之后，启动语音之前调用，若设置了下载保存目录，调用`YIMMessageManager`的`DownloadFile`接口时，其中的savePath参数可以为空字符串。

- 原型：

  ``` c
      YIMErrorcode SetDownloadDir(const XCHAR* path);
  ``` 
     
- 参数：
  `path`：下载目录路径，必须保证该路径可写。

- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义) 
    
#### 设置只识别语音文字
可通过`YIMMessageManager`的`SetOnlyRecognizeSpeechText`接口设置只识别语音文字。此功能是为了只获取语音识别的文字，不发送语音消息，接口调用与发送语音消息相同，但在成功录音后仅会收到识别的语音文本通知,不会收到OnSendAudioMessage回调。
实现只识别语音文字功能的接口调用顺序：SetOnlyRecognizeSpeechText->SendAudioMessage->StopAudioMessage->OnGetRecognizeSpeechText

- 原型：

  ``` c
      YIMErrorcode SetOnlyRecognizeSpeechText(bool recognition);
  ```
  
- 参数：
  `recognition `：true-只识别语音文字，false-识别语音文字并发送语音消息
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 获取语音识别文本的回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnGetRecognizeSpeechText(XUINT64 requestID, YIMErrorcode errorcode, const XString& text) {}
  };  
  ```
  
- 回调参数：
  `requestID`：序列号
  `errorcode`：错误码
  `text`：返回的语音识别文本内容
  
#### 将接收的语音消息标记为已播放
可通过`YIMMessageManager`的`SetVoiceMsgPlayed`接口将接收的语音消息标记为已播放。

- 原型：

  ``` c     
      YIMErrorcode SetVoiceMsgPlayed(XUINT64 messageID, bool played);
  ```
  
- 参数：
  `messageID`：语音消息ID
  `played`：是否已播放，`true`为已播放，`false`为未播放。  

- 返回值：
   `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)          
    
#### 获取麦克风状态

可通过`YIMManager`的`GetMicrophoneStatus`接口获取麦克风的状态。（该接口会临时占用麦克风）

- 原型：

  ``` c
      void GetMicrophoneStatus();
  ```

- 异步回调接口：

  ``` c
  class IYIMAudioPlayCallback
  {
    public:
    virtual void OnGetMicrophoneStatus(AudioDeviceStatus status) {}
  };  
  ```
  
- 回调参数：
  `status`：麦克风状态值，枚举类型，
           STATUS_AVAILABLE=0    //可用
           STATUS_NO_AUTHORITY=1 //无权限
           STATUS_MUTE=2         //静音
           STATUS_UNAVAILABLE=3  // 不可用  

### 录音上传 
#### 启动录音
可通过`YIMMessageManager`的`StartAudioSpeech`接口启动录音。

- 原型：

  ``` c
      YIMErrorcode StartAudioSpeech(XUINT64* requestID, bool translate = true);
  ```

- 参数：
  `requestID`：返回消息id
  `translate`：是否进行语音转文字识别
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 停止录音并发送
可通过`YIMMessageManager`的`StopAudioSpeech`接口停止录音并发送，该接口只上传到服务器并异步返回音频文件的下载链接，下载链接指向的是AMR格式的音频文件，不会自动发送。
该接口对应`StartAudioSpeech()`。

- 原型：

  ``` c
      YIMErrorcode StopAudioSpeech();
  ```
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 相关接口：
  取消录音，调用该接口后，此次录音不会上传。
  
  ``` c
      //取消本次录音
      YIMErrorcode MyIM::Inst()-> GetMessageManager()->CancleAudioMessage();
  ```

- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnStopAudioSpeechStatus( YIMErrorcode errorcode,std::shared_ptr<IAudioSpeechInfo> audioSpeechInfo ) {}
  };  
  ```
  
- 参数：
  `errorcode`：错误码
  `audioSpeechInfo`：录音信息
  
- 备注：
  `IAudioSpeechInfo` 类成员方法如下： 
  `GetRequestID()`：消息ID 
  `GetDownloadURL()`：语音文件的下载地址
  `GetAudioTime()`：录音时长，单位秒
  `GetFileSize()`：文件大小，字节
  `GetLocalPath()`：本地语音文件的路径
  `GetText()`：语音识别结果，可能为空null or ""  
    
#### 根据url下载录音文件
可通过`YIMMessageManager`的`DownloadFile`接口下载录音文件。

- 原型：

  ``` c
      YIMErrorcode DownloadFile(const XCHAR* downloadURL, const XCHAR* savePath);
  ```

- 参数：
  `downloadURL`：语音文件的url地址，从OnStopAudioSpeechStatus回调获得
  `savePath`：下载语音文件的本地存放地址,带文件名的全路径
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` c      
  class IYIMDownloadCallback
  {
    public:
    virtual void OnDownloadByUrl( YIMErrorcode errorcode, const XString& strFromUrl, const XString& savePath );
  };
  ```
  
- 回调参数：
  `errorcode`：错误码
  `strFromUrl`：下载的语音文件url
  `strSavePath`：本地存放地址  
    
### 消息屏蔽 
#### 屏蔽/解除屏蔽用户消息
可通过`YIMMessageManager`的`BlockUser`接口屏蔽/解除屏蔽用户消息，若屏蔽用户的消息，此屏蔽用户发送的私聊/频道消息都接收不到。

- 原型：

  ``` c
      YIMErrorcode BlockUser(const XCHAR* userID, bool block);
  ```
    
- 参数：
  `userID `：需屏蔽/解除屏蔽的用户ID
  `block `：true-屏蔽 false-解除屏蔽
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnBlockUser(YIMErrorcode errorcode, const XString& userID, bool block) {}
  };  
  ```
    
- 回调参数：
  `errorcode `：错误码
  `userID `：用户ID
  `block `：true-屏蔽 false-解除屏蔽    

#### 解除所有已屏蔽用户
可通过`YIMMessageManager`的`UnBlockAllUser`接口解除所有已屏蔽消息的用户

- 原型：

  ``` c
      YIMErrorcode UnBlockAllUser();
  ```
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnUnBlockAllUser(YIMErrorcode errorcode) {}
  };  
  ```
    
- 回调参数：
  `errorcode`：错误码
  
#### 获取被屏蔽消息用户
可通过`YIMMessageManager`的`GetBlockUsers`接口获取被屏蔽消息的用户。

- 原型：

  ``` c
      YIMErrorcode GetBlockUsers();
  ```
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。  

- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnGetBlockUsers(YIMErrorcode errorcode, std::list<XString> userList) {}
  };  
  ```
    
- 回调参数：
  `errorcode`：错误码
  `userList`：屏蔽用户ID列表       

### 消息功能设置
#### 设置消息已读
可通过`YIMMessageManager`的`SetMessageRead`接口设置消息为已读。

- 原型：

  ``` c     
      YIMErrorcode SetMessageRead(XUINT64 messageID, bool read);
  ```
   
- 参数：
  `messageID`：消息ID
  `read`：是否已读，`true`为已读，`false`为未读。   

- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义) 
  
#### 批量设置消息已读
可通过`YIMMessageManager`的`SetAllMessageRead`接口批量设置消息为已读。

- 原型：

  ``` c    
      YIMErrorcode SetAllMessageRead(const XCHAR* userID, bool read);
  ```
  
- 参数：
  `userID`：发消息的用户ID；用户ID为空字符串时，将登录用户的所有消息设置为已读/未读；用户ID不为空字符串时，将接收的由该用户ID发送的消息设置为已读/未读
  `read`：是否已读，`true`为已读，`false`为未读。  

- 返回值：
   `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)

#### 切换获取消息模式
可通过`YIMMessageManager`的`SetReceiveMessageSwitch`接口切换获取消息模式，在自动接收消息和手动接收消息间切换，默认是自动接收消息。

- 原型：

  ``` c      
      // 是否自动接收消息(true:自动接收(默认)   false:不自动接收消息，有新消息达到时，SDK会发出OnReceiveMessageNotify回调，调用方需要调用GetMessage获取新消息)
      YIMErrorcode SetReceiveMessageSwitch( const std::vector<XString>& vecRoomIDs, bool receive);
  ```
  
- 参数：
  `vecRoomIDs`：频道ID的列表
  `receive`：`true`为自动接收消息，`false`为手动接收消息，默认为`true`。
  
- 返回值：
   `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 手动获取消息
可通过`YIMMessageManager`的`GetNewMessage`接口手动获取消息
在手动接收消息模式，需要调用该接口后才能收到 `OnRecvMessage` 通知。

- 原型：

  ``` c
      YIMErrorcode GetNewMessage( const std::vector<XString>& vecRoomIDs );
  ```
  
- 参数：
  `vecRoomIDs`:频道ID
  
- 返回值：
   `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 新消息通知
在手动接收消息模式，会通知有新消息。

- 通知接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnReceiveMessageNotify( YIMChatType chatType, const XString& targetID ) {}
  };  
  ```
  
- 参数：
  `chatType`：聊天类型，可以用于区分是频道聊天还是私聊 
  `targetID`：如果是频道聊天，该值为频道ID；私聊该值为空 

### 消息记录管理 
#### 设置是否保存频道聊天记录
可通过`YIMMessageManager`的`SetRoomHistoryMessageSwitch`接口设置是否保存频道聊天记录。（默认不保存）

- 原型：

  ``` c
      YIMErrorcode SetRoomHistoryMessageSwitch(const std::vector<XString>& vecRoomIDs, bool save);
  ```
  
- 参数：
  `vecRoomIDs`：频道ID列表
  `save`：是否保存频道消息记录（默认不保存）`true`为保存，`false`为不保存，默认false。
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 拉取频道最近聊天记录
可通过`YIMMessageManager`的`QueryRoomHistoryMessageFromServer`接口从服务器拉取频道最近的聊天历史记录。
这个功能默认不开启，需要的请联系我们修改服务器配置。联系我们，可以通过专属游密支持群或者技术支持的大群。

- 原型：

  ``` c
      YIMErrorcode QueryRoomHistoryMessageFromServer(const XCHAR* roomID, int count = 0, int direction = 0)
  ```
  
- 参数：
  `roomID`：频道id
  `count`：消息数量(最大200条)
  `directon`：历史消息排序方向 0：按时间戳升序	1：按时间戳逆序
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` c  
  class IYIMMessageCallback
  {
    public:   
    virtual void OnQueryRoomHistoryMessageFromServer(YIMErrorcode errorcode, const XString& roomID, int remain, std::list<std::shared_ptr<IYIMMessage> >& messageList) {}
  };   
  ```
    
- 回调参数：    
  `errorcode`：错误码
  `roomID`： 房间ID
  `remain`： 剩余消息数量
  `messageList`：消息列表，`IYIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注  
    
#### 本地历史记录管理
##### 查询本地聊天历史记录
可通过`YIMMessageManager`的`QueryHistoryMessage`接口查询本地聊天历史记录。

- 原型：

  ``` c
      YIMErrorcode QueryHistoryMessage(const XCHAR* targetID, YIMChatType chatType, XUINT64 startMessageID = 0, int count = 30, int direction = 0);
  ```
  
- 参数：
  `targetID`：用户ID或者频道ID
  `chatType`：聊天类型，私聊／频道聊天
  `startMessageID`：起始历史记录消息id（与requestid不同），为`0`表示首次查询，将倒序获取`count`条记录
  `count`：最多获取多少条
  `direction`：历史记录查询方向，`startMessageID=0`时,direction使用默认值0；`startMessageID>0`时，`0`表示查询比`startMessageID`小的消息，`1`表示查询比`startMessageID`大的消息
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  
    
- 相关函数：
  对于房间本地记录，需要先设置自动保存房间消息。`SetRoomHistoryMessageSwitch`

- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public: 
    virtual void OnQueryHistoryMessage(YIMErrorcode errorcode, const XString& targetID, int remain, std::list<std::shared_ptr<IYIMMessage> > messageList) {}
  }; 
  ```
  
- 回调参数：
  `errorcode`：错误码
  `targetID`：用户ID/频道ID
  `remain`：剩余历史消息条数
  `messageList`：消息列表，`IYIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注  

##### 根据时间清理本地聊天历史记录
可通过`YIMMessageManager`的`DeleteHistoryMessage`接口清理本地聊天历史记录。建议定期清理本地历史记录。

- 原型：

  ``` c
      YIMErrorcode DeleteHistoryMessage(YIMChatType chatType = ChatType_Unknow, XUINT64 time = 0) ;
  ```
  
- 参数：
  `chatType`：聊天类型，1私聊，2频道聊天
  `time`：Unix timestamp,精确到秒，表示删除这个时间点之前的所有历史记录
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 根据ID清理本地聊天历史记录
可通过`YIMMessageManager`的`DeleteHistoryMessageByID`接口清理本地聊天历史记录。建议定期清理本地历史记录。

- 原型：

  ``` c
      YIMErrorcode DeleteHistoryMessageByID(XUINT64 messageID);
  ```
    
- 参数：
  `messageID`：如果指定了大于0的值，将删除指定消息id的历史记录
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  
    
##### 根据用户ID清理本地私聊历史记录
可通过`YIMMessageManager`的`DeleteSpecifiedHistoryMessage`接口清理本地私聊历史记录。
可以根据用户ID删除对应的本地私聊历史记录,保留消息ID白名单中的消息记录,白名单列表为空时删除与该用户ID间的所有本地私聊记录。

- 原型：

  ``` c
      YIMErrorcode DeleteSpecifiedHistoryMessage(const XCHAR* targetID, YIMChatType chatType, const std::vector<XUINT64>& excludeMesList);
  ```
  
- 参数：
  `targetID`：用户ID
  `chatType`：指定清理私聊消息
  `excludeMesList`：与该用户ID的私聊消息ID白名单
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  
    
##### 根据用户或房间ID清理本地聊天历史记录
可通过`YIMMessageManager`的`DeleteHistoryMessage`接口清理本地私聊历史记录。
可以根据用户ID或者房间ID删除以指定的起始消息ID开始的n条本地私聊历史记录，起始消息ID和消息数量使用默认值时则删除所有消息。

- 原型：

  ``` c
     YIMErrorcode DeleteHistoryMessage(const XCHAR* targetID, YIMChatType chatType, XUINT64 startMessageID = 0, unsigned int count = 0);
  ```
  
- 参数：
  `targetID`：用户ID或者频道ID
  `chatType`：聊天类型，私聊/频道聊天
  `startMessageID`：起始消息ID(默认值0表示最近的一条消息)
  `count`：消息数量(默认值0表示删除所有消息)
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 获取最近私聊联系人列表
可通过`YIMManager`的`GetRecentContacts`接口获取最近私聊联系人列表，该接口是根据本地历史消息记录生成的最近联系人列表，按最后聊天时间倒序排列。该列表会受清理历史记录消息的接口影响。

- 原型：

  ``` c
      YIMErrorcode GetRecentContacts();
  ``` 
    
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)    

- 异步回调接口：

  ``` c   
  class IYIMContactCallback
  {
    public:   
    virtual void OnGetRecentContacts(YIMErrorcode errorcode, std::list<std::shared_ptr<IYIMContactsMessageInfo> >& contactList) {}
  };
  ```
  
- 回调参数：
  `errorcode`：错误码
  `contactList`：最近联系人列表
  
- 备注：
  `IYIMContactsMessageInfo`类成员方法如下：
  `GetContactID()`：联系人ID
  `GetMessageType()`：消息类型，枚举类型，
                 MessageBodyType_Unknow = 0,         //未知类型
		            MessageBodyType_TXT = 1,            //文本消息
		            MessageBodyType_CustomMesssage = 2, //自定义消息
		            MessageBodyType_Emoji = 3,          //表情
		            MessageBodyType_Image = 4,          //图片
		            MessageBodyType_Voice = 5,          //语音
		            MessageBodyType_Video = 6,          //视频
		            MessageBodyType_File = 7,           //文件
		            MessageBodyType_Gift = 8            //礼物 
  `GetMessageContent()`：消息内容
  `GetCreateTime()`：消息创建时间 
  `GetNotReadMsgNum()`：未读消息数量（此值有意义需结合SetMessageRead接口使用，可在收到消息后设置消息已读）
  `GetLocalPath()`：本地路径（语音或文件消息） 
  `GetExtra()`：额外信息

### 高级功能
#### 文本翻译
可通过`YIMMessageManager`的`TranslateText`接口将文本翻译成指定语言的文本，异步返回结果通过`IYIMMessageCallback`的`OnTranslateTextComplete`接口返回。

- 原型

  ``` c
      YIMErrorcode TranslateText(unsigned int* requestID, const XCHAR* text, LanguageCode destLangCode, LanguageCode srcLangCode = LANG_AUTO);
  ```

- 参数：
  `requestID`：请求ID(SDK生成返回)
  `text`：待翻译文本
  `destLangCode`：目标语言编码
  `srcLangCode`：原文本语言编码

- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义) 

- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:
      void OnTranslateTextComplete(YIMErrorcode errorcode, unsigned int requestID, const XString& text, LanguageCode srcLangCode, LanguageCode destLangCode) {}
  };    
  ```
  
- 回调参数：
  `errorcode`：错误码，等于0才是操作成功
  `requestID`：请求ID
  `text`：翻译成指定语言的文本
  `srcLangCode`：源语言编码
  `destLangCode`：目标语言编码
  
#### 敏感词过滤
考虑到客户端可能需要传递一些自定义消息，关键字过滤方法就直接提供出来，客户端可以选择是否通过`YIMManager`的`FilterKeyword`接口选择是否过滤关键字，并且可以根据匹配到的等级能进行广告过滤。比如`level`返回的值为'2'表示广告，那么可以在发送时选择不发送，接收方可以选择不展示。

- 原型：

  ``` c
      static XString FilterKeyword(const XCHAR* text, int* level);
  ```
  
- 参数：
  `text`：消息原文
  `level`：匹配的策略词等级，默认为0，`0`表示正常，其余值是按需求可配
  
- 返回值：
  过滤关键字后的消息，敏感字会替换为"*"。  

## 游戏暂停与恢复 
### 游戏暂停通知
可通过`YIMManager`的`OnPause`接口暂停游戏，
建议游戏放入后台时通知该接口，以便于得到更好重连效果；
调用OnPause(false),在游戏暂停后,若IM是登录状态，依旧接收IM消息；
调用OnPause(true),游戏暂停，即使IM是登录状态也不会接收IM消息；在游戏恢复运行时会主动拉取暂停期间未接收的消息，收到OnRecvMessage()回调。

- 原型：

  ``` c
      void OnPause(bool pauseReceiveMessage);
  ```
    
- 参数：
  `pauseReceiveMessage`：是否暂停接收IM消息，true-暂停接收 false-不暂停接收  

### 游戏恢复运行通知
可通过`YIMManager`的`OnResume`接口恢复游戏运行。
建议游戏从后台激活到前台时通知该接口，以便于得到更好重连效果。

- 原型：

  ``` c
      void OnResume();
  ```      
  
## 用户举报和禁言
### 用户举报 
`YIMMessageManager`的`Accusation`接口提供举报功能，对[用户违规的发言内容]()进行举报，管理员在后台进行审核处理并将结果通知用户。

- 原型

  ``` c
      YIMErrorcode Accusation(const XCHAR* userID, YIMChatType source, int reason, const XCHAR* description, const XCHAR* extraParam);
  ```
  
- 参数：
  `userID`：被举报用户ID
  `source`：来源（私聊 频道）
  `reason`：原因，由用户自定义枚举
  `description`：原因描述
  `extraParam`：附加信息JSON格式 ({"nickname":"","server_area":"","level":"","vip_level":""}）
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 举报处理结果通知：
  当管理员对举报进行审核处理后，会将举办处理结果通知用户

- 原型：

  ``` c
  class IYIMMessageCallback
  {
    public:
    virtual void OnAccusationResultNotify(AccusationDealResult result, const XString& userID, unsigned int accusationTime) {}
  };  
  ```

- 回调参数：
  `result`：处理结果，枚举类型，
            ACCUSATIONRESULT_IGNORE=0 // 忽略   
            ACCUSATIONRESULT_WARNING=1 // 警告
            ACCUSATIONRESULT_FROBIDDEN_SPEAK=2 // 禁言        
  `userID`：被举报用户ID
  `accusationTime`：举报时间  
  
### 禁言查询 
可通过`YIMMessageManager`的`GetForbiddenSpeakInfo`接口查询所在频道的禁言状态。

- 原型：

  ``` c    
      YIMErrorcode GetForbiddenSpeakInfo(); 
  ```

- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   
    
- 异步回调接口：

  ``` c
  class IYIMMessageCallback
  {
    public:      
    virtual void OnGetForbiddenSpeakInfo( YIMErrorcode errorcode, std::vector< std::shared_ptr<IYIMForbidSpeakInfo> > vecForbiddenSpeakInfos ) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码
  `vecForbiddenSpeakInfos`：禁言状态列表,每一个代表一个频道的禁言状态  
  
- 备注：
  `IYIMForbidSpeakInfo` 类成员方法如下：
  `GetChannelID()`：频道ID
  `GetIsForbidRoom()`：此频道是否被禁言，true-被禁言，false-未被禁言
  `GetReasonType()`：禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
  `GetEndTime()`：若被禁言，禁言结束时间  

## 公告管理 
基本流程：申请开通公告功能->后台添加新公告然后设置公告发送时间,公告消息类型,发送时间,接收公告的频道等->(客户端流程) 设置对应监听-> 调用对应接口->回调接收
### 公告功能简述

1.公告发送由后台配置，如类型、周期、发送时间、内容、链接、目标频道、次数、起始结束时间等。

2.公告三种类型：跑马灯，聊天框，置顶公告：
(1) 跑马灯，聊天框公告可设置发送时间，次数和间隔（从指定时间点开始隔固定间隔时间发送多次，界面展示及显示时长由客户端决定）；
(2) 置顶公告需设置开始和结束时间（该段时间内展示）。

3.三种公告均有一次性、周期性两种循环属性：
  一次性公告，到达指定时间点，发送该条公告；
  周期性公告，跟一次性公告发送规则一致，但是可以设置发送周期（在每周哪几天的指定时间发送）。
  
4.跑马灯与聊天框公告只有发送时间点在线的用户才能收到该公告，显示规则由客户端自己决定，两者区别主要是界面显示的区分。

5.置顶公告有显示起始和结束时间，表示该时段内显示，公告发送时间点在线的用户会收到该公告，公告发送时间点未在线用户，在公告显示时段登录，登录后可通过查询公告接口查到该公告。

6.公告撤销
仅针对置顶公告，公告显示时段撤销公告，客户端会收到公告撤销通知，界面进行更新。
   
### 接收公告
管理员在后台发布公告，当到达指定时间会收到该公告，界面根据不同类型的公告进行展示。

- 原型：

  ``` c
  class IYIMNoticeCallback
  {
    public:
    virtual void OnRecvNotice(YIMNotice* notice) {}
  };  
  ```
  
- 参数：
  `notice`：公告信息
  
- 备注：
  `YIMNotice`类成员方法如下：
  `GetNoticeID()`：公告ID，unsigned long long型
  `GetNoticeType()`：公告类型，1-跑马灯公告  2-聊天框公告  3-置顶公告
  `GetChannelID()`：发布公告的频道ID
  `GetContent()`：公告内容
  `GetLinkText()`：公告链接关键字
  `GetLinkAddr()`：公告链接地址
  `GetBeginTime()`：公告开始时间
  `GetEndTime()`：公告结束时间  

### 撤销公告
对于某些类型的公告（如置顶公告），需要在界面展示一段时间，如果管理员在该时间段执行撤销该公告，会收到撤销公告通知，界面进行相应更新。

- 原型：

  ``` c
  class IYIMNoticeCallback
  {
     public:
     virtual void OnCancelNotice(XUINT64 noticeID, const XString& channelID) {}
  };   
  ```
  
- 参数：
  `noticeID`：公告ID
  `channelID`：发布公告的频道ID

### 查询公告
公告在配置的时间点下发到客户端，对于某些类型的公告（如置顶公告）需要在某个时间段显示在，如果用户在公告下发时间点未在线，而在公告展示时间段内登录，应用可根据自己的需要决定是否展示该公告，可通过`YIMManager`的`QueryNotice`接口查询公告，结果通过上面的`OnRecvNotice`异步回调返回。

- 原型：

  ``` c
      YIMErrorcode QueryNotice();
  ```
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   
    
## 地理位置 
### 获取自己当前的地理位置
可通过`YIMLocationManager`的`GetCurrentLocation`接口请求获取自己当前地理位置。异步返回结果通过`IYIMLocationCallback`的`OnUpdateLocation`接口返回。

- 原型：

  ``` c
      YIMErrorcode GetCurrentLocation();
  ```
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` c
  class IYIMLocationCallback
  {
    public:
    virtual void OnUpdateLocation(YIMErrorcode errorcode, std::shared_ptr<GeographyLocation> location) {}
  };  
  ```

- 回调参数：
  `errorcode`：等于0才是操作成功。
  `location`：当前地理位置及行政区划信息。
  
- 备注：
  `GeographyLocation`类成员方法如下：
  `GetDistrictCode()`：unsigned int类型，地理区域编码，可根据此参数组合附近频道id
  `GetCountry()`：国家
  `GetProvince()`：省份
  `GetCity()`：城市
  `GetDistrictCounty()`：区县
  `GetStreet()`：街道
  `GetLongitude()`：double类型，经度
  `GetLatitude()`：double类型，纬度  

### 获取附近的人
可通过`YIMLocationManager`的`GetNearbyObjects`接口获取自己附近的人，若需要此功能，请联系我们开启LBS服务。若已开启服务，此功能生效的前提是自己和附近的人都获取了自己的地理位置，即调用了IM的获取当前地理位置接口。。异步返回结果通过`IYIMLocationCallback`的`OnGetNearbyObjects`接口返回。

- 原型：

  ``` c
      YIMErrorcode GetNearbyObjects(int count, const XCHAR* serverAreaID, DistrictLevel districtlevel = DISTRICT_UNKNOW, bool resetStartDistance = false);
  ```

- 参数：
  `count`：获取数量（一次最大200个）
  `serverAreaID`：区服ID（对应设置用户信息中的区服，如果只需要获得本服务器的，要填；否则填""）
  `districtlevel`：行政区划等级，枚举类型，
                   DISTRICT_UNKNOW=0   //未知
                   DISTRICT_COUNTRY=1  //国家
                   DISTRICT_PROVINCE=2 //省份
                   DISTRICT_CITY=3     //城市
                   DISTRICT_COUNTY=4   //区县
                   DISTRICT_STREET=5   //街道
  `resetStartDistance`：是否重置查找起始距离,true-从距自己0米开始查找，false-从上次查找返回的最远用户开始查找
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` c
  class IYIMLocationCallback
  {
    public:
    virtual void OnGetNearbyObjects(YIMErrorcode errorcode, std::list< std::shared_ptr<RelativeLocation> >  neighbourList, unsigned int startDistance, unsigned int endDistance) {}
  };  
  ```

- 回调参数：
  `errorcode`：等于0才是操作成功。
  `neighbourList`：附近的人地理位置信息列表
  `startDistance`：查找起始距离
  `endDistance`：查找终止距离
  
- 备注：  
   `RelativeLocation`类成员方法如下：
   `GetDistance()`：与自己的距离
   `GetLongitude()`：经度
   `GetLatitude()`：纬度
   `GetUserID()`：附近用户ID
   `GetCountry()`：国家
   `GetProvince()`：省份
   `GetCity()`：城市
   `GetDistrictCounty()`：区县
   `GetStreet()`：街道  

### 地理位置更新
从资源和耗电方面的考虑，SDK不自动监听地理位置的变化，如果调用方有需要可调用`YIMLocationManager`的`SetUpdateInterval`接口设置更新时间间隔，SDK会按设定的时间间隔监听位置变化并通知上层。（如果应用对地理位置变化关注度不大，最好不要设置自动更新）

- 原型：

  ``` c
      void SetUpdateInterval(unsigned int interval);
  ```

- 参数：
  `interval`：时间间隔（单位：分钟）
  
### 获取与指定用户距离 
可调用`YIMLocationManager`的`GetDistance`接口获取与指定用户距离；获取与指定用户距离之前，需要调用`GetCurrentLocation`成功获取自己的地理位置，指定的用户也调用`GetCurrentLocation`成功获取其地理位置。

- 原型：

  ``` c
      YIMErrorcode GetDistance(const XCHAR* userID);
  ```

- 参数：
  `userID`：用户ID
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` c
  class IYIMLocationCallback
  {
    public:
    virtual void OnGetDistance(YIMErrorcode errorcode, const XString& userID, unsigned int distance) {}
  };  
  ```

- 回调参数：
  `errorcode`：错误码
  `userID`: 用户ID
  `distance`: 距离（米） 

## 关系链管理  
### 用户信息管理
#### 用户资料变更的通知
当好友的用户资料变更时会收到此通知，使用方根据需要决定是否重新获取资料变更好友的用户信息。

- 原型：

  ``` c 
  class IYIMUserProfileCallback
  {
     public:
     virtual void OnUserInfoChangeNotify(const XString& userID) {}
  };   
  ```

- 参数：
  `userID`：资料变更用户的用户ID 
  
#### 设置用户信息
可通过`YIMUserProfileManager`的`SetUserProfileInfo`接口设置用户的基本资料，昵称，性别，个性签名，地理位置等。

- 原型：

  ``` c
      YIMErrorcode SetUserProfileInfo(const IMUserSettingInfo &userSettingInfo);
  ```

- 参数：
  `userSettingInfo`：用户基本信息
  
- 备注：
  `IMUserSettingInfo`结构体成员如下：  
  `nickName`：string类型，昵称
  `sex`：枚举类型，性别，IMUserSex.SEX_UNKNOWN=0  //未知性别
                      IMUserSex.SEX_MALE=1     //男性
                      IMUserSex.SEX_FEMALE=2   //女性
  `personalSignature`：string类型，个性签名
  `country`：string类型，国家  
  `province`：string类型，省份
  `city`：string类型，城市
  `extraInfo`：string类型，扩展信息，JSON格式   
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c  
  class IYIMUserProfileCallback
  {
    public:  
    virtual void OnSetUserInfo(YIMErrorcode errorcode) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码
  
#### 查询用户基本资料
可通过`YIMUserProfileManager`的`GetUserProfileInfo`接口查询用户的基本资料，昵称，性别，个性签名，头像，地理位置，被添加权限等。

- 原型：

  ``` c
      YIMErrorcode GetUserProfileInfo(const XCHAR* userID = __XT(""));
  ```

- 参数：
  `userID`：用户ID,参数值为空时获取自己的基本资料
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。   
    
- 异步回调接口：

  ``` c 
  class IYIMUserProfileCallback
  {
    public:    
    virtual void OnQueryUserInfo(YIMErrorcode errorcode, const IMUserProfileInfo &userInfo) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码
  `userInfo`：用户资料
  
- 备注：
  `IMUserProfileInfo`结构体成员如下：  
  `userID`：string类型，用户ID
  `photoURL`：string类型，头像url
  `onlineState`：枚举类型，在线状态，
                 YIMUserStatus.STATUS_ONLINE=0   //在线，默认值（已登录）
                 YIMUserStatus.STATUS_OFFLINE=1  //离线 
                 YIMUserStatus.STATUS_INVISIBLE=2 //隐身                
  `beAddPermission`：枚举类型，被好友添加权限，
               IMUserBeAddPermission.NOT_ALLOW_ADD=0    //不允许被添加
               IMUserBeAddPermission.NEED_VALIDATE=1    //需要验证
               IMUserBeAddPermission.NO_ADD_PERMISSION=2 //允许被添加，不需要验证, 默认值  
  `foundPermission`：枚举类型，被查找时是否显示被添加权限，
                    IMUserFoundPermission.CAN_BE_FOUND=0  //能被其它用户查找到，默认值
                    IMUserFoundPermission.CAN_NOT_BE_FOUND=1  //不能被其它用户查找到                    
  `settingInfo`：IMUserSettingInfo类型，用户基本信息，详见#设置用户信息-备注
  
#### 设置用户头像
可通过`YIMUserProfileManager`的`SetUserProfilePhoto`接口设置用户的头像。

- 原型：

  ``` c
      YIMErrorcode SetUserProfilePhoto(const XCHAR* photoPath);
  ```

- 参数：
  `photoPath`：本地图片绝对路径,图片URL长度在500bytes以内，图片大小在100kb以内
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c   
  class IYIMUserProfileCallback
  {
    public:  
    virtual void OnSetPhotoUrl(YIMErrorcode errorcode, const XString &photoUrl) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码
  `photoUrl`：图片下载路径  
  
#### 切换用户状态
可通过`YIMUserProfileManager`的`SwitchUserStatus`接口切换用户的在线状态。

- 原型：

  ``` c
      YIMErrorcode SwitchUserStatus(const XCHAR* userID, YIMUserStatus userStatus);
  ```

- 参数：
  `userID`：用户ID
  `userStatus`：用户在线状态，枚举类型，
               YIMUserStatus.STATUS_ONLINE=0    //在线，默认值（已登录）
               YIMUserStatus.STATUS_OFFLINE=1   //离线
               YIMUserStatus.STATUS_INVISIBLE=2 //离线
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c  
  class IYIMUserProfileCallback
  {
    public:   
    virtual void OnSwitchUserOnlineState(YIMErrorcode errorcode) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码 
  
#### 设置好友添加权限
可通过`YIMUserProfileManager`的`SetAddPermission`接口设置用户被好友添加的权限。

- 原型：

  ``` c
      YIMErrorcode SetAddPermission(bool beFound, IMUserBeAddPermission beAddPermission);
  ```

- 参数：
  `beFound`：能否被别人查找到，true-能被查找，false-不能被查找
  `beAddPermission`：被其它用户添加的权限，枚举类型，IMUserBeAddPermission:
                   NOT_ALLOW_ADD=0    //不允许被添加
                   NEED_VALIDATE=1    //需要验证
                   NO_ADD_PERMISSION=2 //允许被添加，不需要验证, 默认值  

  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c  
  class IYIMUserProfileCallback
  {
    public:   
    virtual void OnSetUserInfo(YIMErrorcode errorcode) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码    
  
### 好友管理  
#### 查找好友
可通过`YIMFriendManager`的`FindUser`接口查找将要添加的好友，获取该好友的简要信息。

- 原型：

  ``` c
      YIMErrorcode FindUser(int findType, const XCHAR* target);
  ```

- 参数：
  `findType`：查找类型，0-按ID查找，1-按昵称查找
  `target`：对应查找类型选择的用户ID或昵称
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c   
  class IYIMFriendCallback
  {
    public:
    virtual void OnFindUser(YIMErrorcode errorcode, std::list<std::shared_ptr<IYIMUserBriefInfo> >& users) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码 
  `users`：用户简要信息列表
  
- 备注：
  `IYIMUserBriefInfo`结构体成员方法如下：  
  `GetUserID()`：用户ID
  `GetNickname()`：用户昵称
  `GetUserStatus()`：枚举类型，在线状态，YIMUserStatus:
                 STATUS_ONLINE=0   //在线
                 STATUS_OFFLINE=1  //离线
                 STATUS_INVISIBLE=2 //隐身     
       
#### 添加好友
可通过`YIMFriendManager`的`RequestAddFriend`接口添加好友。

- 原型：

  ``` c
      YIMErrorcode RequestAddFriend(std::vector<XString>& users, const XCHAR* comments);
  ```

- 参数：
  `users`：需要添加为好友的用户ID列表
  `comments`：备注或验证信息，长度最大128bytes
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。   
    
- 异步回调接口：

  ``` c 
  class IYIMFriendCallback
  {
    public:
      void OnRequestAddFriend(YIMErrorcode errorcode, const XString& userID) {}
  };    
  ```
  
- 回调参数：
  `errorcode`：错误码 
  `userID`：用户ID
  
- 备注：
  被请求方会收到被添加为好友的通知，被请求方如果设置了需要验证才能被添加，会收到下面的回调：
  
  ``` c  
  class IYIMFriendCallback
  {
    public: 
    virtual void OnBeRequestAddFriendNotify(const XString& userID, const XString& comments) {}
  };  
  ```   
  
  - 参数：     
    `userID`：请求方的用户ID
    `comments`：备注或验证信息
    
  被请求方如果设置了不需要验证就能被添加，收到下面的回调：
  
  ``` c  
  class IYIMFriendCallback
  {
    public: 
    virtual void OnBeAddFriendNotify(const XString& userID, const XString& comments) {}
  };  
  ```   
  
  - 参数：     
    `userID`：请求方的用户ID
    `comments`：备注或验证信息  
  
#### 处理好友请求
当前用户有被其它用户请求添加为好友的请求时，可通过`YIMFriendManager`的`DealBeRequestAddFriend`接口处理被添加为好友的请求。

- 原型：

  ``` c
      YIMErrorcode DealBeRequestAddFriend(const XCHAR* userID, int dealResult);
  ```

- 参数：
  `userID`：请求方的用户ID
  `dealResult`：处理结果，0-同意，1-拒绝
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c   
  class IYIMFriendCallback
  {
    public:
    virtual void OnDealBeRequestAddFriend(YIMErrorcode errorcode, const XString& userID, const XString& comments, int dealResult) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码 
  `userID`：请求方的用户ID
  `comments`：备注或验证信息
  `dealResult`：处理结果，0-同意，1-拒绝 
  
- 备注：
 
   如果被请求方设置了需要验证才能被添加为好友，在被请求方成功处理了请求方的好友请求后，请求方能收到添加好友请求结果的通知，收到下面的回调：
   
  ``` c  
  class IYIMFriendCallback
  {
    public: 
    vitual void OnRequestAddFriendResultNotify(const XString& userID, const XString& comments, int dealResult) {}
  };  
  ```
  
- 参数：   
  `userID`：被请求方的用户ID
  `comments`：备注或验证信息
  `dealResult`：处理结果，0-同意，1-拒绝  
    
#### 删除好友
可通过`YIMFriendManager`的`DeleteFriend`接口删除好友。

- 原型：

  ``` c
      YIMErrorcode DeleteFriend(std::vector<XString>& users, int deleteType = 1);
  ```

- 参数：
  `users`：需删除的好友用户ID列表
  `deleteType`：删除类型，0-双向删除，1-单向删除(删除方在被删除方好友列表依然存在)
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。   
    
- 异步回调接口：

  ``` c  
  class IYIMFriendCallback
  {
    public: 
    virtual void OnDeleteFriend(YIMErrorcode errorcode, const XString& userID) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码 
  `userID`：被删除好友的用户ID
  
- 备注： 
  如果删除方采用的是双向删除，被删除方能收到其被好友删除的通知，收到下面的回调：
   
  ``` c   
  class IYIMFriendCallback
  {
    public:
    virtual void OnBeDeleteFriendNotify(const XString& userID) {}
  };  
  ```
  
- 参数：   
  `userID`：删除方的用户ID   
  
#### 拉黑好友
可通过`YIMFriendManager`的`BlackFriend`接口拉黑或者解除拉黑好友。

- 原型：

  ``` c
      YIMErrorcode BlackFriend(int type, std::vector<XString>& users);
  ```

- 参数：
  `type`：拉黑类型，0-拉黑，1-解除拉黑
  `users`：需拉黑的好友用户ID列表  
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c  
  class IYIMFriendCallback
  {
    public: 
    virtual void OnBlackFriend(YIMErrorcode errorcode, int type, const XString& userID) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码 
  `type`：拉黑类型，0-拉黑，1-解除拉黑
  `userID`：被拉黑方用户ID    
  
#### 查询好友列表
可通过`YIMFriendManager`的`QueryFriends`接口查询当前用户已添加的好友，也能查找被拉黑的好友。

- 原型：

  ``` c
      YIMErrorcode QueryFriends(int type = 0, int startIndex = 0, int count = 50);
  ```

- 参数：
  `type`：查找类型，0-正常状态的好友，1-被拉黑的好友
  `startIndex`：起始序号，作用为分批查询好友（例如：第一次从序号0开始查询30条，第二次就从序号30开始查询相应的count数）
  `count`：数量（一次最大100）  
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。   
    
- 异步回调接口：

  ``` c  
  class IYIMFriendCallback
  {
    public: 
    virtual void OnQueryFriends(YIMErrorcode errorcode, int type, int startIndex, std::list<std::shared_ptr<IYIMUserBriefInfo> >& friends) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码 
  `type`：查找类型，0-正常状态的好友，1-被拉黑的好友
  `startIndex`：起始序号
  `friends`：好友列表 
  
- 备注：
  `IYIMUserBriefInfo`结构体成员方法如下：  
  `GetUserID()`：用户ID
  `GetNickname()`：用户昵称
  `GetUserStatus()`：枚举类型，在线状态，YIMUserStatus:
                 STATUS_ONLINE=0   //在线
                 STATUS_OFFLINE=1  //离线
                 STATUS_INVISIBLE=2 //隐身   
  
#### 查询好友请求列表
可通过`YIMFriendManager`的`QueryFriendRequestList`接口查询当前用户收到的被添加请求的好友列表。

- 原型：

  ``` c
      YIMErrorcode QueryFriendRequestList(int startIndex = 0, int count = 20);
  ```

- 参数：  
  `startIndex`：起始序号，作用为分批查询好友请求（例如：第一次从序号0开始查询10条，第二次就从序号10开始查询相应的count数）
  `count`：数量（一次最大20）  
  
- 返回值：
  `YIMErrorcode`：错误码，详细描述见[错误码定义](#错误码定义)。    
    
- 异步回调接口：

  ``` c  
  class IYIMFriendCallback
  {
    public: 
    virtual void OnQueryFriendRequestList(YIMErrorcode errorcode, int startIndex, std::list<std::shared_ptr<IYIMFriendRequestInfo> >& requestList) {}
  };  
  ```
  
- 回调参数：
  `errorcode`：错误码 
  `startIndex`：起始序号
  `requestList`：好友请求列表
  
- 备注：
  `IYIMFriendRequestInfo`结构体成员方法如下：  
  `GetAskerID()`：请求方ID
  `GetAskerNickname()`：请求方昵称
  `GetInviteeID()`：受邀方ID
  `GetInviteeNickname()`：受邀方昵称
  `GetValidateInfo()`：验证信息
  `GetStatus()`：枚举类型，好友请求状态，YIMAddFriendStatus:
                 STATUS_ADD_SUCCESS=0         //添加完成
                 STATUS_ADD_FAILED=1          //添加失败  
                 STATUS_WAIT_OTHER_VALIDATE=2 //等待对方验证 
                 STATUS_WAIT_ME_VALIDATE=3    //等待我验证
  `GetCreateTime()`：unsigned int类型，请求的发送或接收时间  

## 全球布点选择
**必须在初始化之前调用，** 根据游戏主要发行区域，选择合适的 IM 服务器区域。

- 原型：

  ``` c
      static void YIMManager::SetServerZone(ServerZone zone);
  ```
  
- 参数：
  `zone`：服务器区域，参考ServerZone枚举说明
  
## 服务器部署地区定义 

``` c
ServerZone.China = 0         // 中国
ServerZone.Singapore = 1     // 新加坡  
ServerZone.America = 2 	   // 美国
ServerZone.HongKong = 3 	   // 香港
ServerZone.Korea = 4 		   // 韩国
ServerZone.Australia = 5     // 澳洲
ServerZone.Deutschland = 6	// 德国
ServerZone.Brazil = 7  		// 巴西
ServerZone.India = 8  		   // 印度
ServerZone.Japan = 9		   // 日本
ServerZone.Ireland = 10	   // 爱尔兰
ServerZone.ServerZone_Unknow = 9999
```  

## 错误码定义

| 错误码                                               | 含义         |
| -------------                                       | -------------|
| YIMErrorcode_Success = 0                        | 成功|
| YIMErrorcode_EngineNotInit = 1                  | IM SDK未初始化|
| YIMErrorcode_NotLogin = 2                       | IM SDK未登录|
| YIMErrorcode_ParamInvalid = 3                   | 无效的参数|
| YIMErrorcode_TimeOut = 4                        | 超时|
| YIMErrorcode_StatusError = 5                    | 状态错误|
| YIMErrorcode_SDKInvalid = 6                     | Appkey无效|
| YIMErrorcode_AlreadyLogin = 7                   | 已经登录|
| YIMErrorcode_LoginInvalid = 1001                | 登录无效|
| YIMErrorcode_ServerError = 8                    | 服务器错误|
| YIMErrorcode_NetError = 9                       | 网络错误|
| YIMErrorcode_LoginSessionError = 10             | 登录状态出错|
| YIMErrorcode_NotStartUp = 11                    | SDK未启动|
| YIMErrorcode_FileNotExist = 12                  | 文件不存在|
| YIMErrorcode_SendFileError = 13                 | 文件发送出错|
| YIMErrorcode_UploadFailed = 14                  | 文件上传失败,上传失败 一般都是网络限制上传了|
| YIMErrorcode_UsernamePasswordError = 15,         | 用户名密码错误|
| YIMErrorcode_UserStatusError = 16,               | 用户状态为无效用户|
| YIMErrorcode_MessageTooLong = 17,                | 消息太长|
| YIMErrorcode_ReceiverTooLong = 18,               | 接收方ID过长（检查频道名）|
| YIMErrorcode_InvalidChatType = 19,               | 无效聊天类型|
| YIMErrorcode_InvalidReceiver = 20,               | 无效用户ID|
| YIMErrorcode_UnknowError = 21,                   | 未知错误|
| YIMErrorcode_InvalidAppkey = 22,                 | AppKey无效|
| YIMErrorcode_ForbiddenSpeak = 23,                | 被禁止发言|
| YIMErrorcode_CreateFileFailed = 24,              | 创建文件失败|
| YIMErrorcode_UnsupportFormat = 25,               | 支持的文件格式|
| YIMErrorcode_ReceiverEmpty = 26,                 | 接收方为空|
| YIMErrorcode_RoomIDTooLong = 27,                 | 房间名太长|
| YIMErrorcode_ContentInvalid = 28,                | 聊天内容严重非法|
| YIMErrorcode_NoLocationAuthrize = 29,            | 未打开定位权限|
| YIMErrorcode_UnknowLocation = 30,                | 未知位置|
| YIMErrorcode_Unsupport = 31,                     | 不支持该接口|
| YIMErrorcode_NoAudioDevice = 32,                 | 无音频设备|
| YIMErrorcode_AudioDriver = 33,                   | 音频驱动问题|
| YIMErrorcode_DeviceStatusInvalid = 34,           | 设备状态错误|
| YIMErrorcode_ResolveFileError = 35,              | 文件解析错误|
| YIMErrorcode_ReadWriteFileError = 36,            | 文件读写错误|
| YIMErrorcode_NoLangCode = 37,                    | 语言编码错误|
| YIMErrorcode_TranslateUnable = 38,               | 翻译接口不可用|
| YIMErrorcode_SpeechAccentInvalid = 39,           | 语音识别方言无效|
| YIMErrorcode_SpeechLanguageInvalid = 40,         | 语音识别语言无效|
| YIMErrorcode_HasIllegalText = 41,                | 消息含非法字符|
| YIMErrorcode_AdvertisementMessage = 42,          | 消息涉嫌广告|
| YIMErrorcode_AlreadyBlock = 43,                  | 用户已经被屏蔽|
| YIMErrorcode_NotBlock = 44,                      | 用户未被屏蔽|
| YIMErrorcode_MessageBlocked = 45,                | 消息被屏蔽|
| YIMErrorcode_LocationTimeout = 46,               | 定位超时|
| YIMErrorcode_NotJoinRoom = 47,                   | 未加入该房间|
| YIMErrorcode_LoginTokenInvalid = 48,             | 登录token错误|
| YIMErrorcode_CreateDirectoryFailed = 49,         | 创建目录失败|
| YIMErrorcode_InitFailed = 50,                    | 初始化失败|
| YIMErrorcode_Disconnect = 51,                    | 与服务器断开|    
| YIMErrorcode_TheSameParam = 52,                  | 设置参数相同|
| YIMErrorcode_QueryUserInfoFail = 53,             | 查询用户信息失败|
| YIMErrorcode_SetUserInfoFail = 54,               | 设置用户信息失败|
| YIMErrorcode_UpdateUserOnlineStateFail = 55,     | 更新用户在线状态失败|
| YIMErrorcode_NickNameTooLong = 56,               | 昵称太长(> 64 bytes)|
| YIMErrorcode_SignatureTooLong = 57,              | 个性签名太长(> 120 bytes)|
| YIMErrorcode_NeedFriendVerify = 58,              | 要好友验证信息|
| YIMErrorcode_BeRefuse = 59,                      | 添加好友被拒绝|
| YIMErrorcode_HasNotRegisterUserInfo = 60,        | 未注册用户信息|
| YIMErrorcode_AlreadyFriend = 61,                 | 已经是好友|
| YIMErrorcode_NotFriend = 62,                     | 非好友|
| YIMErrorcode_NotBlack = 63,                      | 不在黑名单中|
| YIMErrorcode_PhotoUrlTooLong = 64,               | 头像url过长(>500 bytes)|
| YIMErrorcode_PhotoSizeTooLarge = 65,             | 头像太大（>100 kb）|
| YIMErrorcode_ChannelMemberOverflow = 66,         | 达到频道人数上限 |
| YIMErrorcode_ALREADYFRIENDS = 1000,               | 服务器错误|
| YIMErrorcode_LoginInvalid = 1001,                 | 服务器错误登录无效|
| YIMErrorcode_PTT_Start = 2000,                    | 开始录音|
| YIMErrorcode_PTT_Fail = 2001,                     | 录音失败|
| YIMErrorcode_PTT_DownloadFail = 2002,             | 语音消息文件下载失败|
| YIMErrorcode_PTT_GetUploadTokenFail = 2003,       | 获取语音消息Token失败|
| YIMErrorcode_PTT_UploadFail = 2004,               | 语音消息文件上传失败|
| YIMErrorcode_PTT_NotSpeech = 2005,                | 没有录音内容|
| YIMErrorcode_PTT_DeviceStatusError = 2006,        | 语音设备状态错误
| YIMErrorcode_PTT_IsSpeeching = 2007,              | 录音中|
| YIMErrorcode_PTT_FileNotExist = 2008,             | 文件不存在|
| YIMErrorcode_PTT_ReachMaxDuration = 2009,         | 达到语音最大时长限制|
| YIMErrorcode_PTT_SpeechTooShort = 2010,           | 录音时间太短|
| YIMErrorcode_PTT_StartAudioRecordFailed = 2011,   | 启动录音失败|
| YIMErrorcode_PTT_SpeechTimeout = 2012,            | 音频输入超时|
| YIMErrorcode_PTT_IsPlaying = 2013,                | 正在播放|
| YIMErrorcode_PTT_NotStartPlay = 2014,             | 未开始播放|
| YIMErrorcode_PTT_CancelPlay = 2015,               | 主动取消播放|
| YIMErrorcode_PTT_NotStartRecord = 2016,           | 未开始语音|
| YIMErrorcode_PTT_NotInit = 2017,                  | 未初始化|
| YIMErrorcode_PTT_InitFailed = 2018,               | 初始化失败|
| YIMErrorcode_PTT_Authorize = 2019,                | 录音权限|
| YIMErrorcode_PTT_StartRecordFailed = 2020,        | 启动录音失败|
| YIMErrorcode_PTT_StopRecordFailed = 2021,         | 停止录音失败|
| YIMErrorcode_PTT_UnsupprtFormat = 2022,           | 不支持的格式|
| YIMErrorcode_PTT_ResolveFileError = 2023,         | 解析文件错误|
| YIMErrorcode_PTT_ReadWriteFileError = 2024,       | 读写文件错误|
| YIMErrorcode_PTT_ConvertFileFailed = 2025,        | 文件转换失败|
| YIMErrorcode_PTT_NoAudioDevice = 2026,            | 无音频设备|
| YIMErrorcode_PTT_NoDriver = 2027,                 | 驱动问题|
| YIMErrorcode_PTT_StartPlayFailed = 2028,          | 启动播放失败|
| YIMErrorcode_PTT_StopPlayFailed = 2029,           | 停止播放失败|
| YIMErrorcode_PTT_RecognizeFailed = 2030,          | 识别失败|
| YIMErrorcode_Fail = 10000                         | 语音服务启动失败|





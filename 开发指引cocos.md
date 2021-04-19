# IM SDK for Cocos2d-x 开发指引

## 概述

游密即时通讯SDK（IM SDK）为玩家提供完整的游戏内互动服务，游戏开发者无需关注IM通讯复杂的内部工作流程，只需调用IM SDK提供的接口，即可快速实现世界聊天、公会聊天、组队聊天、文字、表情、语音等多项功能。

## IM SDK接口调用顺序

![IM SDK接口调用顺序](https://youme.im/doc/images/im_sdk_interface_call_order.png)

## 四步集成IM SDK
### 第一步 注册账号
在[游密官网](https://console.youme.im/user/register)注册游密账号。

### 第二步 添加游戏，获取Appkey
在控制台添加游戏，获得接入需要的Appkey、Appsecret。

### 第三步 下载IM SDK包体
[下载地址](https://www.youme.im/download.php?type=IM)

### 第四步 开发环境配置
[开发环境配置](#快速接入)

## 快速接入
### 1. 导入IM SDK
#### 针对Android导入

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

    # ############# 不使用翻译可以删除下面内容 ##############
    
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
    
    # ###################################################
    ```

4. 将 `yim/lib/android/` 目录中如下文件复制到`proj.android/libs/`目录下。
>`yim.jar`
>`alisr.jar`
>`usc.jar`
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

  ```

     -keep class com.youme.** {*;}
     -keep class com.iflytek.**
     -keepattributes Signature
  ```

#### 针对iOS导入
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
 >`AliyunNlsSdk.framework`  //如果是带语音转文字的sdk需要添加，动态库，需要在`General -> Embedded Binaries`也进行添加。  
 >`USCModule.framework`     //如果是带语音转文字的sdk需要添加。  
 >`libBoringSSL.a`      //如果是带google语音转文字的sdk需要
 >`libgoogleapis.a`//如果是带google语音转文字的sdk需要
 >`libgRPC-Core.a`//如果是带google语音转文字的sdk需要
 >`libgRPC-ProtoRPC.a`//如果是带google语音转文字的sdk需要
 >`libgRPC-RxLibrary.a`//如果是带google语音转文字的sdk需要
 >`libgRPC.a`//如果是带google语音转文字的sdk需要
 >`libnanopb.a`//如果是带google语音转文字的sdk需要
 >`libProtobuf.a`//如果是带google语音转文字的sdk需要

6 如果接入带google语音识别的版本，需在`Build Phases  ->Copy Bundle Resources` 里加入 `gRPCCertificates.bundle`

7. **需要为iOS10以上版本添加录音权限配置**
iOS 10 使用录音权限，需要在`info`新加`Privacy - Microphone Usage Description`键，值为字符串，比如“语音聊天需要录音权限”。
首次录音时会向用户申请权限。配置方式如图(选择Private-Microphone Usage Description)。
![iOS10录音权限配置](https://www.youme.im/doc/images/im_iOS_record_config.jpg)

8. **LBS添加获取地理位置权限**
若使用LBS，需要在`info`新加`Privacy - Location Usage Description`键，值为字符串，比如“查看附近的玩家需要获取地理位置权限”。首次使用定位时会向用户申请权限，配置方式如上图录音权限。

#### 针对Windows导入
1. 将`yim`复制到Cocos2d-x开发工具生成的游戏目录<b>根目录下</b>

2. 在项目 -> 属性 -> C/C++ -> 常规 -> 附加包含目录 中添加`..\yim\include`

3. 拷贝yim/lib/x86 目录下的` msc.dll， yim.dll， yim.lib， libusc.dll， jsoncpp-0.y.z.dll， libeay32.dll， nlscppsdk.dll， pthreadVC2.dll， ssleay32.dll `拷贝到执行目录(如果是64位机器，使用yim/lib/x64)

4. 在项目->属性->链接器->输入->附加依赖库 中添加 `yim.lib`（该库是release模式下的  如果运行的时候是debug模式请添加yim_Debug.lib）

### 2. 初始化
成功导入SDK后，应用启动的第一时间需要调用初始化接口。

#### 包含SDK头文件
- 引入IM SDK命名空间：

  ``` c
      #include "YIM.h"
  ```

  获取游密IM管理器实例，因为是单例，多次调用仍然是返回同一个对象指针。通过`YIMManager::CreateInstance()`可以获取聊天室，消息管理器。
  
  ``` cpp
      // 获取频道管理器
      YIMManager::CreateInstance())->GetChatRoomManager();
      // 获取消息发送管理器
      YIMManager::CreateInstance()->GetMessageManager();      
  ```

#### 初始化IM SDK

- **调用示例：**

  ``` cpp
      YIMManager::CreateInstance()->Init("YoumeAppkey"," YoumeAppSecurity", "");
  ```
  
- **相关接口与参数：**
  - `初始化接口`：
    `YIMErrorcode Init(const XCHAR* appKey, const XCHAR* appSecurity, const XCHAR* packageName)`
    
  - `appKey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `appSecurity`：用户游戏产品的密钥，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `packageName`：游戏包名
  
- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	
- **备注：**
  此接口本是异步操作，未给出异步回调，一般返回的错误码是Success即表示初始化成功。
  
### 3. 设置回调监听  
IM引擎底层对于耗时操作都采用异步回调的方式，函数调用会立即返回，操作结果会同步回调。因此，用户必须实现相关接口并在初始化完成以后进行注册。常用功能需要注册登录回调，聊天室回调，消息回调，如需其它功能查看API接口。

- **调用示例：**

  ``` cpp
      // 回调接口方法实现需自行补充
      class MyIM: public IYIMLoginCallback, IYIMMessageCallback, IYIMChatRoomCallback
      {
          ...
          YIMManager::CreateInstance()->SetLoginCallback(this);
          YIMManager::CreateInstance()->SetMessageCallback(this);
          YIMManager::CreateInstance()->SetChatRoomCallback(this);
          ...
          
          virtual void OnLogin(YIMErrorcode errorcode, const XString& userID) override
	       {
		       if (errorcode == YIMErrorcode_Success)
		       {
		          std::cout << "login success." << std::endl;
		       }
	       }
	       // 其余回调接口重载
	       ...
      }    
  ```
  
- **相关接口：**
	- `登录登出回调注册接口`：
	  `void SetLoginCallback(IYIMLoginCallback* pCallback)`
	  
	- `聊天室回调注册接口`：
	  `void SetChatRoomCallback(IYIMChatRoomCallback* pCallback)`
	  
	- `消息回调注册接口`：
	  `void SetMessageCallback(IYIMMessageCallback* pCallback)`
	  
### 4. 登录IM系统
![登录界面介绍](https://www.youme.im/doc/images/im_login.png)

成功初始化IM后，需要使用IM功能时，先调用IM SDK登录接口。

- **调用示例：**

  ``` cpp
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->Login("123456", "123456", "");
  ```
  
- **相关接口与参数：**
	- `登录接口`：
	  `YIMErrorcode Login(const XCHAR* userID, const XCHAR* passwd ,const XCHAR* token)`
	  
	- `userID`：由调⽤者分配，不可为空字符串，只可由字母或数字或下划线组成，用户首次登录会自动注册所用的用户ID和密码
	- `passwd`：⽤户密码，不可为空字符串，由调用者分配，二次登录时需与首次登录一致，否则会报UsernamePasswordError
	- `token`：用户验证token，可选，如不使用token验证传入:""
	
- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	
- **回调接口与参数：**
   登录接口是异步操作，登录成功的标识是`OnLogin`回调的错误码是Success；调用`Login`同步返回值是Success才能收到`OnLogin`回调；特别的如果调用`Login`多次，收到AlreadyLogin错误码也表示登录成功。 
   
   - `登录回调接口`：
     `virtual void OnLogin(YIMErrorcode errorcode, const XString& userID)`
     
   - `userID`：用户ID
   - `errorcode`：错误码
   
### 5. 进入聊天频道
![频道](https://www.youme.im/doc/images/im_room.png)

登录成功后，如果有群组聊天需要，比如游戏里面的世界、工会、区域等，需要进入聊天频道，调用方为聊天频道设置一个唯一的频道ID，可以进入多个频道。

- **调用示例：**

  ``` cpp
      // 可在登录成功回调里面执行
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->GetChatRoomManager()->JoinChatRoom("roomID");
  ```
  
- **相关接口与参数**
	- `进入频道接口`：
	  `YIMErrorcode JoinChatRoom(const XCHAR* chatRoomID)`
	  
	- `chatRoomID`：请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)

- **回调接口与参数：**
  进入频道接口是异步操作，进入频道成功的标识是`OnJoinChatRoom`回调的错误码是Success；调用`JoinChatRoom`同步返回值是Success才能收到`OnJoinChatRoom`回调。
  
   - `进入频道回调接口`：
     `virtual void OnJoinChatRoom(YIMErrorcode errorcode, const XString&  strChatRoomID)`
     
   - `errorcode`：错误码
   - `strChatRoomID`：频道ID
   
### 6. 发送文本消息
![频道](https://www.youme.im/doc/images/im_room_send_btn.png)

在成功登录IM后，即可发送IM消息。

- **调用示例：**

  ``` cpp
      XUINT64 iRequestID=0;
      // 发送私聊文本消息
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->GetMessageManager()->SendTextMessage("12345",                                                    YIMChatType::ChatType_PrivateChat,                                                    "Msg content!", "attachMsg",                                                   &iRequestID);
  ```
  
- **相关接口与参数：**
	- `发文本消息接口`：
     `YIMErrorcode SendTextMessage(const XCHAR* receiverID, YIMChatType chatType, const XCHAR* text, const XCHAR* attachParam, XUINT64* requestID)`
     
	- `receiverID`：接收者ID,（⽤户ID或者频道ID）
	- `chatType`：聊天类型,私聊传`1`，频道聊天传`2`；需要频道聊天得先成功进入频道后发文本消息，此频道内的成员才能接收到消息
	- `text`：聊天内容
	- `attachParam`：发送文本附加信息
	- `requestID`：消息序列号，用于校验一条消息发送成功与否的标识

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	
- **回调接口与参数：**
  发送文本消息接口是异步操作，发送文本消息成功的标识是`OnSendMessageStatus`回调的错误码是Success；调用`SendTextMessage`同步返回值是Success才能收到`OnSendMessageStatus`回调；文本消息发送成功，接收方会收到`OnRecvMessage`回调，能从该回调中取得消息内容，消息发送者等信息。
  
   - `发送文本消息回调接口`：
     `virtual void OnSendMessageStatus(XUINT64 requestID, YIMErrorcode errorcode, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime)`
     
   - `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
   - `errorcode`：错误码
   - `sendTime`：消息发送时间
   - `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言
   - `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
   - `forbidEndTime`：若在频道被禁言，禁言结束时间   

- **拓展功能：**
  发送消息时，可以将玩家头像、昵称、角色等级、vip等级等要素打包成json格式发送。

### 7. 发送语音消息
![发送语音消息](https://www.youme.im/doc/images/im_send_voice_message.jpg)

在成功登录IM后，可以发送语音消息
>- 按住语音按钮时，调用`SendAudioMessage`启动录音
>- 松开按钮，调用`StopAudioMessage`接口，发送语音消息
>- 按住过程若需要取消发送，调用`CancleAudioMessage`取消本次语音发送

- **调用示例：**

  ``` cpp      
      // 启动录音
      XUINT64 iRequestID=0;
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->GetMessageManager()->SendAudioMessage("12345",                                                    YIMChatType::ChatType_PrivateChat, &iRequestID);
      
      // 结束录音并发送
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->GetMessageManager()->StopAudioMessage("");
      
      //如需取消发送则调用取消录音
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->GetMessageManager()->CancleAudioMessage();
  ```
  
- **相关接口与参数：**
	- `发送语音消息接口`：
	  `YIMErrorcode SendAudioMessage(const XCHAR* receiverID, YIMChatType chatType, XUINT64* requestID)`
	  
	  若不需要文字识别的语音，使用下面接口。
	  `YIMErrorcode SendOnlyAudioMessage(const XCHAR* receiverID, YIMChatType chatType, XUINT64* requestID)`
	  
	- `结束录音接口`：
	  `YIMErrorcode StopAudioMessage(const XCHAR* extraParam)`
	  
	- `取消录音接口`：
	  `YIMErrorcode CancleAudioMessage()`
	  
	- `receiverID`：接收者ID（⽤户ID或者频道ID）
	- `chatType`：消息类型，表示私聊还是频道消息
	- `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
	- `extraParam`：发送语音消息附加信息，主要用于调用方特别需求

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	
- **回调接口与参数：**
   调用`StopAudioMessage`接口后，会收到发送语音的回调，如果录音成功会收到开始发送语音的回调`OnStartSendAudioMessage`，无论录音是否成功都会收到发送语音消息结果的回调`OnSendAudioMessageStatus`，`OnStartSendAudioMessage`在接收时间上比`OnSendAudioMessageStatus`快，常用于上屏显示。
   
   - `发送语音消息回调接口`：
     `virtual void OnStartSendAudioMessage(XUINT64 requestID,YIMErrorcode errorcode,const XString& text, const XString& audioPath, unsigned int audioTime)`
     
     `virtual void OnSendAudioMessageStatus(XUINT64 requestID,YIMErrorcode errorcode,const XString& text,const XString& audioPath, unsigned int audioTime, unsigned int sendTime, bool isForbidRoom,  int reasonType, XUINT64 forbidEndTime)`
     
   - `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
   - `errorcode`：错误码
   - `text`：语音转文字识别的文本内容，如果没有用带语音转文字的接口，该字段为空字符串
   - `audioPath`：录音生成的wav文件的本地完整路径
   - `audioTime`：录音时长(单位秒)
   
   - `sendTime`：消息发送时间
   - `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言
   - `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
   - `forbidEndTime`：若在频道被禁言，禁言结束时间
   
- **备注：**
  此套接口是异步操作，发送语音消息成功的标识是收到`OnStartSendAudioMessage`，或者`OnSendAudioMessageStatus`回调的错误码是Success；调用`StopAudioMessage`同步返回值是Success才能收到回调；语音消息发送成功，接收方会收到`OnRecvMessage`回调，能从该回调中下载语音文件。

### 8. 接收消息
![接收消息](https://www.youme.im/doc/images/im_receive_message.jpg)
>- 通过`OnRecvMessage`接口被动接收消息，需要开发者实现

- **调用示例：**

  ``` cpp  
  virtual void OnRecvMessage(std::shared_ptr<IYIMMessage> message) override
  {		
		YIMMessageBodyType msgType = message->GetMessageBody()->GetMessageType();
     if( msgType == YIMMessageBodyType::MessageBodyType_TXT ){
        IYIMMessageBodyText* pMsgText = (IYIMMessageBodyText*)message->GetMessageBody();
        
        std::cout<< "收到文本消息：" << pMsgText->GetMessageContent() << ",发送者：" << message->GetReceiveID() << ",接受者：" << message->GetSenderID() << std::endl;
     }
     else if ( msgType == YIMMessageBodyType::MessageBodyType_Voice ){
        IYIMMessageBodyAudio* pMsgVoice = (IYIMMessageBodyAudio*)message->GetMessageBody();
        std::cout<< "收到语音消息，文本内容：" << pMsgVoice->GetText() << ",消息ID：" << message->GetMessageID() << "附加参数：" << pMsgVoice->GetExtraParam() <<   "语音时长：" << pMsgVoice->GetAudioTime() << std::endl;
       
        //下载语音文件
        YIMManager::CreateInstance()->GetMessageManager()->DownloadFile(messageID,                                                    "savePath");
     } 
     ...          
  }
  ```	
  
- **备注：**
	收到此回调，可以根据消息基类的`GetMessageBody()->GetMessageType()`获取消息类型并进行相应处理。

### 9. 接收语音消息并播放
![接收语音消息](https://www.youme.im/doc/images/im_receive_message-2.jpg)
>- 通过`GetMessageBody()->GetMessageType()`分拣出语音消息`（MessageBodyType_Voice）`
>- ⽤`GetMessageID()`获得消息ID
>- 点击语音气泡，调用函数`DownloadFile`下载语音消息
>- 调用方调用函数`StartPlayAudio`播放语音消息

- **调用示例：**

  ``` cpp      
      // 下载语音
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->GetMessageManager()->DownloadFile(messageID,                                                    "savePath");
      
      // 播放语音,可在下载成功的回调里面播放语音
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->StartPlayAudio("path");      
  ```
  
- **相关接口和参数：**
  - `下载语音消息接口`：
    `YIMErrorcode DownloadFile(XUINT64 messageID, const XCHAR* savePath)`
    
  - `播放语音消息接口`：
    `YIMErrorcode StartPlayAudio(const XCHAR* path)`  
    
	- `messageID`：消息ID
	- `savePath`：语音文件保存路径
	- `path`：待播放文件路径

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)

- **回调接口与参数：**
  下载语音接口是异步操作，下载语音成功的标识是`OnDownload`回调的错误码为Success，调用`DownloadFile`接口同步返回值是Success才能收到`OnDownload`回调。成功下载语音消息后即可播放语音消息。
  
  - `下载回调接口`：
    `virtual void OnDownload( YIMErrorcode errorcode, std::shared_ptr<IYIMMessage> msg, const XString& savePath )`	
    
  - `errorcode`：错误码
  - `msg`：消息基类
  - `savePath`：保存路径

  - `播放完成回调接口`：
    `virtual void OnPlayCompletion(YIMErrorcode errorcode, const XString& path)` 
    
  - `errorcode`：错误码
  - `path`：文件路径

### 10. 登出IM系统
注销账号时，调用登出接口`Logout`登出IM系统。

- **调用示例：**

  ``` cpp            
      YIMErrorcode ymErrorcode = YIMManager::CreateInstance()->Logout();      
  ```
  
- **相关接口：**
	- `登出接口`：
	  `YIMErrorcode Logout()`  
	  
- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)  
	  
- **回调接口：**
  登出是异步操作，登出成功的标识是`OnLogout`回调的错误码为Success，调用`Logout`接口同步返回值是Success才能收到`OnLogout`回调。
  
  - `登出回调接口`：
    `virtual void OnLogout(YIMErrorcode errorcode)`
    
  - `errorcode`：错误码  

## 典型场景集成方案
主要分为世界频道聊天，用户私聊，直播聊天室；集成的时都需要先初始化SDK，登录IM系统

***流程图

### 启动应用，初始化IM SDK
应用启动的第一时间需要调用初始化接口。

- **接口与参数：**
  - `Init`：初始化接口
  - `appKey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `appSecurity`：用户游戏产品的密钥，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `packageName`：游戏包名

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	  
- **代码示例与详细说明：**
	- [Cocos2D-x示例](#初始化IM SDK)

### 登录应用时，登录IM系统
![登录界面介绍](https://www.youme.im/doc/images/im_login.png)
>- 点击`登录`按钮时，调用IM SDK登录接口。

- **接口与参数：**
	- `Login`：登录接口
	- `userID`：由调⽤者分配，不可为空字符串，只可由字母或数字或下划线组成，用户首次登录会自动注册所用的用户ID和密码
	- `passwd`：⽤户密码，不可为空字符串，由调用者分配，二次登录时需与首次登录一致，否则会报UsernamePasswordError
	- `token`：用户验证token，可选，如不使用token验证传入:""
	
- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)

- **代码示例与详细说明：**
	- [Cocos2D-x示例](#4. 登录IM系统)

### 典型场景
#### 世界频道聊天
![频道](https://www.youme.im/doc/images/im_room.png)
>- 进入应用后，调用加入频道接口，进入世界、公会、区域等需要进入的聊天频道。
>- 应用需要为各个聊天频道设置一个唯一的频道ID。
>- 成功进入频道后，发送频道消息，文本、语音消息等。

- **接口与参数**
	- `JoinChatRoom`：加入频道
	- `chatRoomID`：请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	
- **代码示例与详细说明：**
	- [Cocos2D-x示例](#5. 进入聊天频道)

#### 用户私聊
>- 成功登录IM系统后，可和其它登录IM的用户发私聊消息，文本、语音消息等；发文本消息接口的聊天类型参数使用 `1`-私聊类型
>- 调用流程查看[发送文本消息](#发送文本消息)，[发送语音消息](#发送语音消息)

#### 直播聊天室
>- 用户集成直播SDK后，导入IM SDK
>- 初始化IM SDK
>- 登录IM 系统
>- 进入指定的聊天频道
>- 发送频道消息（例如：弹幕式），调用流程查看[发送文本消息](#发送文本消息)

### 发送文本消息
![频道](https://www.youme.im/doc/images/im_room_send_btn.png)
>- 点击发送按钮，调用发消息接口，将输入框中的内容发送出去
>- 发送出的消息出现在聊天框右侧
>- 表情消息可以将表情信息打包成Json格式发送

- **相关接口与参数：**
	- `SendTextMessage`：发文字消息接口
	- `receiverID`：接收者ID,（⽤户ID或者频道ID）
	- `chatType`：聊天类型,私聊传`1`，频道聊天传`2`；需要频道聊天得先成功进入频道后发文本消息，此频道内的成员才能接收到消息
	- `text`：聊天内容
	- `attachParam`：发送文本附加信息
	- `requestID`：消息序列号，用于校验一条消息发送成功与否的标识

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)

- **代码示例与详细说明：**
	- [Cocos2D-x示例](#6. 发送文本消息)

- **拓展功能：**
发送消息时，可以将玩家头像、昵称、角色等级、vip等级等要素打包成json格式发送。

### 发送语音消息
![发送语音消息](https://www.youme.im/doc/images/im_send_voice_message.jpg)
>- 按住语音按钮时，调用`SendAudioMessage`
>- 松开按钮，调用`StopAudioMessage`接口，发送语音消息
>- 按住过程若需要取消发送，调用`CancleAudioMessage`取消发送

- **相关接口与参数：**
	- `SendAudioMessage`：发送语音消息接口
	- `StopAudioMessage`：结束录音接口
	- `CancleAudioMessage`：取消录音接口
	
	- `receiverID`：接收者ID（⽤户ID或者群ID）
	- `chatType`：消息类型，表示私聊还是频道消息
	- `requestID`：消息序列号，用于校验一条消息发送成功与否的标识
	- `extraParam`：发送语音消息附加信息，主要用于调用方特别需求

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)

- **代码示例与详细说明：**
	- [Cocos2D-x示例](#7. 发送语音消息)

### 接收消息
![接收消息](https://www.youme.im/doc/images/im_receive_message.jpg)
>- 通过`OnRecvMessage`接口被动接收消息，需要开发者实现，接收消息进行相应的展示，如果是语音消息需要下载语音，以及下载完成后的语音播放

- **相关接口与参数：**
	- `OnRecvMessage`：收消息接口

- **代码示例与详细说明：**
	- [Cocos2D-x示例](#8. 接收消息)

### 接收语音消息并播放
![接收语音消息](https://www.youme.im/doc/images/im_receive_message-2.jpg)
>- 通过`GetMessageType()`分拣出语音消息`（MessageBodyType.Voice）`
>- ⽤`GetMessageID()`获得消息ID
>- 点击语音气泡，调用`DownloadFile`接口下载语音文件
>- 调用方播放语音文件

- **相关接口与参数：**
  - `DownloadFile`：下载语音文件接口
  - `StartPlayAudio`：播放语音文件接口
   
	- `messageID`：消息ID
	- `savePath`：语音文件保存路径
	- `path`：待播放文件路径

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	
- **代码示例与详细说明：**
	- [Cocos2D-x示例](#9. 接收语音消息并播放)

### 登出IM系统
![更换账号](https://www.youme.im/doc/images/im_change_account.jpg)
>- 如下两种情况需要登出IM系统：
>- 注销账号时，调用`Logout`接口登出IM系统
>- 退出应用时，调用`Logout`接口登出IM系统

- **相关接口：**
	- `Logout`：登出接口

- **返回值：**
	- `YouMeIMErrorcode`：错误码，详细描述见[错误码定义](/doc/IMSDKCocosC++.php#错误码定义)
	
- **代码示例与详细说明：**
	- [Cocos2D-x示例](#10. 登出IM系统)
	



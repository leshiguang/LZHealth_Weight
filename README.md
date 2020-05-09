# LZHealth_Weight

## 一、SDK介绍

## 1.1 SDK组成说明
 - LSNetworkModule：乐智健康网络库，对开源库AFNetworking库的封装，提供统一的网络请求接口和序列化约定，提供https双向认证，请求参数签名等安全措施
 - LSHealth_WeightSDK：乐智健康体重库，对ios原生WKWebview的封装后体重页面的载体，支持Http-Cache、离线化加速访问
 - LSDeviceModule：乐智健康设备连接、蓝牙数据传输库，支持蓝牙设备绑定、长连接、重连等

## 1.2 iOS版本支持
- 支持的 iOS 版本为 8.0 及以上版本。

## 1.3 注意事项

 1. framework接入方式支持x86_64, i386, armv7, arm64,如果打包时不需要某个架构支持，可以使用脚本移除

```
# This script loops through the frameworks embedded in the application and
# removes unused architectures.
find . -name '*.framework' -type d | while read -r FRAMEWORK
do
if ( "$FRAMEWORK"="FrameworkDIR" )
then
echo $FRAMEWORK
else
#FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_NAME=$(/usr/libexec/PlistBuddy -c "Print :CFBundleExecutable" "$FRAMEWORK/Info.plist")
echo $FRAMEWORK_EXECUTABLE_NAME

FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"
echo $(lipo -info "$FRAMEWORK_EXECUTABLE_PATH")

FRAMEWORK_TMP_PATH="$FRAMEWORK_EXECUTABLE_PATH-tmp"
fi
# remove simulator's archs if location is not simulator's directory
case "${TARGET_BUILD_DIR}" in
*"iphonesimulator")
    echo "No need to remove archs"
    ;;
*)
    if $(lipo "$FRAMEWORK_EXECUTABLE_PATH" -verify_arch "i386") ; then
    lipo -output "$FRAMEWORK_TMP_PATH" -remove "i386" "$FRAMEWORK_EXECUTABLE_PATH"
    echo "i386 architecture removed"
    rm "$FRAMEWORK_EXECUTABLE_PATH"
    mv "$FRAMEWORK_TMP_PATH" "$FRAMEWORK_EXECUTABLE_PATH"
    fi
    if $(lipo "$FRAMEWORK_EXECUTABLE_PATH" -verify_arch "x86_64") ; then
    lipo -output "$FRAMEWORK_TMP_PATH" -remove "x86_64" "$FRAMEWORK_EXECUTABLE_PATH"
    echo "x86_64 architecture removed"
    rm "$FRAMEWORK_EXECUTABLE_PATH"
    mv "$FRAMEWORK_TMP_PATH" "$FRAMEWORK_EXECUTABLE_PATH"
    fi
    ;;
esac
echo "Completed for executable $FRAMEWORK_EXECUTABLE_PATH"
echo $(lipo -info "$FRAMEWORK_EXECUTABLE_PATH")
done
```

 2. 其它问题请直接发送到邮箱yong.wu@lifesense.com

## 二、接入方式
目前支持Cocoapods 和Framework两种接入方式
### 2.1 Framework接入方式
下载地址


### 2.2 Cocoapods接入方式

 1、 添加pod仓库
```ruby
source https://github.com/leshiguang/cocoapods.git
```
 2、 引入pod依赖
```ruby
pod LZHealth_Weight
```
## 三、开发流程及API说明
### 3.1 初始化SDK
```java
/**
 * 初始化蓝牙SDK
 * @param context
 */
void init(Context context);
```

### 3.2 账号打通
1、调用鉴权API
```java
/**
*  @param tenantId:  Integer 必传，外部用户所属的租户ID，不传默认为乐心
*  @param subscriptionId: Integer 必传，应用对应的租户订阅ID，同appType
*  @param associatedId:   String  必传，关联ID，外部用户唯一标识
*/
void authorize(Integer tenantId, Integer subscriptionId, String associatedId, AuthorizationCallback callback)
```
2、回调信息说明
```json
{
    "userId": 25118766,                                //用户ID
    "accessToken": "9ca7c0d16c5e42efae4e2a0e9c8fbaf1", //登录token
    "needInfo": false,                                 //是否需要补充信息（新注册用户为true）
    "associatedId": "2431812172",                      //外部用户标识，同入参
    "businessToken": "d1b06139d8cf0d2e9ac117d4114eb83d"//外部业务token，同入参
}
```
3、设置（或更新）用户信息
```java
/**
 * 设置用户信息
 *
 * @param userInfo
 */
void setUserInfo(final DeviceUserInfo userInfo);
```
### 3.3 获取支持的产品列表
```java
/**
 * 获取所有允许接入的产品列表
 * @param callback
 */
void getProduct(GetProductListCallback callback);

```
### 3.4 选择绑定设备方式
蓝牙SDK支持蓝牙搜索绑定、SN号绑定、扫码绑定三种方式，可按需选择
####3.4.1、扫描蓝牙绑定方式
1、开始搜索设备
```java
/**
 * 开始搜索设备，搜索时间大概需要10秒左右
 *
 * @param callback 搜索的回调,需要开发者聚合
 */
void searchDevice(DisplayProduct displayProduct, SearchResultCallback callback);
```
2、主动发起中断（停止）搜索
```java
/**
 * 停止搜索
 */
void stopSearch();

```
3、选择目标设备开始配对
```java
/**
 * 根据搜索选中的蓝牙设备来绑定；
 *
 * @param lseDeviceInfo
 * @param callback
 */
void bindDeviceBySearchResult(LSEDeviceInfo lseDeviceInfo, BindResultCallback callback);
```
4、中断绑定流程
```java
/**
 * 取消搜索绑定流程
 *
 * @param lseDeviceInfo
 */
void interruptBindDevice(LSEDeviceInfo lseDeviceInfo);
```

#### 3.4.2 扫码绑定方式
```java
/**
 * 绑定设备 (对服务器)
 */
void bindDevice(String qrcode, BindResultCallback callback);
```

#### 3.4.3 SN（串号）绑定方式
1、读取设备信息
```java
/**
 * 根据sn获取设备信息
 */
void getDeviceBySn(String sn, GetDevInfoCallback callback);

```
2、绑定
```java
/**
 * 绑定设备 (对服务器)
 */
void bindDevice(String qrCode, BindResultCallback callback);
```
#### 3.4.4 Qrcode绑定方式
1、读取设备信息
```java
/**
 * @param qrCode
 * @param callback
 */
void getDeviceByCode(String qrCode, GetDevInfoCallback callback);
```

2、绑定
/**
 * 绑定设备 (对服务器)
 */
void bindDevice(String qrCode, BindResultCallback callback);

### 3.5 解绑
```java
/**
 * 解绑
 * @param deviceId
 * @param callback
 */
void unBindDevice(String deviceId, OnResultCallback callback);
```

### 3.6 接收数据
1、开启数据接收服务
```java
/**
 * 开始接收数据
 */
void startDataReceive();
```
2、停止数据接收服务
```java
/**
 * 停止接收数据
 * @return
 */
boolean stopDataReceive();
```


### 3.7 体重集成
1、获取ViewController
```objective-c
UIViewController *controller = [LSWeightViewController createWeightViewController];
```

## 四、Demo



## 五、技术支持群
![avatar](support.png)

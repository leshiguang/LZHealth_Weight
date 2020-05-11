
## 一、SDK介绍

## 1.1 SDK组成说明
 - lifesense-android-service-X：乐智健康android服务注册与发现SDK，包括编译插件、服务注册Api，服务发现Api

 - lifesense-android-device-service：乐智健康设备连接、设备数据同步服务，包含蓝牙传输层、应用层各项功能

 - lifesense-android-webview：乐智健康webview库，支持Http-Cache、离线化加速访问，提供各类体重服务需要的桥接口

 - lifesense-android-weight-service: 乐智健康体重解决方案，提供体重数据上传、数据查询能力

 - lifesense-android-account: 乐智健康账号服务，提供第三方账号鉴权能力
## 1.2 Android版本支持
- 支持的 android 版本为 18 及以上版本。



## 二、接入方式
目前支持aar 和gradle依赖两种接入方式

### 2.1 准备工作
#### 2.1.1 准备申请材料：
- 确定应用接入的（企业）组织名称，并说明使用场景、用途、评估应用接入的量级
- 确定应用的package_name（乐心会对使用的app包进行合法性校验）
- 确定应用需要接入的设备型号列表
- 准备一个接入者公司的github账号（用于获取android开发sdk）
- 材料确定后，发送申请接入邮件，审批会在1个工作日内完成

申请成功将会收到两封邮件：
- [1] 收到乐心授权的tenantId、sequeneId
- [2] 收到github仓库开发合作者邀请信息（android开发关注信息）

### 2.1 aar接入方式
下载地址：https://github.com/orgs/leshiguang/packages?tab=packages

### 2.2 gradle接入方式

1、 添加maven仓库
```groovy
maven {  
    url "https://maven.pkg.github.com/leshiguang/maven-repository"  
    credentials {  
        username GITHUB_USERNAMNE  
        password GITHUB_TOKEN  
    }  
}

```
参数说明：
 - username为接入方申请appid时提交的github profile 名称
 - password为接入方在github生成的token， Creating a personal access token for the command line

2、在application工程中配置gradle插件
 ```java
 dependencies {  
         classpath "com.lifesense.android:lifesense-android-service-plugin:0.1.0"  
 }  
```
3、在基础工程中添加依赖
``` java
dependencies {  
    api 'com.lifesense.bluetooth:lifesense-ble-module:1.7'  
}  
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

## 四、DemoApplication

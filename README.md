# TabTestSDK接入文档
## 一、Tencent ABTest iOS SDK
TAB实验平台配套iOS SDK（TabTestSDK.framework）
## 二、SDK集成

TabTestSDK SDK推荐通过CocoaPods方式集成</br>
TabTestSDK SDK最低兼容系统版本 iOS 8.0</br>
TabTestSDK SDK依赖MMKV 三方库

### 通过CocoaPods集成

1、在工程的Podfile里面添加以下代码：

```text
source 'https://github.com/tgrowing/Tab_specs.git'
source 'https://github.com/CocoaPods/Specs.git'

pod 'TabTestSDK'
```
2、保存并执行pod install,然后用后缀为.xcworkspace的文件打开工程</br>
注意: 命令行下执行pod search TabTestSDK,如显示的TabTestSDK版本不是最新的，则先执行pod repo update操作更新本地repo的内容

## 2. SDK使用

### 2.1 导入头文件

在工程的AppDelegate.m文件导入头文件

```
#import <TabTestSDK/TabTestSDK-umbrella.h>
```
如果是Swift工程，请在对应bridging-header.h中导入

### 2.2 SDK初始化配置
TabTestSDKConfig为sdk配置对象，<font color=red>注意：</font>appid和guid不能为空
```objc
/// ABTest配置信息
@interface TabTestSDKConfig : NSObject <NSCopying>
/// 从TAB平台获取
@property (nonatomic, copy) NSString *appid;

/// guid 代表用户身份标识
@property (nonatomic, copy) NSString *guid;
/// 场景id
@property (nonatomic, copy, nullable) NSString *sceneId;

/// 设置用户标签属性
@property (nonatomic, copy, nullable) NSDictionary<NSString *, NSString *> *userProfiles;

/// 设置加载试验的layerCode， 请求试验的时候会只请求这几个layerCode.
@property (nonatomic, copy, nullable) NSArray<NSString *> *layerCodes;

/// 设置是否自动轮询拉取试验分流数据 YES：开启自动轮询 NO：关闭自动轮询 默认为：YES
@property (nonatomic, assign) BOOL autoPoll;

/// 设置网络请求的超时时间
@property (nonatomic, assign) int requestTimeout;
/// 网络环境，设置为Debug以后试验曝光和行为日志上报状态为"debug"，默认为release
@property (nonatomic, assign) TabTestEnvironment environmentType;

@end
```
### 2.2 创建SDK实例
<font color=red>注意：</font>可以通过传入不同的appid和sceneId来创建sdk多实例，目前一个APP仅支持一个appid，可以通过传入不同sceneId来创建多实例
```objc
/// 根据config创建一个SDK实例
/// @param config 配置信息
+ (instancetype)createSDKWithConfig:(TabTestSDKConfig *)config;
```
### 2.3 获取SDK实例
```objc
/// 通过appid和sceneId获取一个SDK实例，其中sceneId可以为空
/// @param appid appid
/// @param sceneId 场景Id
+ (instancetype)tabTestSDK:(NSString *)appid sceneId:(NSString * _Nullable)sceneId;
```
## 三、SDK使用之试验接口
### 3.1 启动试验SDK
创建完SDK实例以后，需要手动启动start方法。启动start方法会做这些事情：第一步先从本地获取缓存的试验列表信息；第二步会异步请求试验列表的信息。
另外，后台会每十分钟检查一次试验列表信息是否需要刷新，具体刷新频率由后台下发。
```objc
/// 启动SDK实例，在创建完SDK实例以后一定要手动启动SDK才行
/// @param handler 执行结果回调
- (void)startWithCompleteHandler:(void(^ _Nullable)(BOOL result))handler;
```
#### Demo实践
```objc
    /// 创建初始化配置
    TabTestSDKConfig *config = [[TabTestSDKConfig alloc] init];
    config.appid = @"appid";
    config.guid = @"guid";
    config.sceneId = @"sceneId"; //如果没有可以为空
    config.environmentType = TabTestEnvironment_Debug;//切到debug环境
    config.userProfiles = @{@"sex" : @"male"};
    /// 创建SDK实例
    TabTestSDK *defaultSDK = [TabTestSDK createSDKWithConfig:config];
    /// 获取SDK实例，如果sceneId不存在，则为空
    defaultSDK = [TabTestSDK tabTestSDK:@"appid" sceneId:@"sceneId"];
    [defaultSDK startWithCompleteHandler:^(BOOL result) {
        NSLog(@"sdk start result = %d", result);
    }];
```

### 4. 获取实验数据

#### 4.1 通过实验标识获取实验数据，并上报一次实验曝光。

``` objc
exp = [sdk getCachedExpAndReport:@"exp1"];
if ([exp.assignment isEqualToString:@"MyExpTestVersionA"]) {
    // 实现实验版本A的逻辑
    // update View on UIThread or other logic
} else if ([exp.assignment isEqualToString:@"MyExpTestContorlVersion"]) {
    // 实验实验对照版本的逻辑
   // update View on UIThread or other logic
} else {
  // 默认版本的逻辑
}

```

#### 4.2 通过实验标识获取实验数据，不会上报实验曝光。

``` objc
RomaExp *exp = [sdk getCachedExp:@"exp1"];
if ([exp.assignment isEqualToString:@"MyExpTestVersionA"]) {
    // 实现实验版本A的逻辑
   // update View on UIThread or other logic
} else if ([exp.assignment isEqualToString:@"MyExpTestContorlVersion"]) {
  // 实验实验对照版本的逻辑
 // update View on UIThread or other logic
} else {
 // 默认版本的逻辑
}
```

#### 4.3 也可以通过层域code获取试验信息

``` objc
    
    // 通过层域标签获取试验信息
    RomaExp *exp = [sdk getExpByLayerCode:@"layer1"];
    // 通过层域标签获取试验信息，并且进行试验曝光
    exp = [sdk getExpByLayerCodeAndReport:@"layer1"];
    
```
### 5. 强制刷新试验列表接口
```objc
/// 强制发起一次网络策略拉取
/// @param handler 刷新结果是否成功
- (void)forceUpdateExps:(void(^ _Nullable)(BOOL result))handler;
```

### 6. 试验曝光
SDK提供API上报实验曝光事件。

``` objc
[sdk reportExpExpose:exp];
```
### 7. 行为日志上报
#### 瞬时行为采集
``` objc
// items是业务方可以携带自定义属性，items数组值的顺序要跟平台配置的字段保持一致，最多能传入20个自定义属性
NSArray *items = @[@"cancel", @"blue"];//依次写入button的名称、颜色
[sdk.momentEvent onEvent:@"btn_click" items:items];
```
#### 时长行为采集
针对计时类的行为采集，业务方可以通过durationEvent接口方法完成，支持计时开始、暂停、继续、结束等功能，业务方不需要
自己算时长。
``` objc
// 需要记录eventID，后续接口需要使用。
self.eventId = [sdk.durationEvent startDurationEvent:@"app_duration"];
// 当app退到后台的时候可以暂停计时。因为eventId是唯一的，因此可以避免两个相同事件都在计时的时候，记错的情况
[sdk.durationEvent pauseDurationEvent:self.eventId];
// 当app激活的时候，可以继续计时
[sdk.durationEvent resumeDurationEvent:self.eventId];
// 当app要结束生命周期的时候，停止计时并进行上报数据
// items是业务方可以携带自定义属性，items数组值的顺序要跟平台配置的字段保持一致，最多能传入19个自定义属性(因为被时长占了一个)
[sdk.durationEvent endDurationEvent:self.eventId items:@[@"demo_app"]];
```
### 8. 切换账号
可以在使用过程中切换账号，切换guid的时候，重新对试验信息进行一次远端拉取操作
```objc
/// 切换用户id
/// @param guid 账号id
/// @param handler 操作结果是否成功
[sdk switchGuid:@"newGuid" completeHandler:^(BOOL result) {
    NSLog(@"SDK切换实验数据完成 %d", result);
}];
```
## 四、版本更新日志
1.0.0版本 更新日期：
* 支持ABTest基础能力

ISC

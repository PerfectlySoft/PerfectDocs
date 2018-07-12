# iOS Notifications

APNs 远程消息函数库。该组件用于您的服务器向iOS / macOS设备推送消息。

首先修改 Package.swift file：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Notifications.git", majorVersion: 3)
```

## 综述

该程序在服务器上运行，通常是苹果移动设备在启动后，会自动在苹果网站上注册，以获得一个设备编号。

苹果设备需要在获得该编号之后，主动到⚠️**您的**⚠️服务器上登记。这样您的服务器就可以记录这个设备编号然后通过APN苹果网络服务给设备发消息。


## 获取 APN 授权码

要使用苹果设备消息推送服务APN，您需要首先获得一个APN授权代码。这个代码可以在苹果开发者网站上取得。登录到您的开发者账号并选择 "Certificates, IDs &amp; Profiles" 菜单，在"Keys"下，选择"All"。

如果还没有创建或者下载授权码，请点击”➕“创建一个新代码。输入私有钥匙的名称，并选择 **Apple Push Notifications service (APNs)**，即 Apple推送通知服务（APNs）。该授权码可以同时在开发阶段和生产部署阶段使用，适用于所有您的iOS / macOS设备。

点击 "Continue" 后，再点击"Confirm"，可以下载 **私有钥匙** 。此时必须下载并保存该文件，并拷贝屏幕上十位字长的“钥匙代码”。

最后需要确定您的开发团队账号。点击"Account"浏览器上方的，选择 "Membership" 会员视图能够找到"Team ID"团队编号，也是10位字符串。请记录该字符串。

## 服务器配置

如果需要从服务器向苹果设备推送消息，您需要以下三个信息：

1. 刚才下载的私有钥匙文件
2. 10位授权码
3. 10位开发团队代码
4. 苹果设备或操作系统在APN上的编号

上述内容用于推送消息，注意不要在您的服务器源代码上保存这些东西，最好区别与服务器程序分开保存。为了简单说明，本手册假设钥匙文件在当前工作目录下，并且各个编号代码都已经以Swift变量形式导入到程序。

在服务器代码开始段，请导入函数库 `import PerfectNotifications`。随后请配置好上述内容用于实际消息推送。

``` swift
import PerfectNotifications

// 应用程序名称(即Bundle Identifier)，我们用这个名称来配置，但是不一定非得是这个形式
let notificationsAppId = "my.app.id"

let apnsKeyIdentifier = "AB90CD56XY"
let apnsTeamIdentifier = "YX65DC09BA"
let apnsPrivateKeyFilePath = "./APNsAuthKey_AB90CD56XY.p8"

NotificationPusher.addConfigurationAPNS(
	name: notificationsTestId, 
	production: false, // 如果是调试程序，则将这个生产服务器标志设置为“伪”
	keyId: apnsKeyIdentifier, 
	teamId: apnsTeamIdentifier, 
	privateKeyPath: apnsPrivateKeyFilePath)
```

配置完成后就可以随时推送消息了。具体做法是创建一个`NotificationPusher`对象，设置应用程序代码或者“消息主题”，然后调用 `pushAPNS`函数：

``` swift
let deviceIds: [String] = [...]
let n = NotificationPusher(apnsTopic: notificationsTestId)
n.pushAPNS(
	configurationName: notificationsTestId, 
	deviceTokens: deviceIds, 
	notificationItems: [.alertBody("Hello!"), .sound("default")]) {
		responses in
		print("\(responses)")
		...
}
```

创建 NotificationPusher 的消息主题是必要的。此外，可选参数还包括消息有效期、优先级和消息层次折叠代码，详见苹果APN有关文档。

## 函数界面

详细函数调用参考如下：

```swift
public class NotificationPusher {
	
	/// 追加APNS 配置
	public static func addConfigurationAPNS(
		name: String, 
		production: Bool, 
		keyId: String, 
		teamId: String, 
		privateKeyPath: String)

	/// 初始化推送实例并设置消息主题
	public init(
		apnsTopic: String,
		expiration: APNSExpiration = .immediate,
		priority: APNSPriority = .immediate,
		collapseId: String? = nil)
		
	/// 推送消息
	/// 需要提供已经设定好了配置名称和设备编号
	/// 需要提供待通知对象列表 APNSNotificationItems
	/// 需要提供推送结果回调函数
	public func pushAPNS(
		configurationName: String, 
		deviceToken: String, 
		notificationItems: [APNSNotificationItem], 
		callback: @escaping (NotificationResponse) -> ())
	
	/// 向多个设备推送消息
	/// 需要提供预制配置名称、一个或者多个设备代码。所有设备都会收到相同的消息
	/// 需要提供待通知对象列表 APNSNotificationItems
	/// 需要提供推送结果回调函数
	public func pushAPNS(
		configurationName: String, deviceTokens: [String],
		notificationItems: [APNSNotificationItem],
		callback: @escaping ([NotificationResponse]) -> ())
}
```

其他数据结果包括 APNSNotificationItem ：

```swift
/// Items to configure an individual notification push.
public enum APNSNotificationItem {
    /// alert body child property
	case alertBody(String)
    /// alert title child property
	case alertTitle(String)
    /// alert title-loc-key
	case alertTitleLoc(String, [String]?)
    /// alert action-loc-key
	case alertActionLoc(String)
    /// alert loc-key
	case alertLoc(String, [String]?)
    /// alert launch-image
	case alertLaunchImage(String)
    /// aps badge key
	case badge(Int)
    /// aps sound key
	case sound(String)
    /// aps content-available key
	case contentAvailable
	/// aps category key
	case category(String)
	/// aps thread-id key
	case threadId(String)
    /// custom payload data
	case customPayload(String, Any)
    /// apn mutable-content key
	case mutableContent
}

public enum APNSPriority: Int {
	case immediate = 10
	case background = 5
}

/// 消息有效期，超过之后不会在发送
public enum APNSExpiration {
	/// 如果无法立刻到达则丢弃消息
	case immediate
	/// 当前时间加秒数
	case relative(Int)
	/// UTC绝对时间
	case absolute(Int)
}

/// 推送后的响应对象
public struct NotificationResponse: CustomStringConvertible {
	/// The response code for the request.
	public let status: HTTPResponseStatus
	/// The response body data bytes.
	public let body: [UInt8]
	/// The body data bytes interpreted as JSON and decoded into a Dictionary.
	public var jsonObjectBody: [String:Any]
	/// The body data bytes converted to String.
	public var stringBody: String
	public var description: String
}
```

## 其他注意事项

APNs 请求需要从您的服务器向苹果服务器 "api.development.push.apple.com" 或者 "api.push.apple.com" 443 端口进行通信。一次请求可以向一个或者多个设备推送消息。整个连接过程会一直持续，并且可以用于发送后续消息。如果连接终止或者空闲太久，则会重新打开一个新的连接。这种方式是苹果建议的方式用于处理大量同步通知。

可以参考代码示范 [Perfect-NotificationsExample](https://github.com/PerfectExamples/Perfect-NotificationsExample) 用于配置您自己的服务器和应用程序消息推送。

# 这个工具已经不推荐使用了！！

原因是，SFViewController的方案在iOS10以后，Openurl事件会以各种方式被系统吞吃掉事件，duplicate事件，等等，因为有了更好的替换方案

__所以非常非常不推荐继续使用这个SafariAutoLogin方案了__


新的方案请使用剪切板方案，详情看说明

[iOS app与浏览器深度链接 DeeperLink](http://awhisper.github.io/2016/05/11/iOSBrowserDomainBridge/)

># Update 2016.11.23

>更新一个最新的方案，老文章里面介绍的2个方案都不如这个易操作，唯一的问题是就看iOS10的覆盖率了，覆盖率不够之前下文的`设备指纹方案`可以与这个方案互补

>- 只支持iOS10以上
>- 通过剪切板
>- 跨越浏览器app与宿主app，传递数据

>[clipboard.js](https://github.com/zenorocha/clipboard.js)

>一个1w Star的js库

>因为iOS10系统给js开放了API可以操作剪切板，因此直接使用这个库，可以很方便的在任何App的webview（包含safari，微信，qq，百度框等）将数据存入剪切板

>然后很方便的在其他任何app中，读取出剪贴板的数据用于互通

>这个方案在准确度上，成功率上，适用范围上全面优于下文讨论的`设备指纹`和`SafariViewController方案`

>忘掉`SafariViewController方案`吧，这个方案现在非常的坑

>原因是，SFViewController的方案，Openurl事件会以各种方式被系统吞吃掉事件，duplicate等等，都会导致失败率非常的高

# 以下为原说明文档，不推荐使用，不推荐使用，不推荐使用



感谢SafariAutoLoginTest这个demo项目提供的思路

我把它从demo展示，封装成了易用的工具

[SafariAutoLoginTest Github地址](https://github.com/mackuba/SafariAutoLoginTest)

# VKSafariDomainBridge

按理说整个流程应该分为2部分

- wap页面功能：
	- 用户浏览wap页的存cookie （wap地址1，用户访问用的）
	- 隐藏safari浏览的读cookie页面（wap地址2，静默代码访问用的）
	- 隐藏safari跳转openurl功能
- app内功能：
	- 打开隐藏safari
	- 收听openurl的回调，处理数据
	- 关闭隐藏safari

鉴于实在是不会h5相关的开发，所以我封装的工具就只包含app内的功能，H5部分已经与朋友联调通过，确认可行

```objectivec
//初始化 VKSafariDomainBridge
NSURL *url = [NSURL URLWithString:@"wap地址2，静默代码访问用的url"];
NSString *key = @"xxkey"
[VKSafariDomainBridge VKSetupSafariBridgeUrl:url AndKey:key];
```
url的地址就是`wap地址2，静默代码访问用的`
key作为协议识别关键字，隐藏safari发起的跳转，通过这个key识别，才会走入VKSafariDomainBridge的处理逻辑，如果是其他正规渠道的openurl跳转，key不匹配，便直接走正常逻辑，不会进行VKSafariDomainBridge处理

想要获取wap用户数据的时候

```objectivec
[[VKSafariDomainBridge VKSingleton]VKGetSafariInfo:^(BOOL success, NSString *info) {
        NSLog(@"%@ status = %@",info,@(success));
}];
```
通过回调，如果成功success会返回YES，并且整个跳转含有数据的url会被转成string，通过block返回，如果失败，则会返回NO

# application:openURL:options:处理

整个工具需要通过appdelegate的openurl回调来实现，因此写成category形式，只要导入工程，便可以一行代码不需要写，自动生效。

`application:openURL:options:`这个方法，如果没有在工程中用到，会自动添加，保证了openurl回调可以正常工作。

如果已经在工程中使用，已经有很多openurl协议要处理了，那么我的category会生成一个新方法，交换掉老的系统函数（MethodSwizzling），先判断url协议里是否含有上面提到的专属`Key`，含有则走我的处理逻辑，如果不含有，调用老函数，保证原项目功能无异常。

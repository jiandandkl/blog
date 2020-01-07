---
uuid: 154876d0-2eaa-11ea-91cc-d3a0c5ead53d
author: jiandandkl
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars2.githubusercontent.com/u/16009933?v=4
title: 两个半月的业余时间用Flutter做了个app(技术篇)
date: 2020-01-03 17:37:46
tags: flutter
---

> 技术背景: 做了几年前端,会用node
> 写这篇文章是自己对这段时间做个技术总结,记录一些开发过程中比较难以解决的问题和经验,同时希望对Flutter感兴趣但还在观望的同学加入Flutter开发,简单易上手

**学习资料**
[Flutter 实战](https://book.flutterchina.club/), 入门可以看这本电子书,很多flutter知识都是从这里学习到的,在此感谢下作者

## 一些小结

#### 1. 路由跳转
1. 注册路由
main.dart

```dart
MaterialApp(
  routes:{
    'my_page': (context) => MyPage(),
    ...
  }
)
```

2. 点击跳转
xxx.dart

```dart
  FlatButton(
    onPressed: () {
      // 可通过arguments传递参数
      Navigator.pushNamed(context, 'my_page', arguments: {'data': data});
    }
  )
```

#### 2. 循环widget
Row(children: strings.map((item) => Text(item)).toList());

#### 3. 宽高设置
前端开发中html标签是可以自行设定宽高,颜色,但在flutter中只有几个widget可以(刚开始接触有点不习惯)

```dart
// 可设置宽高及颜色
Container(
  // 满屏
  width: MediaQuery.of(context).size.width,
  height: 200,
  color: Colors.blue
)
// 可设置宽高
SizedBox(
  width: 200,
  height: 200
)
```

* 如果要给按钮设置大小,便可通过在外包裹个Container来控制大小:

```dart
Container(
  width: 80,
  height: 30,
  child: FlatButton()
)
```

* 也可以使用`ButtonTheme`来修改按钮大小:

```dart
ButtonTheme(
  minWidth: 65.0,
  height: 24.0,
  child: OutlineButton()
)
```

* 图片可自行设置宽高

#### 4. 区分环境
把启动文件按环境分为main_local,main_dev.dart,main.dart,根据需要启动不同文件,详见 [Flutter 实现根据环境加载不同配置](https://yuanxuxu.com/2018/09/13/flutter-load-config-by-env/)

* [android studio设置运行环境](https://www.jianshu.com/p/b9e7c00075e1), 设置后可以很快切换环境
![](/img/dujun/dev.png)

* vscode 设置`launch.json`文件的`"program": "lib/main.dart"`为不同的启动文件即可

> 不过带来的问题是涉及`main.dart`的修改都需要改三份文件

#### 5. flutter封装的组件
flutter封装了一些组件,可以让开发者专注于业务,如TabBar, DatePicker等

#### 6. dio增加配置信息
token,版本号,ip等信息需要由app传到服务端,不能每次请求都获取一遍这些信息,可以统一设置一次
dao_basis.dart
```dart
Dio dio = Dio()..interceptors.add(InterceptorsWrapper(
  // 请求时的处理
  onRequest: (RequestOptions options) async {
    String _platform = 'android';
    String _version = packageInfo.version;
    options.headers..addAll({
      'platform': _platform,
      'version': _version,
    })
  }
  // 相应时的处理
  onResponse: (Response options) async {
    // 对响应状态码如401的处理
    if (options.data['code'] == 401) {
      ...
    }
  }
  // 发生错误时的处理
  onError: (DioError err) {
    ...
  }
))
```

user_dao.dart

```dart
// 其他请求引用上个文件即可
import 'dao_basis.dart';
Response res = await dio.get('xxx);
```

#### 7. ios证书设置指南
[极光推送文档](https://docs.jiguang.cn/jpush/client/iOS/ios_cer_guide/), 比腾讯信鸽的详细且新

## 奇怪的问题
1. 连接真机失败
[Error connecting to the service protocol: HttpException](https://github.com/flutter/flutter/issues/25112)
刚开始切换wifi还好使,后来渐渐这招失败次数也多了;意外发现重新连接下手机就可以了,不过最近发现这招也有不行的时候了,虽然失败次数少。

2. `Check failed: vm. Must be able to initialize the VM`
忘了改了什么,用vscode运行和xcode运行都不行,用android studio试了下居然可以了,也是很奇怪

3. 安卓真机运行不了
先运行安卓模拟器,再运行真机

4. `MissingPluginException(No implementation found for method startWork on channel xxx)`
新安装的包,有时需要到ios目录`pod install`

## 使用的第三方包
罗列下用到的第三方包(常用的如dio,flutter_swiper等就不列了),省的大家再去寻找啦

#### 弹窗
[fluttertoast](https://pub.dev/packages/fluttertoast#-readme-tab-)
使用很方便,不需要传`context`,引入后随意调用;
不过貌似取消不了,详见issue [Fluttertoast.cancel() not work](https://github.com/PonnamKarthik/FlutterToast/issues/116)

#### loading
进行网络请求等操作时的loading状态
[modal_progress_hud](https://pub.dev/packages/modal_progress_hud)

#### 网络图片处理
缓存网络图片,且带有loading的placeholder
[cached_network_image](https://pub.dev/packages/cached_network_image)

#### 下拉刷新
下拉刷新及上拉加载
[flutter_easyrefresh](https://pub.dev/packages/flutter_easyrefresh)

**app名为小鱼干,主要功能是提供上门喂猫服务,欢迎下载使用**
![](/img/dujun/misym-9zeqw.gif)



# 2024年安卓平台集成WebView 方案的最佳实践, 以及一些问题的解决方案

## 1。 WebView 方案的选型

- Android WebView
- GeckoView
- Crosswalk
- Webview_instrumentation_apk
- X5内核

## 2。 WebView 方案的对比

| 方案                          | 优点                                                                       | 缺点                                                                                                                                                                              |
|-----------------------------|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Android WebView             | 1. 系统自带，无需额外集成<br> 2.版本越高渲染性能越好，对css/js的支持能力更强                           | 1. 60~120 的各个版本的WebView 兼容性难以处理， 并且有一些品牌手机无法通过安装的方式升级WebView  <br> 2. disk cache缓存大小限制在20M，照片太多会造成缓存失效的额外请求 <br> 3. 相机适配是一个很大的问题， 开源的扫码JS，对于WebView的兼容性是灾难性的。                   |
| GeckoView                   | 1. 作为独立的库引入项目，可以获得源码支持<br> 2. 对缓存的支持更强， 比WebView开放的设置更加丰富， 比如缓存，代理，内容拦截等 | 1. 对文件选择依然有一些BUG， 但是影响不大 <br>2. 性能弱于WebView的方案  <br>3. 很多网站对firefox 引擎支持不太好  <br> 资料相对于系统WebView太少，Android开发者适配有难度，需要阅读源码。 <br> 4. 引入armv8的方案大约会增加60M的大小，不过可以自己修改源码达到动态下载so 的方案 |
| webview_instrumentation_apk | 1. WebView 基于该方案编译，网页兼容性强，接口设置基本能和WebView保持一致性 <br> 2. 有源码方便排查问题         | 1. 使用软解码模式容易掉帧， 使用硬解码进入多任务后直接黑屏 <br>2. 性能比WebView 差不少  <br> 3. 编译耗时太长， 开源自构建的项目已经停止维护                                                                                           |
| X5内核                        | 1. 微信的小程序使用该方案，兼容性强 <br> 2. 内核更新后渐渐的跟上了Chromium 的版本号，性能不会太差              | 1. 没有源码支持，出现问题后，可能无法得到X5团队的支持 <br> 2. 对于商业应用可能有一定的风险。                                                                                                                           |
| Crosswalk                   | 1. 2017年停止维护                                                             | 1。 2017年停止维护                                                                                                                                                                    |

## 方案排名

1. Android WebView ， 需要兼容各个厂商WebView， 能够容忍某些手机某些功能可能无法使用的情况下， 可以使用该方案。 ( 兼容40。X~
   120。X 的版本是一个非常大的挑战)
2. GeckoView， 作为独立的库引入项目，可以获得源码支持，但是性能弱于WebView的方案， 统一网页渲染的版本给开发者人员带来了很大的便利，
   但是集成成本较高， 需要Android后端开发者的支持。 如果团队能力强， 可以考虑使用该方案。
3. 使用Android WebView + GeckoView 的混合方案。 通过判断手机WebView版本来决定使用哪个方案。 比如手机的WebView 版本版本低于90。X，
   则使用GeckoView， 否则使用WebView。 在需要使用兼容更多陈旧的设备的时候， 可以考虑使用该方案。
4. webview_instrumentation_apk 公司有人才，有时间，有精力，有资源，可以考虑使用该方案。 但是需要注意的是， 该方案的工作非常繁琐，
   需要自己编译，自己维护，自己解决问题。
5. X5个人项目可以使用， 商业项目不建议使用。 无法获取源码支持， 无法保证安全性。

总结: 其中前三个方案是比较靠谱的。 从利益最大化的角度个人推荐使用方案3， 并且方案3的实现也是比较简单的，我们团队采用的就是该方案。
第4种方案， 鉴于国内只有X5和U4内核商业化了，并且是2大巨头，我个人不觉得国内会有小公司去使用第4种方案。

## WebView 一些问题

### 页面白屏

1. 页面JS语法，不兼容。 修改js到兼容的语法
2. 页面的JS 加载失败。排查网络问题

对于一些老的设备， 无法升级到最新的系统， 无法升级到最新的WebView，如何处理网页的语法兼容性， 甚至是要修改一些依赖库的源码，
这是一个非常大的挑战。
当然无论是Android的兼容性问题，还是WebView的兼容性问题，还有很重要的就是WebView在部分手机重多任务再次进入后白屏(非常严重),都需要开发者自己去解决， 或者向一些技术咨询公司寻求帮助。

### WebView 集成

1. h5页面和原生页面的交互
2. 调用WebView摄像头
3. 调用WebView的文件选择器
4. WebView 实现全局的代理或者实现全局自定义DNS解析
5. WebView 获取地理位置
6. WebView 套壳,WebView加载本地资源
7. WebView自定义错误页面

上面的这些都是WebView 需要适配的内容

### WebView 为什么快?

WebView的一部分进程是系统进程，和其他方案有天然的优势， 提前初始化，内存占用也少。

## 参考地址:
1. GeckoView 官方地址  https://github.com/mozilla/geckoview
2. CrossWalk(chromium_53)官方地址 https://github.com/crosswalk-project/crosswalk
3. CrossWalk(chromium_77)第三方维护版本 https://github.com/ks32/CrosswalkNative
4. webview_instrumentation_apk https://chromium.googlesource.com/chromium/src/+/master/docs/android_build_instructions.md
5. 自编译webview_instrumentation_apk https://github.com/ridi/chromium-aw
6. X5内核官方地址 https://x5.tencent.com/tbs/guide/sdkInit.html

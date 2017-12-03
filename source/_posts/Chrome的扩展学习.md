---
title: Chrome的扩展学习
tags:
  - Chrome App
  - Chrome Extension
  - NaCl
  - Native Client
  - PNaCl
  - Portable Native Client
id: 729
categories:
  - 学习
date: 2015-04-22 21:55:16
---

之前做过一些Chrome Extension方面的开发，最近无意间看到Chrome已经不再支持NPAPI插件了（[Saying Goodbye to Our Old Friend NPAPI](http://blog.chromium.org/2013/09/saying-goodbye-to-our-old-friend-npapi.html)），查看其文档，现在做Chrome的扩展开发主要就是Chrome App、Chrome Extension、Native Client三种了。<!--more-->

Chrome Extension相对还算熟悉，新出现的Chrome App其实可以这样简单理解，是一种扩展的Chrome Extension，启动时不再在Chrome的工具栏（[Browser Actions](https://developer.chrome.com/extensions/browserAction)）、地址栏（[Page Actions](https://developer.chrome.com/extensions/pageAction)）、页面右键菜单项中添加功能，而是直接打开一个新的窗口，然后在里面运行由HTML、CSS、JavaScript的应用程序。还有一点是Manifest的版本现在是2，App是从[Manifest版本](https://developer.chrome.com/extensions/manifestVersion)2才开始支持的，所以App里的<span class="pl-s"><span class="pl-pds">"</span>manifest_version<span class="pl-pds">"必须是</span></span> <span class="pl-c1">2，Extension的<span class="pl-pds">"</span>manifest_version<span class="pl-pds">"可以是1或2，不过官方建议使用2。</span></span>

另外App分为Hosted Apps和Packaged Apps，Hosted顾名思义就是应用托管到服务器上（也有crx，但crx里只有manifest文件），Packaged就是把所有资源都打包到crx中，然后下载到Chrome里安装运行。以下引用官方的解释（[Extensions and Apps in the Chrome Web Store](https://developer.chrome.com/webstore/apps_vs_extensions)）。
> ## Hosted Apps, Packaged Apps, and Extensions
> 
> There are actually two kinds of apps: hosted and packaged. A hosted app wraps an online website, so the CRX package can be as simple as a single manifest.json file pointing to the website. A packaged app contains the whole kit and kaboodle inside the CRX package—HTML, CSS, and so on, all run from the user’s hard drive.
> 
> 
> Packaged apps are a kind of missing link between extensions and hosted apps. They look the same as a hosted app to the user, but under the covers, they are really like traditional extensions with that special “launch” parameter. They have access to almost all functionality afforded to regular extensions—context menu, background pages, and so on. The only exception is that packaged apps can't add buttons to the address bar.
> 
> 
> Returning to the example in the previous section, it’s perfectly valid for a packaged app to add an item to Google Chrome's [context menu](http://code.google.com/chrome/extensions/contextMenus.html). However, it’s completely invalid for a hosted app to do the same thing. In some respects, a packaged app lets you have your cake and eat it: the appearance of a packaged app with the power of an extension. But there are still plenty of reasons to use pure extensions and hosted apps.
体验了几个Chrome App，还是非常酷炫的，如[Any.do](https://chrome.google.com/webstore/detail/anydo/ocgddccilgpeepgglnlpchkpgamkgmld)、[Mancala](https://chrome.google.com/webstore/detail/mancala/cjlhjhpnhabnfepdfemepiilbjbkecpe)，后面有机会开发跨平台的桌面应用（游戏）就可以用Chrome App尝试！

而Native Client（还分为Portable Native Client：PNaCl和普通的Native Client：NaCl, Chrome使用的是PNaCl）则是稍微底层一点的东西，即可以使用C、C++来调用更底层的一些API，如3D Graphic、File I/O、Audio等，它是作为App和Extension可选的模块放到crx里运行的。
> **Native Client** is a sandbox for running compiled C and C++ code in the browser efficiently and securely, independent of the user’s operating system. **Portable Native Client** extends that technology with architecture independence, letting developers compile their code once to run in any website and on any architecture with ahead-of-time (AOT) translation.
> 
> 
> In short, Native Client brings the **performance** and **low-level control** of native code to modern web browsers, without sacrificing the **security** and **portability** of the web. Watch the video below for an overview of Native Client, including its goals, how it works, and how Portable Native Client lets developers run native compiled code on the web.
 

其实在[Saying Goodbye to Our Old Friend NPAPI](http://blog.chromium.org/2013/09/saying-goodbye-to-our-old-friend-npapi.html)中他提到代替NPAPI的方式（见下面引用），只是[Native Messaging API](http://developer.chrome.com/extensions/messaging.html#native-messaging)是属于Extension的，[Legacy Browser Support](https://support.google.com/chrome/a/answer/3019558?hl=en)用的很少，简单阐述就是让Chrome不能处理的旧的网页调用其他浏览器（如IE）去处理。
> There are several alternatives to NPAPI. In cases where standard web technologies are not yet sufficient, developers and administrators can use [NaCl](https://developers.google.com/native-client/), [Apps](http://developer.chrome.com/apps/), [Native Messaging API](http://developer.chrome.com/extensions/messaging.html#native-messaging), and [Legacy Browser Support](https://support.google.com/chrome/a/answer/3019558?hl=en) to transition from NPAPI. Moving forward, our goal is to evolve the standards-based web platform to cover the use cases once served by NPAPI.
参考资料
1\. [Chrome App](https://developer.chrome.com/apps/about_apps)
2\. [Chrome Extension](https://developer.chrome.com/extensions)
3\. [Native Client](https://developer.chrome.com/native-client)
4\. [Choosing an App Type](https://developer.chrome.com/webstore/choosing)
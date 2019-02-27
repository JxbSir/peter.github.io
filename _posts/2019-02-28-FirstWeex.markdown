---
layout: post
title: iOS项目中集成Weex
date: 2019-02-28 08:00:00
tags: iOS
---

# 设置Podfile

```
source 'git@github.com:CocoaPods/Specs.git'
target 'FirstWeex' do
    platform :ios, '8.0'
    pod 'WeexSDK', '0.20.1'
end

```

> 网络不好的情况，`pod install`的时候会在`git clone`卡住，原因就是github在海外，所以我们此时最好是通过VPN代理
> 
> 配置下`git config`
> 
```
git config --global http.https://github.com.proxy socks5://127.0.0.1:1086
```
> > **1086** 是端口号，请看下自己机器的sock代理端口号
> > 
![](http://xbqn.nbshk.cn/20190227143823_ejnXVB_Screenshot.jpeg)
> ok，此时`pod install`就瞬间成功
> 

# 安装Weex环境

若未安装npm：`brew install npm`

安装weex：

```
npm install weex-toolkit -g
```

然后通过weex命令创建一个空模板：

```
weex create WeexDemo
cd WeexDemo
```

在src文件里面就是默认的vue文件，通过`npm start`启动web看到效果

![](http://xbqn.nbshk.cn/20190227175204_mtBhLq_Screenshot.jpeg)

通过`npm run build`可以将vue打包成bundlejs文件，会自动生成在dlist文件夹下面，将生成的index.js文件放到iOS工程里面

![](http://xbqn.nbshk.cn/20190227175404_AGAFMY_Screenshot.jpeg)

然后开始在代码中初始化weex，建议写在`didFinishLaunchingWithOptions `中

```
WXAppConfiguration.setAppGroup("jinxuebin")
WXAppConfiguration.setAppName("firstweex")
WXAppConfiguration.setAppVersion("1.0")
WXSDKEngine.initSDKEnvironment()
```

然后去Controller创建一个weex实例

```
class ViewController: UIViewController {

    private var weexView: UIView?
    
    private lazy var weex: WXSDKInstance = {
        let weex = WXSDKInstance()
        weex.viewController = self
        weex.frame = self.view.frame
        return weex
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        weex.onCreate = { [weak self](wView) in
            guard let wView = wView else {
                return
            }
            self?.weexView?.removeFromSuperview()
            self?.weexView = wView
            self?.view.addSubview(wView)
        }
        weex.onFailed = { (error) in
            print(error)
        }
        weex.renderFinish = { (_) in
            print("finish")
        }
        
        
        let bundleUrl = Bundle.main.url(forResource: "index", withExtension: "js")!
        weex.render(with: bundleUrl, options: ["bundleUrl": bundleUrl.absoluteString], data: nil)
    }

    deinit {
        weex.destroy()
    }

}
```

启动项目成功，如图

![](http://xbqn.nbshk.cn/20190227175747_upRVhO_Screenshot.jpeg)

### 不过，有没有发现，图片并没有显示出来，我们得找找问题在哪里？

> 其实是因为Weex自身没有实现加载图片的功能，只是定义了协议`WXImgLoaderProtocol`，需要由使用者来实现
> 
> 我们来创建一个实现类`WeexImageLoader.swift`，我们就用三方库`SDWebImage`来举例
>

```
//
//  WeexImageLoader.swift
//  FirstWeex
//
//  Created by Peter on 2019/2/27.
//  Copyright © 2019 Peter. All rights reserved.
//

import UIKit
import WeexSDK
import SDWebImage

class WeexImageLoader: NSObject {

}

extension WeexImageLoader : WXImgLoaderProtocol {
    
    func downloadImage(withURL url: String!, imageFrame: CGRect, userInfo options: [AnyHashable : Any]! = [:], completed completedBlock: ((UIImage?, Error?, Bool) -> Void)!) -> WXImageOperationProtocol! {
        
        guard let URL = URL(string: url) else {
            return nil
        }
        
        return SDWebImageManager.shared()?.downloadImage(with: URL, options: [], progress: { (_, _) in
            
        }, completed: { (image, error, type, success, url) in
            completedBlock(image, error, success)
        }) as? WXImageOperationProtocol
    }
    
}
```

然后把实现类注册到Weex：

```
WXSDKEngine.registerHandler(WeexImageLoader(), with: WXImgLoaderProtocol.self)
```

然后启动，发现图片就加载出来了

![](http://xbqn.nbshk.cn/20190227192448_Lozi5y_Screenshot.jpeg)


### OK，第一个Weex项目就启动成功了
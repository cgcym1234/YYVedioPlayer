# YYVedioPlayer

## 环境

- Xcode 11.3
- Swift 5.1



## 需求

有时候在电脑上下周好了电影，但是想用手机看，而系统又没有自带这套操作的工具，

于是就干脆自己写一个吧，顺便练习下刚学的Swift UI。

> 说明一下，这篇文章主要是演示Swift UI，工程里面使用的播放控件是基于IJKMediaFramework封装好的一个ViewController，在Github上找的这个工程[ Swift-IJKPlayer](https://github.com/limxing/Swift-IJKPlayer)小改了下，懒得自己再写🙃，也练习了Swift UI中使用UIKit。

完整工程代码：[YYVedioPlayer](https://gitlab.com/cgcym1234/YYVedioPlayer.git)

先来看下最终效果：

![](http://120.79.102.161/Blogs/note/Blog/%E5%B7%A5%E4%BD%9C/pic/yyvideo/yyvideo.gif)



## 分析

需求很简单，思路也很简单，只需要2个界面就ok：

1. 一个播放器界面
2. 一个列表页可以展示指定目录的所有文件和子目录
   - 点击文件就将对应url传递给播放器界面
   - 点击目录就push一个新的列表页并展示对应的内容
   - 左滑删除功能

而我们打开App的时候，首先要展示的是Documents下面的内容，因为在Info.plist里面设置UIFileSharingEnabled = true后，电脑里的文件只能拷贝到App的这个目录。

下面就开始撸代码了。



## Service层

首先，创建一个VedioManager，提供文件模型和相关的方法：

- load方法根据传入的路径，返回File数组（目录放在数组前面，普通文件在后面）
- delete方法删除指定的File，以及它包含的内容

里面用到了YYFile，这个其实是[JohnSundell](https://www.swiftbysundell.com/) 写的库[Files](https://github.com/JohnSundell/Files)，很好用，我只是照着写了一遍，方便更好理解和使用。

> 另外他的博客全是Swift相关教程，很屌很炸天。

```
extension VedioManager {
    struct File: Hashable {
        let name: String
        let path: String
        let isFolder: Bool
    }
}

class VedioManager {
    static let root = App.dirDocument.path
    
    static func load(at path: String) -> [File] {
        if let folder = try? YYFile().createFolderIfNeeded(at: path) {
            let folders = folder.subfolders.map {
                File(name: $0.name, path: $0.path, isFolder: true)
            }
            
            let files = folder.files.map {
                File(name: $0.name, path: $0.path, isFolder: false)
            }
            
            return folders + files
        }
        
        return []
    }
    
    static func delete(_ file: File) -> Bool {
        do {
            try FileManager.default.removeItem(atPath: file.path)
            return true
        } catch {
            print(error)
        }
        
        return false
    }
}
```



## 另外2篇文章

### [2-列表页](http://120.79.102.161/Blogs/note/Blog/MyBlog/iOS/SwiftUI/%E4%BD%BF%E7%94%A8SwiftUI%E6%9E%84%E5%BB%BA%E8%A7%86%E9%A2%91App/2-%E5%88%97%E8%A1%A8%E9%A1%B5.md)

### [3-播放器页](http://120.79.102.161/Blogs/note/Blog/MyBlog/iOS/SwiftUI/%E4%BD%BF%E7%94%A8SwiftUI%E6%9E%84%E5%BB%BA%E8%A7%86%E9%A2%91App/3-%E6%92%AD%E6%94%BE%E5%99%A8%E9%A1%B5.md)


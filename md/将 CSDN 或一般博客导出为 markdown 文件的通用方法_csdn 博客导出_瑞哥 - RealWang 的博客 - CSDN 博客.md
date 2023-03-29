> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/wangrui1573/article/details/124662720)

> 环境：CSDN 博客或者其他博客  
> 问题：需要将文章导出为 MD 文件  
> 办法：使用浏览器 conesole 代码或者简阅

#### 1. 第一种方式比较简单、无脑，但是却不支持新版编辑器的文章

1.  登陆 CSDN，点击链接：[https://blog-console-api.csdn.net/](https://blog-console-api.csdn.net/)
    
2.  按 F12，打开浏览器 console, 粘贴下列内容回车，你会看到浏览器标题的数字变化，已经开始下载
    

```
var s=document.createElement('script');s.type='text/javascript';document.body.appendChild(s);s.src='//cdn.jsdelivr.net/gh/ame-yu/csdn-move@latest/dist/index.js';
```

3.  打开自动下载的压缩包，你会发现很多大小为 0 的 MD 文件，我一开始猜测可能是草稿或者未审核后的文字，后来去 github issue 上发现是编辑器版本的问题  
    ![](https://img-blog.csdnimg.cn/0f2ef21351514ca184bcdb12b2173ffc.png)
    
4.  这个是时候你需要选择下面的方法来转存这些为 0 的文件
    

#### 2. 第二种方式是使用[浏览器插件](https://so.csdn.net/so/search?q=%E6%B5%8F%E8%A7%88%E5%99%A8%E6%8F%92%E4%BB%B6&spm=1001.2101.3001.7020)简悦实现的，速度够快，但是要一篇篇操作，好过复制粘贴

1.  先安装简悦浏览器插件
    
2.  配置插件，授权 github，按教程生成 token 复制到简悦
    
3.  在 github 上创建一个仓库和目录
    
4.  配置写入目录  
    ![](https://img-blog.csdnimg.cn/8eefea85278d48f08cac0e5736ce945e.png)
    
5.  打开你要导出的文章，点击插件上的保存到 github  
    ![](https://img-blog.csdnimg.cn/c4db51f234864060846bed9f05cf0569.png)
    
6.  到 github 的对应目录查看并下载, 使用 git clone/push 或者直接浏览器下载即可  
    ![](https://img-blog.csdnimg.cn/0c5e91df352043488b495e288f451eae.png)
    
7.  这个方式是目前测试最快的了，有更快的欢迎在下面留言分享，谢谢~
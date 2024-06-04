## 下载安装
官网下载：地址 `https://www.sublimetext.com/`
输入注册码：打开`Sublime text`，然后点击菜单`Help->Enter Lisence:`
```
ZYNGA INC.
50 User License
EA7E-811825
927BA117 84C9300F 4A0CCBC4 34A56B44
985E4562 59F2B63B CCCFF92F 0E646B83
0FD6487D 1507AE29 9CC4F9F5 0A6F32E3
0343D868 C18E2CD5 27641A71 25475648
309705B3 E468DDC4 1B766A18 7952D28C
E627DDBA 960A2153 69A2D98A C87C0607
45DC6049 8C04EC29 D18DFA40 442C680B
1342224D 44D90641 33A3B9F2 46AADB8F
```

## 汉化
接着我们按Ctrl+Shift+P打开万能搜索框，然后输入`Install Package`
![image-20221122115707581](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20221122115707581.png)

稍等一下会重新打开一个搜索栏，搜索 `chinses`，选择`ChineseLocalizeitions`
- 等待提示安装完成
- 大功告成，如需切换语言选择help -> Languages

## 中文乱码问题
在`sublime`中打开后发现出现乱码，是因为`sublime text3`默认不支持`GB2312`和`GBK`
接着我们按`Ctrl+Shift+P`打开万能搜索框，然后输入`Install Package`
这时会加载所有的`Packages`列表，看到列表之后再输入`ConvertToUTF8`回车，就会下载安装这个包了
![image-20221122115747307](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20221122115747307.png)

装好之后会看到这个包的说明文件，然后重新打开文件，就大功告成了。

## 关闭更新提示
![image-20221122115808404](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20221122115808404.png)

要关闭sublime text 3 更新提示, 
- 1.sublime text 3 已注册(否则此方法无效);
- 2.点击菜单栏Preferences => Settings,
修改右边的`User Settings`,添加一行: 
```
"update_check": false
```
修改完结果如下：然后重启就OK了  
![image-20221122115826512](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20221122115826512.png)
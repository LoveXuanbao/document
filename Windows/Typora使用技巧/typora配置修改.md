# 一、更改编辑器的宽度

## 1、设置 源码编辑器 的宽度

`Typora` 安装目录，默认为 `C:\Program Files\Typora\resources\app\style\` 下，找到 `base-control.css` 文件 ，打开后搜索 `#typora-source` ，找到 `max-width` （或者直接搜索 `max-width` ） ，将其值改为 1200 ，如图所示：

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20200425183504861.png)

### 1.1、验证

重启 Typora 后，进入 源码模式：

测试一下，看到 灰底部分 的宽度确实是改成 1200px ，比之前宽了好多 。

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20190517142257657.png)



## 2、设置 编辑器（主题） 的宽度

普通编辑器的配置文件 并不在 `Typora` 的安装目录， 根据下面图所示（ 文件 --> 编好设置 --> 打开主题文件夹）：

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20190519220646677.png)


即是在 `C:\Users\Administrator\AppData\Roaming\Typora\themes` 目录下，

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20190517141356910.png)

如上图所示： 主题有 `github.css`、`newsprint.css`、`night.css`、`pixyll.css`、`whitey.css` 。
选择使用主题的`css`文件，搜索 `#write` ，修改其属性 `max-width` 的值改为 `1060px` 。所图所示：

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/2019051714164127.png)



# 二、修改代码的颜色，不是代码块的

如下图所示，修改代码的颜色，不是代码块。

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20190519220646677.png)

选择当前使用的主题：

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20200421095607799.png)

我使用主题是 `github`，所以打开 `github.css`，在 230行 的`code`标签中，添加 `color: red`; ，所下图所示：

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20200421095747247.png)

然后，重启编辑器， 查看效果，如下图所示：

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20200421095359618-1587811322112.png)

# 三、TOC目录字体大小调整

可以通过所选主题的`主题名.css`文件中的`.md-toc`类修改文字大小，例如`vue.css`修改

```css
.md-toc {
    margin-top: 20px;
    padding-bottom: 20px;
  font-size: 150%;   /* 目录文字大小 */
}
```

# 四、插入图片靠左

## Typora插入图片靠左靠右居中设置

#### 方式一、在Typora图片标签处添加`style=" float:left"`(居左)

> Typora是支持html语言的，可以使用img标签插入图片，缺点是此方式导出的pdf图片显示不正确

```
<img src="D:\Personal files\Typora\typora-pic\image-20200506153717232.png" alt="image-20200506153717232.png" style=" float:left" />
```



#### 方式二、修改主题样式.css文件（居左）

> 该方式到处的pdf图片可以正常显示。

设置路径：文件-->偏好设置-->外观-->打开主题文件夹，

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20211014142320.png)

选择使用主题的css文件打开

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20211014142352.png)

在最后插入如下css代码

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20211014142233.png)

```
p .md-image:only-child{
    width: auto;
    text-align: left;
}
```

# 五、添加页内跳转

两种方法

1. 利用`markdown`语法实现
2. 利用`html`语法实现

## 利用markdown语法实现

```shell
[hello](#world)

### world
```

把这两句内容复制到`typora`，记住，一定要<font color=#FF0000>按住Ctrl并点击</font>才能实现效果，不按`Ctrl`系统会认为你是要移动焦点。

## 利用html语法实现

```shell
<a href="#233">hello</a>

<span name="233">233</span>
```

把这段话复制到`typora`，记住，一点要<font color=#FF0000>按住Ctrl并点击</font>才能实现效果，不按`ctrl`系统会认为你是要移动焦点，目前只有`name`可以达到效果，`id`不能用，不明白为什么
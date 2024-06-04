## 一、不装vimdesktop集成everythin
在TC的主目录下找到usercmd.ini（如果没有的话，手工新建一个），在其中输入下面的设置代码：
```shell
[em_Everything]  
cmd=C:\Program Files\Everything\Everything.exe  
param="-search "%P "" 
```

在上面的设置代码中，第一个是Everything的可执行文件路径，第二个是参数。这个命令的目的是在当前目录(%P)下进行搜索。如果希望是全局搜索，则可以将param中后面的"%P “去掉。这里需要注意的是，在”%P "中包含有空格，这样做的好处是在搜索的时候将会包含有子目录。如果只是希望在当前目录下搜索而不需要包含子目录，可以将此空格去掉。

接下TC中配置->其他，进行快捷键设置。这里使用Windows资源管理器中常用的Ctrl+F作为搜索的快捷键。在自定义快捷键的地方选中Ctrl和F后，在命令后面的放大镜弹出窗口中可以找到前面设置好的em_Everything命令，并按后面的确定按钮使其生效.
 或者直接在wincmd.ini文件中配置快捷键即可.
通过这样的设置后，按下Ctrl+F，即可以通过Everything在当前目录下搜索文件了。

## 二、装vimdesktop集成everything
在vimdesktop的\conf目录下，找到vimd.ini文件在最后一行添加如下配置
```shell
<w-f>=Run|C:\Program Files (x86)\Everything\Everything.exe			//everything的执行文件路径
```
windows 官网下载方法：https://windows10.pro/microsoft-com-software-download-windows10-iso/



系统安装完成后

以管理员运行命令提示符

```
slmgr.vbs /upk  (此时弹出宽口显示“已成功卸载了产品密钥”)

slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX （弹出窗口提示：“成功的安装了产品密钥”）

slmgr /skms zh.us.to （弹出窗口提示：“密钥管理服务计算机名成功的设置为 zh.us.to”）

slmgr /ato （弹出窗口提示：“成功的激活了产品”）
```
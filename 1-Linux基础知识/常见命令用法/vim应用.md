### 删除 # 开头的注释行

```bash
:g/^#/d
```



### 文件中所有包含  `docker.gridsum`  的行的内容加上双引号

```bash
%s/\(docker.gridsum.*\)$/"\1"/g
```

例子：

![image-20230825113746636](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825113746636.png)



### 删除每行第一个空格后的所有字符

```bash
:%s/\s.*//g
```

### vim粘贴带#保留原格式

```bash
:set paste
```

### 删除匹配行

```bash
:g/^GDP.*/d
```

## 反向引用

- `\( xxx\)`小括号包围要引用的对象
- \1  引用第一个被包围的对象

```bash
%s#\(repository.test.com.*\).6#\1.0#g
```



### 删除匹配行后面得所有行

```
sed '/status/,$d'
```


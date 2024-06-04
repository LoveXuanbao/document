### 前端Dockerfile

```bash
FROM docker.gridsumdissector.com/frontend/nginx-base:1.21.6
COPY --chown=nginx dist /usr/html/
COPY --chown=nginx update.sh /usr/html/update.sh
RUN chmod +x /usr/html/update.sh
```

### updata.sh

```bash
#!/bin/sh
if [ -n "${BASE_URL}" ];then
echo " base url: "${BASE_URL}
sed -i "s@###baseUrl###@${BASE_URL}@" /usr/html/js/config.*.js
fi
```

### Java Dockerfile示例

```bash
# 基于OpenJDK 8基础镜像
FROM docker.gridsumdissector.com/library/openjdk:8u282-jre-slim
# 修改此处为实际输出⽬录
ARG TARGET=target/
# 修改此处实际⽣成的jar包名称
ARG MYAPPNAME=myapp.jar
# 切换⽤户
USER root
# 设置⼯作⽬录
WORKDIR /workdir
# jar包⽂件路径为:/workdir/app.jar
COPY $TARGET/$MYAPPNAME app.jar
Open Created 5 days ago by 张秋⽣-平台产品交付部-北京
# 拷⻉启动脚本
COPY docker-entrypoint.sh /usr/local/bin/
# 运⾏shell命令
# 修改时间及时区并为启动脚本添加可执⾏权限
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
&& echo 'Asia/Shanghai' >/etc/timezone \
&& chmod +x /usr/local/bin/docker-entrypoint.sh
# 暴露服务端⼝
EXPOSE 8080
# 启动⼊⼝点
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
```

### 多阶段Dockerfile

```bash
# python 3.8
FROM law-harbor.internal.gridsumdissector.com/infra/python-builder:3.8-20220528 AS builder
ARG PYPI_MIRROR=https://mirrors.aliyun.com/pypi/simple/
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN python -m venv /opt/venv \
&& pip install --upgrade pip \
&& pip install -r requirements.txt
# python 3.8
FROM law-harbor.internal.gridsumdissector.com/infra/python38-bullseye-slim:1.0.0-20220729 AS runner
COPY --from=builder --chown=worker /opt/venv /opt/venv
# 拷⻉代码，最好只拷⻉代码
COPY --chown=worker . /workdir
EXPOSE 8080
# 启动脚本/app/entrypoint.sh需要业务根据框架实现
ENTRYPOINT [ "sh","/workdir/entrypoint.sh" ]
```




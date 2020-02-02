# CloudNativeJS

> Cloud Native Development with Node.js, Docker, and Kubernetes

## Parti-I express & docker

- `express CloudNativeJS`

- `wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/Dockerfile`

- `wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/.dockerignore`

- `docker build -t nodeserver -f Dockfile .`
  - 更新 Debian 好慢啊

```bash
# szy0syz @ Jerrys-iMac in ~/git/CloudNativeJS on git:master x [16:10:54] C:130
$ docker run -i -p 3000:3000 -t nodeserver

> cloud-native-js@1.0.0 start /app
> node ./bin/www

GET / 200 138.422 ms - 170
GET /stylesheets/style.css 200 2.532 ms - 111
GET /favicon.ico 404 3.845 ms - 160
```

- `wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/Dockerfile-tools`
- `wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/run-debug`

```shell
# 修改配置
if [ "$SUPPORT_NPM_INSPECT" == "1" ]; then
  node --inspect=0.0.0.0 ./bin/www
else
  node --debug=0.0.0.0 ./bin/www
fi
```

- `wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/run-dev`

```shell
#!/bin/bash
# Simple shell script to run application in dev mode
PATH=node_modules/.bin:$PATH nodemon ./bin/www
```

### 创建tools镜像

> 啥作用呢？热更新，根据代码实时重构镜像。

- `docker build -t nodeserver-tools -f Dockerfile-tools .`
- `docker run -i -v "$PWD"/package.json:/tmp/package.json -v "$PWD"/node_modules_linux:/tmp/node_modules -w /tmp -t node:10 npm install`
- `docker run -i -p 3000:3000 -v "$PWD"/:/app -v "$PWD"/node_modules_linux:/app/node_modules -t nodeserver-tools /bin/run-dev`
  - 这样就开启了一个实时热更新的开发镜像

```bash
# szy0syz @ Jerrys-iMac in ~/git/CloudNativeJS on git:master x [16:23:50] C:1
$ docker run -i -p 3000:3000 -v "$PWD"/:/app -v "$PWD"/node_modules_linux:/app/node_modules -t nodeserver-tools /bin/run-dev
[nodemon] 2.0.2
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node ./bin/www`
GET / 304 553.277 ms - -
GET /stylesheets/style.css 304 2.781 ms - -
GET / 200 11.224 ms - 174
GET /stylesheets/style.css 304 2.358 ms - -
```

### 关于debugger

- `docker run -i --expose 9229 -p 9229:9229 -p 3000:3000 -v "$PWD"/:/app -v "$PWD"/node_modules_linux:/app/node_modules -t nodeserver-tools /bin/run-debug`

![i101](http://cdn.jerryshi.com/1580633081211.jpg)
![2](http://cdn.jerryshi.com/1580633314083.jpg)
![3](http://cdn.jerryshi.com/1580633317629.jpg)

### 构建生成环境的镜像

- `wget https://raw.githubusercontent.com/CloudNativeJS/docker/master/Dockerfile-run`
- `docker build -t nodeserver-run -f Dockerfile-run .`
- `docker run -i -p 3000:3000 -t nodeserver-run`
- `docker tag nodeserver-run nodeserver:1.0.0`
- 接下来登录 DockerHub 或者 阿里云 上传镜像

```bash
docker login --username=185505508@qq.com registry.cn-hangzhou.aliyuncs.com
docker tag 9022c9fcffba registry.cn-hangzhou.aliyuncs.com/szy0syz/nodeserver:1.0.0
docker push registry.cn-hangzhou.aliyuncs.com/szy0syz/nodeserver:1.0.0
```

## Kubernetes & Helm

![k8s](http://cdn.jerryshi.com/WX20200202-170744.png?imageView2/3/w/800)

> MaxOS 安装k8s相对简单

![ks8install](http://cdn.jerryshi.com/20200202171452.png)

### 安装 Helm

> brew install kubernetes-helm

- `helm repo`
- `helm repo list`
- <https://hub.helm.sh/>
- <https://github.com/CloudNativeJS/helm>
- 拷贝repo中chart文件夹到项目根目录

### 启动

- `helm install nodeserver chart/nodeserver`

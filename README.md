# 原地址
- https://github.com/truthhun/DocHub

--

# 说明
最近又要部署文库，把从零部署，把最基础的过程记录一下，方便后面使用，方便定制

# 修改
1.后台直接添加用户

# 编译
下载项目下来，进行二次开发，修改工程go文件，静态文件，配置app.conf，什么的
	app.conf 没域名就关掉
		enablexsrf = false

！注意，工程中 github.com/TruthHun/DocHub/ 的应用要改成 github.com/xxx/DocHub/
xxx 就是你自己本地的目录，也可以是你github端的用户名（那样你要上传代码后编译才是拉取云端代码）

使用liteide编译得到对应系统的可执行文件
选择system得到：Dochub.exe 方便本地调试逻辑，但是文档存储转化需要依赖软件的本地没装没法调
选择cross-linux64得到：Dochub
开始build可能有问题，我按照提示目录下运行下面命令就能编译了
	`go mod init`
	`go mod tidy`
	`go mod vendor`

# 打包
DocHub_linux_amd64.tar.gz，我用7z先打成tar，然后gz压缩得到包，包内目录结构
	DocHub_linux_amd64.tar.gz
	  conf
	    app.conf 我这里指向安装虚拟机的主机上的mysql，创建一个空数据库dochub
	  dictionary
	  static
	  views
	  DocHub

# 运行
我的环境是centos，先安装好docker
下载运行环境和需要的软件
	`docker pull truthhun/dochub:env`
minio千万不下用最新的，最新的好像不能单机用
	`docker pull minio/minio:RELEASE.2022-01-25T19-56-04Z`


上传 DocHub_linux_amd64.tar.gz 和 Dockerfile - jht(改名字为：Dockerfile)
构建镜像，DocHub_linux_amd64.tar.gz 和 改名字为：Dockerfile 放在一个目录
	`docker build -t app/dochub:v1.0 .`

运行minio，9001和9002分别是私有和公开的端口，"/home/docker/app_minio/" 一般映射到容器外
```
docker run -d --name app_minio \
-p 9002:9002 -p 9003:9003 \
-e "MINIO_ROOT_USER=admin" \
-e "MINIO_ROOT_PASSWORD=app123456" \
-v /home/docker/app_minio/data:/data \
-v /home/docker/app_minio/config:/root/.minio \
minio/minio:RELEASE.2022-01-25T19-56-04Z server /data \
--console-address ":9002" --address ":9003"
```

进入minio做一些配置
虚拟机ip：端口，要想其他机器访问，把对应端口映射到宿主机端口即可
用户名密码 admin app123456 注意这个用户名密码好像没地方改
移植时候，"/home/docker/app_minio/" 也是用这个用户名密码
	http://192.168.174.132:9002/
	http://192.168.174.132:9003/
找到Buckets菜单，创建两个Buckets
	dochub-public 设置public
	dochub-private 设置private

运行程序
`docker run -d -p 9001:9001 --name app_dochub-v1.0 app/dochub:v1.0`

访问文库后台
	http://192.168.174.132:9001/admin
		admin admin 芝麻开门
	找到 云存储配置，选择minio项目，从上到下
		admin
		app123456
		192.168.174.132:9003
		dochub-public
		http://192.168.174.132:9003/dochub-public
		dochub-private
		http://192.168.174.132:9003/dochub-private
	提交更改，正常没问题这里会提示成功




卸载旧版本
~~~
sudo yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine

~~~
执行即可
~~~

sudo yum install -y yum-utils \ 
device-mapper-persistent-data \ 
lvm2

sudo yum-config-manager \ 
--add-repo \ 
https://download.docker.com/linux/centos/docker-ce.repo 

sudo yum install docker-ce docker-ce-cli containerd.io

#启动docker
sudo systemctl start docker
#查看docker服务状态 running 就是启动成功
sudo systemctl status docker

sudo systemctl enable docker
~~~

改加速镜像

~~~
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://3hrms5p2.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

~~~

如果aliyun的配置了导致出现网络问题了

~~~
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
	    "https://3hrms5p2.mirror.aliyuncs.com",
        "https://do.nark.eu.org",
        "https://dc.j8.work",
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

~~~

安装docker compose
~~~
yum install -y epel-release
yum install -y docker-compose
~~~

升级docker-compose版本
https://blog.csdn.net/dbfedbf/article/details/132741390
~~~
#docker-compose-linux-x86_64
mv docker-compose-linux-x86_64 docker-compose
chmod +x docker-compose
whereis docker-compose  # /usr/bin/docker-compose
 
mv /usr/bin/docker-compose /usr/bin/docker-compose.bak    # 备份一下原有的脚本
mv ./docker-compose   /usr/bin/         # 将当前目录的docker-compose  拷贝到  /usr/bin 目录下
 
docker-compose --version     #Docker Compose version v2.17.3
~~~

~~~
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
		"https://3hrms5p2.mirror.aliyuncs.com",       
		"https://docker.1ms.run",
        "https://docker.m.daocloud.io",
        "https://docker.1panel.top",
        "https://hub.rat.dev",
        "https://docker.anyhub.us.kg",
        "https://dockerhub.icu",
        "https://docker.awsl9527.cn",
        "https://registry.docker-cn.com",
        "https://nrbewqda.mirror.aliyuncs.com",
        "https://dmmxhzvq.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
~~~


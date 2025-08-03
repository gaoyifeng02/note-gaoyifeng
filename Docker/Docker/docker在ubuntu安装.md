
修改root密码
~~~
sudo passwd root
~~~

获取root权限
~~~
su root
~~~

查看防火墙状态
~~~~
ufw status
~~~~

关闭防火墙
~~~
ufw disable
~~~

安装docker
~~~
apt-get install docker.io docker-compose
~~~

安装ssh
~~~
apt install openssh-server
~~~



~~~
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
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
systemctl daemon-reload
systemctl restart docker
~~~

~~~
docker restart nacos-alone
docker restart mysql5.7-alone
docker restart mysql8-alone
docker restart redis-alone
docker restart nginx-alone
docker restart rabbitmq-alone
docker restart elasticsearch-7-alone
docker restart zookeeper-alone
~~~

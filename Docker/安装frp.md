
https://blog.csdn.net/y_wu794/article/details/150637146

~~~
docker run -d --name frps --restart=unless-stopped -p 7000:7000 -v /opt/frp/frps.toml:/root/frp/frp_0.51.3_linux_386/frps.toml snowdreamtech/frps:0.51.3
~~~


~~~
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/root/frp/frp_0.51.3_linux_386/frps -c /root/frp/frp_0.51.3_linux_386/frps.toml

[Install]
WantedBy=multi-user.target
~~~

/root/frp/frp_0.51.3_linux_386

sudo systemctl daemon-reload
sudo systemctl enable frps
sudo systemctl start frps
sudo systemctl status frps



启动
~~~
 docker-compose -f docker-compose.yml up -d
~~~











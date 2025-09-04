

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




启动
~~~
 docker-compose -f docker-compose.yml up -d
~~~











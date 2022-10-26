
#### 一、基本环境
mac m1虚拟机Ubuntu 20.04.4 服务器版

服务端需要用到一些端口 所以直接关闭防火墙

```shell
sudo ufw status #查看防火墙状态 inactive是关闭，active是开启
sudo ufw disable #关闭防火墙
sudo ufw enable  #开启防火墙
```

#### 二、依赖

##### 2.1、基础依赖
```shell
sudo apt-get update

#用于安装依赖
sudo apt-get install aptitude

#libssl1.0.1-dev 找不到 解决:可以找高版本的 一些依赖的api需要手动替换
#libglib2.3.4-dev 找不到 解决:使用libglib3.0-cil-dev
sudo aptitude install libmicrohttpd-dev libjansson-dev libnice-dev \
    libssl1.0.1-dev libsrtp-dev libsofia-sip-ua-dev libglib2.3.4-dev \
    libopus-dev libogg-dev libcurl4-openssl-dev pkg-config gengetopt \
    libtool automake

sudo apt install cmake g++
sudo aptitude install libconfig-dev
sudo aptitude install libssl-dev
sudo aptitude install doxygen graphviz

# ffmpeg库 支持--enable-post-processing
sudo aptitude install libavcodec-dev libavformat-dev libswscale-dev libavutil-dev

```

##### 2.2、WebSocket

```shell
git clone https://github.com/warmcat/libwebsockets.git
cd libwebsockets
git branch -a 查看选择最新的稳定版本，目前的是remotes/origin/v3.2-stable
git checkout v3.2-stable 切换到最新稳定版本
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_C_FLAGS="-fpic" ..
make && sudo make install
```

##### 2.3、libsrtp

```shell
wget https://github.com/cisco/libsrtp/archive/v2.2.0.tar.gz
tar xfv v2.2.0.tar.gz
cd libsrtp-2.2.0
./configure --prefix=/usr --enable-openssl
make shared_library && sudo make install
```

##### 2.4、libusrsctp

```shell
# 编译报Taking address of packed member 'xx' of class or structure '' may result in an unaligned pointer value。
#原因是结构体添加了__attribute__((packed))的宏 统一去除后解决
git clone https://github.com/Kurento/libusrsctp.git
cd libusrsctp
./bootstrap
./configure
make
sudo make install
```

##### 2.5、libmicrohttpd

```shell
wget https://ftp.gnu.org/gnu/libmicrohttpd/libmicrohttpd-0.9.71.tar.gz
tar zxf libmicrohttpd-0.9.71.tar.gz
cd libmicrohttpd-0.9.71/
./configure
make
sudo make install
```

##### 2.6、Janus
```shell
git clone https://github.com/meetecho/janus-gateway.git
git tag 查看当前的 tag
git  checkout v1.1.0
sh autogen.sh
#janus安装在/opt/janus目录
./configure --prefix=/opt/janus --enable-websockets --enable-post-processing --enable-docs --enable-rest --enable-data-channels
make
sudo make install



#./configure后的配置打印
ompiler:                  gcc
libsrtp version:           2.x
SSL/crypto library:        OpenSSL
DTLS set-timeout:          not available
Mutex implementation:      GMutex (native futex on Linux)
DataChannels support:      yes
Recordings post-processor: yes
TURN REST API client:      yes
Doxygen documentation:     yes
Transports:
    REST (HTTP/HTTPS):     yes
    WebSockets:            yes
    RabbitMQ:              no
    MQTT:                  no
    Unix Sockets:          yes
    Nanomsg:               no
Plugins:
    Echo Test:             yes
    Streaming:             yes
    Video Call:            yes
    SIP Gateway:           yes
    NoSIP (RTP Bridge):    yes
    Audio Bridge:          yes
    Video Room:            yes
    Voice Mail:            yes
    Record&Play:           yes
    Text Room:             yes
    Lua Interpreter:       no
    Duktape Interpreter:   no
Event handlers:
    Sample event handler:  yes
    WebSocket ev. handler: yes
    RabbitMQ event handler:no
    MQTT event handler:    no
    Nanomsg event handler: no
    GELF event handler:    yes
External loggers:
    JSON file logger:      no
JavaScript modules:        no
```

##### 2.7、nginx

```shell
#下载nginx 1.15.8版本
wget http://nginx.org/download/nginx-1.15.8.tar.gz
tar xvzf nginx-1.15.8.tar.gz
cd nginx-1.15.8/


# 配置，一定要支持https
./configure --with-http_ssl_module 

# 编译
make

#安装
sudo make install
```

##### 2.8、coturn

```shell
sudo apt-get install libssl-dev
sudo apt-get install libevent-dev

#git clone https://github.com/coturn/coturn 
#cd coturn
# 提供另一种安装方式turnserver是coturn的升级版本
wget http://coturn.net/turnserver/v4.5.0.7/turnserver-4.5.0.7.tar.gz
tar xfz turnserver-4.5.0.7.tar.gz
cd turnserver-4.5.0.7

./configure 
make 
sudo make install
```

##### 2.9、安装sturn

```shell
sudo apt install openssl libboost-dev
wget http://www.stunprotocol.org/stunserver-1.2.7.tgz   
tar -zxvf stunserver-1.2.7.tgz  
cd stunserver-1.2.7
sudo make

#由于没有make install命令 所以将当前路径直接配置在/etc/profile中
#PATH=$PATH:/home/ms/rtc/stunserver/:.
```



#### 3、配置

##### 3.1、生成证书

```shell
#安装在/User/用户名/cert目录下 随后配置需要用到 目录位置随意
mkdir -p ~/cert
cd ~/cert
# CA私钥
openssl genrsa -out key.pem 2048
# 自签名证书
openssl req -new -x509 -key key.pem -out cert.pem -days 1095
```

##### 3.2、nginx配置

```shell
vim /usr/local/nginx/conf/nginx.conf
#直接增加一个
# HTTPS server
    #
    server {
        listen       443 ssl;
        server_name  localhost;
        # 配置步骤3.1相应的key
        ssl_certificate      /home/ubuntu/cert/cert.pem;
        ssl_certificate_key  /home/ubuntu/cert/key.pem;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        # 指向janus demo所在目录
        location / {
            root   /opt/janus/share/janus/demos;
            index  index.html index.htm;
        }
    }
```

##### 3.2、janus配置

先配置video room 需要配置的文件

```shell
# 进到对应的目录
cd /opt/janus/etc/janus
# 拷贝文件
sudo cp janus.jcfg.sample janus.jcfg
sudo cp janus.transport.http.jcfg.sample janus.transport.http.jcfg
sudo cp janus.transport.websockets.jcfg.sample janus.transport.websockets.jcfg
sudo cp janus.plugin.videoroom.jcfg.sample janus.plugin.videoroom.jcfg
sudo cp janus.transport.pfunix.jcfg.sample janus.transport.pfunix.jcfg
sudo cp janus.plugin.streaming.jcfg.sample janus.plugin.streaming.jcfg
sudo cp janus.plugin.recordplay.jcfg.sample janus.plugin.recordplay.jcfg
sudo cp janus.plugin.voicemail.jcfg.sample janus.plugin.voicemail.jcfg
sudo cp janus.plugin.sip.jcfg.sample janus.plugin.sip.jcfg
sudo cp janus.plugin.nosip.jcfg.sample janus.plugin.nosip.jcfg
sudo cp janus.plugin.textroom.jcfg.sample  janus.plugin.textroom.jcfg
sudo cp janus.plugin.echotest.jcfg.sample janus.plugin.echotest.jcfg
```

配置janus.jcfg

```shell
# stun_server服务器地址 都安装在本地测试 所以使用127.0.0.1
stun_server = "127.0.0.1"
        stun_port = 3478
        nice_debug = false

#turn_server服务器地址
# credentials to authenticate...
        turn_server = "127.0.0.1"
        turn_port = 3478
        turn_type = "udp"
        turn_user = "lqf"
        turn_pwd = "123456"
```

配置janus.transport.http.jcfg

```shell
general: {
        #events = true                                  # Whether to notify event handlers about transport events (default=true)
        json = "indented"                               # Whether the JSON messages should be indented (default),
                                                                        # plain (no indentation) or compact (no indentation and no spaces)
        base_path = "/janus"                    # Base path to bind to in the web server (plain HTTP only)
        threads = "unlimited"                   # unlimited=thread per connection, number=thread pool
        http = true                                             # Whether to enable the plain HTTP interface
        port = 8088                                             # Web server HTTP port
        #interface = "eth0"                             # Whether we should bind this server to a specific interface only
        #ip = "192.168.0.1"                             # Whether we should bind this server to a specific IP address (v4 or v6) only
        #开启https
        https = true                                    # Whether to enable HTTPS (default=false)
        secure_port = 8089                              # Web server HTTPS port, if enabled
        #secure_interface = "eth0"              # Whether we should bind this server to a specific interface only
        #secure_ip = "192.168.0.1"              # Whether we should bind this server to a specific IP address (v4 or v6) only
        #acl = "127.,192.168.0."                # Only allow requests coming from this comma separated list of addresses
}

certificates: {
				#主要配置秘钥路径
        cert_pem = "/home/ubuntu/cert/cert.pem"
        cert_key = "/home/ubuntu/cert/key.pem"
        #cert_pwd = "secretpassphrase"
        #ciphers = "PFS:-VERS-TLS1.0:-VERS-TLS1.1:-3DES-CBC:-ARCFOUR-128"
}
```

配置janus.transport.websockets.jcfg

```shell
general: {
        #events = true                                  # Whether to notify event handlers about transport events (default=true)
        json = "indented"                               # Whether the JSON messages should be indented (default),
                                                                        # plain (no indentation) or compact (no indentation and no spaces)
        #pingpong_trigger = 30                  # After how many seconds of idle, a PING should be sent
        #pingpong_timeout = 10                  # After how many seconds of not getting a PONG, a timeout should be detected

        ws = true                                               # Whether to enable the WebSockets API
        ws_port = 8188                                  # WebSockets server port
        #ws_interface = "eth0"                  # Whether we should bind this server to a specific interface only
        #ws_ip = "192.168.0.1"                  # Whether we should bind this server to a specific IP address only
        #开启wss
        wss = true                                              # Whether to enable secure WebSockets
        wss_port = 8989                         # WebSockets server secure port, if enabled
        #wss_interface = "eth0"                 # Whether we should bind this server to a specific interface only
        #wss_ip = "192.168.0.1"                 # Whether we should bind this server to a specific IP address only
        #ws_logging = "err,warn"                # libwebsockets debugging level as a comma separated list of things
                                                                        # to debug, supported values: err, warn, notice, info, debug, parser,
                                                                        # header, ext, client, latency, user, count (plus 'none' and 'all')
        #ws_acl = "127.,192.168.0."             # Only allow requests coming from this comma separated list of addresses
}

certificates: {
				#主要配置秘钥路径
        cert_pem = "/home/ubuntu/cert/cert.pem"
        cert_key = "/home/ubuntu/cert/key.pem"
        #cert_pwd = "secretpassphrase"
}
```



##### 4、运行服务器:

1、启动stunserver
stunserver
2、启动turnserver
sudo nohup turnserver -L 0.0.0.0 --min-port 30000 --max-port 60000  -a -u ms:123456 -v -f -r nort.gov &
3、启动janus
/opt/janus/bin/janus --debug-level=5 --log-file=$HOME/janus-log
4、启动nginx
sudo /usr/local/nginx/sbin/nginx

运行网页:
https://ip(服务器ip地址)/videoroomtest.html

进入首页后，找到 videoRoom，Start



##### 参考:

https://zhuanlan.zhihu.com/p/205056453
stunserver: https://www.jianshu.com/p/81e92402bb7f

Janus 官网：https://janus.conf.meetecho.com/index.html 参考文档：https://github.com/meetecho/janus-gateway
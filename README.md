# jerry_sync

# 1. Install python3.7+ first
###########################
#!/bin/bash
yum -y install epel-release
sleep 1
yum -y install zlib-devel bzip2-devel openssl-devel openssl-static ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel lzma gcc

sleep 1

yum -y groupinstall "Development tools"

cd /usr/local/src/ && wget -c https://www.python.org/ftp/python/3.9.4/Python-3.9.4.tgz

if [ $? -eq 0 ];then
tar xvf Python-3.9.4.tgz
else
exit 1
fi

mv Python-3.9.4 /usr/local/python-3.9 && cd /usr/local/python-3.9/

./configure --prefix=/usr/local/sbin/python-3.9 && make && make install

if [ $? -eq 0 ];then
rm -rf /usr/bin/python
ln -sv /usr/local/sbin/python-3.9/bin/python3 /usr/bin/python
else
exit 1
fi

sed -i '1s#usr/bin/python#usr/bin/python2.7#' /usr/bin/yum
sed -i '1s#usr/bin/python#usr/bin/python2.7#' /usr/libexec/urlgrabber-ext-down


rm -rf /usr/bin/pip
ln -s /usr/local/sbin/python-3.9/bin/pip3 /usr/bin/pip

# 2. Install docker and docker-compose
#!/bin/bash
if [ `whoami` != root ]
then
echo "Please login as root to continue :)"
exit 1
fi

if [ ! -d /home/tools/ ];then
mkdir -p /home/tools
else
rm -rf /home/tools && mkdir -p /home/tools
fi

#cd /home/tools && wget -c http://download.cashalo.com/schema/docker-18.09.6.tgz && wget -c http://download.cashalo.com/schema/docker.service
cd /home/tools && wget -c https://mirrors.aliyun.com/docker-ce/linux/static/stable/x86_64/docker-19.03.9.tgz && wget -c http://download.cashalo.com/schema/docker.service
if [ $? -eq 0 ];
then
tar zxvf /home/tools/docker-19.03.9.tgz
else
exit 1
fi
mv /home/tools/docker/* /usr/bin/
mkdir /etc/docker

#sudo tee /etc/docker/daemon.json <<-'EOF'
#{
#  "registry-mirrors": ["https://4a0dj5if.mirror.aliyuncs.com"]
#}
#EOF

cat >/usr/lib/systemd/system/docker.service<<EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service containerd.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP \$MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
EOF

sleep 1
systemctl daemon-reload && systemctl start docker && systemctl enable docker
#Install docker-compose
#sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
#Apply executable permissions to the binary
sudo chmod +x /usr/local/bin/docker-compose

# so easy when u etl by clickhouse 

# jerry_sync

# 1. Install python3.7+ first
###########################
yum -y install epel-release


yum -y install zlib-devel bzip2-devel openssl-devel openssl-static ncurses-devel sqlite-devel \
readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel lzma gcc


yum -y groupinstall "Development tools"

cd /usr/local/src/ && wget -c https://www.python.org/ftp/python/3.9.4/Python-3.9.4.tgz

tar xvf Python-3.9.4.tgz

mv Python-3.9.4 /usr/local/python-3.9 && cd /usr/local/python-3.9/

./configure --prefix=/usr/local/sbin/python-3.9 && make && make install


rm -rf /usr/bin/python

ln -sv /usr/local/sbin/python-3.9/bin/python3 /usr/bin/python


sed -i '1s#usr/bin/python#usr/bin/python2.7#' /usr/bin/yum
sed -i '1s#usr/bin/python#usr/bin/python2.7#' /usr/libexec/urlgrabber-ext-down


rm -rf /usr/bin/pip
ln -s /usr/local/sbin/python-3.9/bin/pip3 /usr/bin/pip



rm -rf /home/tools && mkdir -p /home/tools

cd /home/tools 

wget -c https://mirrors.aliyun.com/docker-ce/linux/static/stable/x86_64/docker-19.03.9.tgz  

mv /home/tools/docker/* /usr/bin/
mkdir /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://4a0dj5if.mirror.aliyuncs.com"]
}
EOF

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


systemctl daemon-reload && systemctl start docker && systemctl enable docker

sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# so easy when u etl by clickhouse 

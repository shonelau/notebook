# step1: install apr
```bash
wget http://mirrors.shu.edu.cn/apache//apr/apr-1.6.5.tar.gz
tar xvf apr-1.6.5.tar.gz
cd apr-1.6.5/
./configure
sudo make
sudo make install
```
#  step2: install apr-util
```bash
wget http://mirrors.shu.edu.cn/apache//apr/apr-util-1.6.1.tar.gz
tar xvf apr-util-1.6.1.tar.gz
cd apr-util-1.6.1/
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
sudo make
sudo make install
```
# step3: install dependencies
```bash
sudo apt install libpcre3 libpcre3-dev
sudo apt install openssl libssl-dev
sudo apt-get install zlib1g-dev
```
# step4: install apache
 ```bash
 wget http://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.38.tar.gz
 tar xvf httpd-2.4.38.tar.gz
 cd httpd-2.4.38/
./configure \
--with-apr-util=/usr/local/apr-util \
--prefix=/usr/local/httpd2.4.38 \
--sysconfdir=/etc/httpd2.4.38 \
--enable-so \
--enable-ssl \
--enable-cgi \
--enable-rewrite \
--with-zlib \
--with-pcre \
--with-mpm=prefork \
--enable-modules=most \
--enable-mpms-shared=all
sudo make
sudo make install
```
# step5 configure and run

##　configure

```bash
cd /etc/httpd2.4.38
sudo nano httpd.conf
```
uncommt proxy and proxy_uwsgi to enable uwsgi　　
LoadModule proxy_module modules/mod_proxy.so　　
LoadModule proxy_uwsgi_module modules/mod_proxy_uwsgi.so　　

## run
cd /usr/local/httpd2.4.38/bin
sudo ./apachectl start


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

## install uWsgi
```bash 
wget https://projects.unbit.it/downloads/uwsgi-2.0.18.tar.gz
tar xvf uwsgi-2.0.18.tar.gz 
cd  uwsgi-2.0.18/
make
```

## add fair1.conf

```bash
sudo nano /etc/httpd2.4.38/extra/fair1.conf
```

fair1.conf

```
ProxyRequests Off
ProxyPass /web uwsgi://127.0.0.1:9199/
```

httpd.conf

```
<IfModule proxy_module>
Include /etc/httpd2.4.38/extra/fair1.conf
</IfModule>
```

## edit a test application
```bash
cd uwsgi-2.0.18
nano web.py
```

web.py
```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"]
```

run application
webq.py 
``` python
#!/usr/bin/env python
"""
A minimal Quixote demo.  If you have the 'quixote' package in your Python
path, you can run it like this:

  $ python demo/mini_demo.py

The server listens on localhost:8080 by default.  Debug and error output
will be sent to the terminal.
"""

from quixote.publish import Publisher
from quixote.directory import Directory, export

class RootDirectory(Directory):

    @export(name='')
    def index(self):
        return '''<html>
                    <body>Welcome to the Quixote demo.  Here is a
                    <a href="hello">link</a>.
                    </body>
                  </html>
                '''

    @export
    def hello(self):
        return '<html><body>Hello world!</body></html>'


def create_publisher():
    #return Publisher(RootDirectory(),display_exceptions='plain')
    Publisher(RootDirectory(),display_exceptions='plain')

create_publisher()
wsgi_app = quixote.get_wsgi_app()
```
run uwsgi
```bash
#use simple function
./uwsgi --socket 127.0.0.1:9199 --plugin python2 --wsgi-file ./web.py
#use quixote
./uwsgi --socket 127.0.0.1:9199 --wsgi-file ./webq.py --callable wsgi_app --master --processes 4
```

#　问题

can not use 3031 as uwsgi 's default port。

# links
[Python uWSGI 安装配置](https://m.runoob.com/python3/python-uwsgi.html)

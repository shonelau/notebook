# install uWsgi
```bash 
wget https://projects.unbit.it/downloads/uwsgi-2.0.18.tar.gz
tar xvf uwsgi-2.0.18.tar.gz 
cd  uwsgi-2.0.18/
make
```

# add web.conf for apache

```bash
sudo nano /etc/httpd2.4.38/extra/web.conf
```

fair1.conf

```
ProxyRequests Off
ProxyPass /web uwsgi://127.0.0.1:9199/
```
# edit httpd.conf

httpd.conf

```
<IfModule proxy_module>
Include /etc/httpd2.4.38/extra/fair1.conf
</IfModule>
```

# add a simple test application
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

# add a quixote application with wsgi interface

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
# run applications
```bash
#use simple function
./uwsgi --socket 127.0.0.1:9199 --wsgi-file ./web.py
#use quixote
./uwsgi --socket 127.0.0.1:9199 --wsgi-file ./webq.py --callable wsgi_app --master --processes 4
```

# 问题
can not use 3031 as uwsgi 's default port。

# links
[Python uWSGI 安装配置](https://m.runoob.com/python3/python-uwsgi.html)
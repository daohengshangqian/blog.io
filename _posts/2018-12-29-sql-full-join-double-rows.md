---
layout: post
title: Oracle 三表全外联结实践
date: 2018-12-29
categories: blog
tags: [SQL]
description: Oracle 三表全外联结实践
---
```sql

#!/usr/bin/env python
#coding:utf-8

from wsgiref.simple_server import make_server

def DaoServer(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return '<h1>Hello,dao!</h1>'

if __name__ == '__main__':
    httpd = make_server('', 8000, DaoServer)
    print ("Serving HTTP on port 8000...")
    httpd.serve_forever()


```

### 错误栈如下

```sql

127.0.0.1 - - [20/Dec/2018 07:18:51] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:18:51] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 53872)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:18:51] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:18:51] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 53873)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:18:51] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:18:51] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 53874)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:20:53] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:20:53] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54000)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:20:53] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:20:53] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54001)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:20:53] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:20:53] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54002)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:21:54] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:21:54] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54018)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:21:54] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:21:54] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54019)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:21:54] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:21:54] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54020)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:26:54] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:26:54] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54098)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:26:54] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:26:54] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54099)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------
127.0.0.1 - - [20/Dec/2018 07:26:54] "GET / HTTP/1.1" 200 0
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance
127.0.0.1 - - [20/Dec/2018 07:26:54] "GET / HTTP/1.1" 500 59
----------------------------------------
Exception happened during processing of request from ('127.0.0.1', 54100)
Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 266, in write
    "write() argument must be a bytes instance"
AssertionError: write() argument must be a bytes instance

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 141, in run
    self.handle_error()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 368, in handle_error
    self.finish_response()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 180, in finish_response
    self.write(data)
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 274, in write
    self.send_headers()
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 331, in send_headers
    if not self.origin_server or self.client_is_modern():
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 344, in client_is_modern
    return self.environ['SERVER_PROTOCOL'].upper() != 'HTTP/0.9'
TypeError: 'NoneType' object is not subscriptable

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\Program\Python\Python37\lib\socketserver.py", line 313, in _handle_request_noblock
    self.process_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 344, in process_request
    self.finish_request(request, client_address)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 357, in finish_request
    self.RequestHandlerClass(request, client_address, self)
  File "D:\Program\Python\Python37\lib\socketserver.py", line 717, in __init__
    self.handle()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 133, in handle
    handler.run(self.server.get_app())
  File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 144, in run
    self.close()
  File "D:\Program\Python\Python37\lib\wsgiref\simple_server.py", line 35, in close
    self.status.split(' ',1)[0], self.bytes_sent
AttributeError: 'NoneType' object has no attribute 'split'
----------------------------------------

```

## 解决方法

### 查看报错代码

#### run方法

```sql

File "D:\Program\Python\Python37\lib\wsgiref\handlers.py", line 138, in run
   def run(self, application):
        """Invoke the application"""
        # Note to self: don't move the close()!  Asynchronous servers shouldn't
        # call close() from finish_response(), so if you close() anywhere but
        # the double-error branch here, you'll break asynchronous servers by
        # prematurely closing.  Async servers must return from 'run()' without
        # closing if there might still be output to iterate over.
        try:
            self.setup_environ()
            self.result = application(self.environ, self.start_response)
            self.finish_response()   # 此处报错
        except:
            try:
                self.handle_error()
            except:
                # If we get an error handling an error, just give up already!
                self.close()
                raise   # ...and let the actual server figure it out.

```


#### finish_response 方法

```sql

   def finish_response(self):
        """Send any iterable data, then close self and the iterable

        Subclasses intended for use in asynchronous servers will
        want to redefine this method, such that it sets up callbacks
        in the event loop to iterate over the data, and to call
        'self.close()' once the response is finished.
        """
        try:
            if not self.result_is_file() or not self.sendfile():
                for data in self.result:
                    self.write(data)  # write 方法报错
                self.finish_content()
        finally:
            self.close()

```
#### write方法 

```sql


    def write(self, data):
        """'write()' callable as specified by PEP 3333"""
        ### 此处断言data 必须为byte类型
        assert type(data) is bytes, \
            "write() argument must be a bytes instance"

        if not self.status:
            raise AssertionError("write() before start_response()")

        elif not self.headers_sent:
            # Before the first output, send the stored headers
            self.bytes_sent = len(data)    # make sure we know content-length
            self.send_headers()
        else:
            self.bytes_sent += len(data)

        # XXX check Content-Length and truncate if too many bytes written?
        self._write(data)
        self._flush()


```


#### 打印data类型

```sql

  def finish_response(self):
        """Send any iterable data, then close self and the iterable

        Subclasses intended for use in asynchronous servers will
        want to redefine this method, such that it sets up callbacks
        in the event loop to iterate over the data, and to call
        'self.close()' once the response is finished.
        """
        try:
            if not self.result_is_file() or not self.sendfile():
                for data in self.result:
                    print('dao dao dao data type:'+str(type(data)))
                   
                    # self.write(data)
                #self.finish_content()
        finally:
            self.close()

```

结果为

```sql

127.0.0.1 - - [20/Dec/2018 08:29:07] "GET / HTTP/1.1" 200 0
dao dao dao data type:<class 'str'>
dao dao dao data type:<class 'str'>
dao dao dao data type:<class 'str'>


```


#### 修复方式，通过encode方法，转为byte类型


```sql

   def finish_response(self):
        """Send any iterable data, then close self and the iterable

        Subclasses intended for use in asynchronous servers will
        want to redefine this method, such that it sets up callbacks
        in the event loop to iterate over the data, and to call
        'self.close()' once the response is finished.
        """
        try:
            if not self.result_is_file() or not self.sendfile():
                for data in self.result:
                    print('dao dao dao data type:'+str(type(data)))
                    data=data.encode()
                    self.write(data)
                self.finish_content()
        finally:
            self.close()

```


#### 修复后可成功返回 

```sql

Hello,dao!

```
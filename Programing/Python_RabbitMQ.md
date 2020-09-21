# Python RabbitMQ

[TOC]

## 安装
```bash
sudo yum install rabbitmq-server
sudo pip install pika
```

## RabbitMQ
### 启动rabbitmq
```bash
sudo rabbitmq-server --detached &
```

### 查看状态
```bash
sudo rabbitmqctl status
```

### 停止 rabbitmq
```bash
sudo rabbitmqctl stop
```

### 


## hello world

send.py
```python
import sys  
import pika  
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))  
channel = connection.channel()  
channel.queue_declare(queue = 'hello')  
if len (sys.argv) < 2 :  
    print 'message is empty!'  
    sys.exit(0)  
message = sys.argv[1]  
channel.basic_publish(exchange = '', routing_key='hello', body = message)  
print "[x] sent: '" + message + "'\n"  
connection.close()
```


receive.py
```python
import pika  
connection = pika.BlockingConnection(pika.ConnectionParameters( 'localhost' ))  
channel = connection.channel()  
channel.queue_declare(queue = 'hello' )  
print '[*] Waiting for messages. To exit press CTRL+C'  
   
def callback(ch, method, properties, body):  
     print body

channel.basic_consume(callback, queue = 'hello' , no_ack = True )
channel.start_consuming()  
```

## 工作队列（work queue /task queue）

manager.py
```python
import pika  
import sys  
parameters = pika.ConnectionParameters(host = 'localhost' )  
connection = pika.BlockingConnection(parameters)  
channel = connection.channel()  
channel.queue_declare(queue = 'task_queue' , durable = True )  
message = ' ' .join(sys.argv[ 1 :]) or "Hello World!"  
channel.basic_publish(exchange = '',  
                       routing_key = 'task_queue' ,  
                       body = message,  
                       properties = pika.BasicProperties(  
                          delivery_mode = 2 , # make message persistent  
                       ))  
print " [x] Sent %r" % (message,)  
connection.close()  
```


worker.py
```python
import pika  
import time  
connection = pika.BlockingConnection(pika.ConnectionParameters(  
         host = 'localhost' ))  
channel = connection.channel()  
channel.queue_declare(queue = 'task_queue' , durable = True )  
print ' [*] Waiting for messages. To exit press CTRL+C'  
   
def callback(ch, method, properties, body):  
     print " [x] Received %r" % (body,)  
     time.sleep( body.count( '.' ) )  
     print " [x] Done"  
     ch.basic_ack(delivery_tag = method.delivery_tag)  
   
channel.basic_qos(prefetch_count = 1 )  
channel.basic_consume(callback,  
                       queue = 'task_queue' )  
channel.start_consuming()  
```

## 发布和订阅
emitlog.py
```python
import pika  
import sys  
   
connection = pika.BlockingConnection(pika.ConnectionParameters(  
         host = 'localhost' ))  
channel = connection.channel()  
   
channel.exchange_declare(exchange = 'logs' ,  
                          type = 'fanout' )  
   
message = ' ' .join(sys.argv[ 1 :]) or "info: Hello World!"  
channel.basic_publish(exchange = 'logs' ,  
                       routing_key = '',  
                       body = message)  
print " [x] Sent %r" % (message,)  
connection.close()  
```

```python
#!/usr/bin/env python  
import pika  
   
connection = pika.BlockingConnection(pika.ConnectionParameters(  
         host = 'localhost' ))  
channel = connection.channel()  
channel.exchange_declare(exchange = 'logs' ,  
                          type = 'fanout' )  
result = channel.queue_declare(exclusive = True )  
queue_name = result.method.queue  
channel.queue_bind(exchange = 'logs' ,  
                    queue = queue_name)  
print ' [*] Waiting for logs. To exit press CTRL+C'  
   
def callback(ch, method, properties, body):  
     print " [x] %r" % (body,)  
   
channel.basic_consume(callback,  
                       queue = queue_name,  
                       no_ack = True )  
channel.start_consuming()  
```

## 路由模式 （选择接收信息）
emitlog.py
```python
import pika  
import sys  
   
connection = pika.BlockingConnection(pika.ConnectionParameters(  
         host = 'localhost' ))  
channel = connection.channel()  
channel.exchange_declare(exchange = 'direct_logs' ,  
                          type = 'direct' )  
severity = sys.argv[ 1 ] if len (sys.argv) > 1 else 'info'  
message = ' ' .join(sys.argv[ 2 :]) or 'Hello World!'  
channel.basic_publish(exchange = 'direct_logs' ,  
                       routing_key = severity,  
                       body = message)  
print " [x] Sent %r:%r" % (severity, message)  
connection.close()  
```

recelog.py
```python
import pika  
import sys  
   
connection = pika.BlockingConnection(pika.ConnectionParameters(  
         host = 'localhost' ))  
channel = connection.channel()  
   
channel.exchange_declare(exchange = 'direct_logs' ,  
                          type = 'direct' )  
result = channel.queue_declare(exclusive = True )  
queue_name = result.method.queue  
severities = sys.argv[ 1 :]  
if not severities:  
     print >> sys.stderr, "Usage: %s [info] [warning] [error]" % \  
                          (sys.argv[ 0 ],)  
     sys.exit( 1 )  
for severity in severities:  
     channel.queue_bind(exchange = 'direct_logs' ,  
                        queue = queue_name,  
                        routing_key = severity)  
print ' [*] Waiting for logs. To exit press CTRL+C'  
def callback(ch, method, properties, body):  
     print " [x] %r:%r" % (method.routing_key, body,)  
channel.basic_consume(callback,  
                       queue = queue_name,  
                       no_ack = True )  
channel.start_consuming()  
```




```python

```



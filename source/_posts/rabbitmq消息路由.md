---
title: rabbitmq消息路由
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-26 22:15:04
password:
summary:
tags:
- rabbitmq
categories:
- rabbitmq
---

---
## direct交换器

------------

### 特点
- 投递的消息有一个或者多个确定的目标。
- 检查字符串是否相等，不允许使用模式匹配。
- 绑定相同路由键的队列都能收到该路由键对应的消息。
- 适用于RPC消息通信模式下的路由应答消息

示例代码：Direct交换器
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        exchange = rabbitpy.Exchange(channel, 'direct-example',
                                     exchange_type='direct')
        exchange.declare()
```

### 示例场景
RPC worker消费图片实现面部识别，将结果发回给消息发布方。


![](https://img2020.cnblogs.com/blog/1030146/202010/1030146-20201009211436091-1891495455.jpg)



- 客户端应用程序上传图像
- 应用程序处理请求，用唯一ID标识远程请求并创建一条消息
- 图像发布到交换器，消息属性的reply-to对应相应队列的名称, correlation-id对应请求ID
- 消息路由到队列，
- 消费者消费队列中的消息
- 结果以RPC请求形式返回前端。

注意：RabbitMQ最大帧大小为131072字节，，消息体超过这个大小，就需要在AMQP协议级别分块。预先分配占用7字节，因此，每个消息体帧只能承载131065字节图片数据。

示例代码：RPC Publisher
```python
import os
import rabbitpy
import time
from ch6 import utils

# Open the channel and connection
connection = rabbitpy.Connection()
channel = connection.channel()

exchange = rabbitpy.DirectExchange(channel, 'rpc-replies')
exchange.declare()

# Create the response queue that will automatically delete, is not durable and
# is exclusive to this publisher
queue_name = 'response-queue-%s' % os.getpid()
response_queue = rabbitpy.Queue(channel,
                                queue_name,
                                auto_delete=True,
                                durable=False,
                                exclusive=True)
# Declare the response queue
if response_queue.declare():
    print('Response queue declared')

# Bind the response queue
if response_queue.bind('rpc-replies', queue_name):
    print('Response queue bound')

# Iterate through the images to send RPC requests for
for img_id, filename in enumerate(utils.get_images()):

    print('Sending request for image #%s: %s' % (img_id, filename))

    # Create the message
    message = rabbitpy.Message(channel,
                               utils.read_image(filename),
                               {'content_type': utils.mime_type(filename),
                                'correlation_id': str(img_id),
                                'reply_to': queue_name},
                               opinionated=True)

    # Pubish the message
    message.publish('direct-rpc-requests', 'detect-faces')

    # Loop until there is a response message
    message = None
    while not message:
        time.sleep(0.5)
        message = response_queue.get()

    # Ack the response message
    message.ack()

    # Caculate how long it took from publish to response
    duration = (time.time() -
                time.mktime(message.properties['headers']['first_publish']))

    print('Facial detection RPC call for image %s total duration: %s' %
          (message.properties['correlation_id'], duration))

    # Display the image in the IPython notebook interface
    utils.display_image(message.body, message.properties['content_type'])

print('RPC requests processed')

# Close the channel and connection
channel.close()
connection.close()

```

示例代码：RPC worker
```python
import os
import rabbitpy
import time
from ch6 import detect
from ch6 import utils

# Open the connection and the channel
connection = rabbitpy.Connection()
channel = connection.channel()

# Create the worker queue
queue_name = 'rpc-worker-%s' % os.getpid()
queue = rabbitpy.Queue(channel, queue_name,
                       auto_delete=True,
                       durable=False,
                       exclusive=True)

# Declare the worker queue
if queue.declare():
    print('Worker queue declared')

# Bind the worker queue
if queue.bind('direct-rpc-requests', 'detect-faces'):
    print('Worker queue bound')

# Consume messages from RabbitMQ
for message in queue.consume_messages():

    # Display how long it took for the message to get here
    duration = time.time() - int(message.properties['timestamp'].strftime('%s'))
    print('Received RPC request published %.2f seconds ago' % duration)

    # Write out the message body to a temp file for facial detection process
    temp_file = utils.write_temp_file(message.body,
                                      message.properties['content_type'])

    # Detect faces
    result_file = detect.faces(temp_file)

    # Build response properties including the timestamp from the first publish
    properties = {'app_id': 'Chapter 6 Listing 2 Consumer',
                  'content_type': message.properties['content_type'],
                  'correlation_id': message.properties['correlation_id'],
                  'headers': {
                      'first_publish': message.properties['timestamp']}}

    # The result file could just be the original image if nothing detected
    body = utils.read_image(result_file)

    # Remove the temp file
    os.unlink(temp_file)

    # Remove the result file
    os.unlink(result_file)

    # Publish the response response
    response = rabbitpy.Message(channel, body, properties, opinionated=True)
    response.publish('rpc-replies', message.properties['reply_to'])

    # Acknowledge the delivery of the RPC request message
    message.ack()

```

## fanout交换器

### 特点

- 所有发往fanout交换器中的消息会被投递到所有该交换器绑定的队列中。
- 消息投递不需要检测路由键，性能更好

示例代码
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        exchange = rabbitpy.Exchange(channel,
                                    'fanout-rpc-requests',
                                     exchange_type='fanout')
        exchange.declare()
```
### 示例场景

![](https://img2020.cnblogs.com/blog/1030146/202010/1030146-20201009211416324-1179890043.jpg)




示例程序：Publisher
```python
import os
import rabbitpy
import time
from ch6 import utils

# Open the channel and connection
connection = rabbitpy.Connection()
channel = connection.channel()

# Create the response queue that will automatically delete, is not durable and
# is exclusive to this publisher
queue_name = 'response-queue-%s' % os.getpid()
response_queue = rabbitpy.Queue(channel,
                                queue_name,
                                auto_delete=True,
                                durable=False,
                                exclusive=True)
# Declare the response queue
if response_queue.declare():
    print('Response queue declared')

# Bind the response queue
if response_queue.bind('rpc-replies', queue_name):
    print('Response queue bound')

# Iterate through the images to send RPC requests for
for img_id, filename in enumerate(utils.get_images()):

    print 'Sending request for image #%s: %s' % (img_id, filename)

    # Create the message
    message = rabbitpy.Message(channel,
                               utils.read_image(filename),
                               {'content_type': utils.mime_type(filename),
                                'correlation_id': str(img_id),
                                'reply_to': queue_name},
                               opinionated=True)

    # Pubish the message
    message.publish('fanout-rpc-requests')

    # Loop until there is a response message
    message = None
    while not message:
        time.sleep(0.5)
        message = response_queue.get()

    # Ack the response message
    message.ack()

    # Caculate how long it took from publish to response
    duration = (time.time() -
                time.mktime(message.properties['headers']['first_publish']))

    print('Facial detection RPC call for image %s total duration: %s' %
          (message.properties['correlation_id'], duration))

    # Display the image in the IPython notebook interface
    utils.display_image(message.body, message.properties['content_type'])

print 'RPC requests processed'

# Close the channel and connection
channel.close()
connection.close()

```

示例程序：detect worker
```python
import os
import rabbitpy
import time
from ch6 import detect
from ch6 import utils

# Open the connection and the channel
connection = rabbitpy.Connection()
channel = connection.channel()

# Create the worker queue
queue_name = 'rpc-worker-%s' % os.getpid()
queue = rabbitpy.Queue(channel, queue_name,
                       auto_delete=True,
                       durable=False,
                       exclusive=True)

# Declare the worker queue
if queue.declare():
    print('Worker queue declared')

# Bind the worker queue
if queue.bind('fanout-rpc-requests'):
    print('Worker queue bound')

# Consume messages from RabbitMQ
for message in queue.consume_messages():

    # Display how long it took for the message to get here
    duration = time.time() - int(message.properties['timestamp'].strftime('%s'))
    print('Received RPC request published %.2f seconds ago' % duration)

    # Write out the message body to a temp file for facial detection process
    temp_file = utils.write_temp_file(message.body,
                                      message.properties['content_type'])

    # Detect faces
    result_file = detect.faces(temp_file)

    # Build response properties including the timestamp from the first publish
    properties = {'app_id': 'Chapter 6 Listing 2 Consumer',
                  'content_type': message.properties['content_type'],
                  'correlation_id': message.properties['correlation_id'],
                  'headers': {
                      'first_publish': message.properties['timestamp']}}

    # The result file could just be the original image if nothing detected
    body = utils.read_image(result_file)

    # Remove the temp file
    os.unlink(temp_file)

    # Remove the result file
    os.unlink(result_file)

    # Publish the response response
    response = rabbitpy.Message(channel, body, properties)
    response.publish('rpc-replies', message.properties['reply_to'])

    # Acknowledge the delivery of the RPC request message
    message.ack()

```

示例程序：Hash Consumer
```python
import os
import hashlib
import rabbitpy

# Open the connection and the channel
connection = rabbitpy.Connection()
channel = connection.channel()

# Create the worker queue
queue_name = 'hashing-worker-%s' % os.getpid()
queue = rabbitpy.Queue(channel, queue_name,
                       auto_delete=True,
                       durable=False,
                       exclusive=True)

# Declare the worker queue
if queue.declare():
    print('Worker queue declared')

# Bind the worker queue
if queue.bind('fanout-rpc-requests'):
    print('Worker queue bound')

# Consume messages from RabbitMQ
for message in queue.consume_messages():

    # Create the hashing object
    hash_obj = hashlib.md5(message.body)

    # Print out the info, this might go into a database or log file
    print('Image with correlation-id of %s has a hash of %s' %
          (message.properties['correlation_id'],
           hash_obj.hexdigest()))

    # Acknowledge the delivery of the RPC request message
    message.ack()

```


## topic交换器

### 特点

- 消息会被投递到匹配路由键的队列中（*匹配下一.之前的所有字符,#匹配所有字符）。

### 示例场景

![](https://img2020.cnblogs.com/blog/1030146/202010/1030146-20201009211241462-186905715.jpg)



## headers交换器

### 特点

- 使用消息属性中的headers属性匹配。
- queue.bind，x-match指定匹配策略，其他参数表示绑定值
- 绑定策略可能会使得性能降低

示例代码
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        exchange = rabbitpy.Exchange(channel,
                                     'headers-rpc-requests',
                                     exchange_type='headers')
        exchange.declare()

```

示例程序：publisher
```python
import os
import rabbitpy
import time
from ch6 import utils

# Open the channel and connection
connection = rabbitpy.Connection()
channel = connection.channel()

# Create the response queue that will automatically delete, is not durable and
# is exclusive to this publisher
queue_name = 'response-queue-%s' % os.getpid()
response_queue = rabbitpy.Queue(channel,
                                queue_name,
                                auto_delete=True,
                                durable=False,
                                exclusive=True)
# Declare the response queue
if response_queue.declare():
    print('Response queue declared')

# Bind the response queue
if response_queue.bind('rpc-replies', queue_name):
    print('Response queue bound')

# Iterate through the images to send RPC requests for
for img_id, filename in enumerate(utils.get_images()):

    print('Sending request for image #%s: %s' % (img_id, filename))

    # Create the message
    message = rabbitpy.Message(channel,
                               utils.read_image(filename),
                               {'content_type': utils.mime_type(filename),
                                'correlation_id': str(img_id),
                                'headers': {'source': 'profile',
                                            'object': 'image',
                                            'action': 'new'},
                                'reply_to': queue_name},
                               opinionated=True)

    # Pubish the message
    message.publish('headers-rpc-requests')

    # Loop until there is a response message
    message = None
    while not message:
        time.sleep(0.5)
        message = response_queue.get()

    # Ack the response message
    message.ack()

    # Caculate how long it took from publish to response
    duration = (time.time() -
                time.mktime(message.properties['headers']['first_publish']))

    print('Facial detection RPC call for image %s total duration: %s' %
          (message.properties['correlation_id'], duration))

    # Display the image in the IPython notebook interface
    utils.display_image(message.body, message.properties['content_type'])

print('RPC requests processed')

# Close the channel and connection
channel.close()
connection.close()

```


示例程序：worker
```python
import os
import rabbitpy
import time
from ch6 import detect
from ch6 import utils

# Open the connection and the channel
connection = rabbitpy.Connection()
channel = connection.channel()

# Create the worker queue
queue_name = 'rpc-worker-%s' % os.getpid()
queue = rabbitpy.Queue(channel, queue_name,
                       auto_delete=True,
                       durable=False,
                       exclusive=True)

# Declare the worker queue
if queue.declare():
    print('Worker queue declared')

# Bind the worker queue
if queue.bind('headers-rpc-requests',
              arguments={'x-match': 'all',
                         'source': 'profile',
                         'object': 'image',
                         'action': 'new'}):
    print('Worker queue bound')

# Consume messages from RabbitMQ
for message in queue.consume_messages():

    # Display how long it took for the message to get here
    duration = time.time() - int(message.properties['timestamp'].strftime('%s'))
    print('Received RPC request published %.2f seconds ago' % duration)

    # Write out the message body to a temp file for facial detection process
    temp_file = utils.write_temp_file(message.body,
                                      message.properties['content_type'])

    # Detect faces
    result_file = detect.faces(temp_file)

    # Build response properties including the timestamp from the first publish
    properties = {'app_id': 'Chapter 6 Listing 2 Consumer',
                  'content_type': message.properties['content_type'],
                  'correlation_id': message.properties['correlation_id'],
                  'headers': {
                      'first_publish': message.properties['timestamp']}}

    # The result file could just be the original image if nothing detected
    body = utils.read_image(result_file)

    # Remove the temp file
    os.unlink(temp_file)

    # Remove the result file
    os.unlink(result_file)

    # Publish the response response
    response = rabbitpy.Message(channel, body, properties, opinionated=True)
    response.publish('rpc-replies', message.properties['reply_to'])

    # Acknowledge the delivery of the RPC request message
    message.ack()

```

## 交换器路由

交换器间绑定，使用RPC方法Exchange.Bind。

### 示例场景


![](https://img2020.cnblogs.com/blog/1030146/202010/1030146-20201009211108563-1049881640.jpg)


示例代码：
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        tpc = rabbitpy.Exchange(channel, 'events',
                                exchange_type='topic')
        tpc.declare()
        xch = rabbitpy.Exchange(channel, 'distributed-events',
                                exchange_type='x-consistent-hash')
        xch.declare()
        xch.bind(foo, '#')
```


## 一致性哈希交换器

用于消息队列的负载均衡，可以提升吞吐量

### 示例场景


示例代码：采用路由键的哈希值来分发消息
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        exchange = rabbitpy.Exchange(channel, 'image-storage',
                                     exchange_type='x-consistent-hash')
        exchange.declare()
```

示例代码：header中的属性值作为哈希值
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        exchange = rabbitpy.Exchange(channel, 'image-storage',
                                     exchange_type='x-consistent-hash',
                                     arguments={'hash-header': 'image-hash'})
        exchange.declare()

```

示例代码：队列的创建与绑定
```python
import rabbitpy

with rabbitpy.Connection() as connection:
    with connection.channel() as channel:
        for queue_num in range(4):
            queue = rabbitpy.Queue (channel, 'server%s' % queue_num)
            queue.declare()
            queue.bind('image-storage', '10')
```
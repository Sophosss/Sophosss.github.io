---
title: 简单上手Scrapy pipeline
author: Sophosss
layout: post
---

### Pipeline：

在一个item被spider抓取后，被发送到pipeline，pipeline通过顺序执行处理这些item。

每个pipeline是一个实现简单方法的Python类。它们收到一个item并对其执行操作，同时决定该item是否应继续通过或被丢弃并且不再处理。

![theme](https://Sophosss.github.io/assets/images/4.png)

官方文档提供了pipeline的一些主要功能，翻译总结一下就是：

- 验证爬取的数(检查item的某些字段，比如name)。
- 去重(并丢弃)，比如init，set。
- 结果保存文件或存储在数据库中。

以下是将处理结果存储在json文件的示例：

```python
import json

class JsspiderPipeline(object):
    
    def __init__(self):
        self.f = open("file.json","w")
        
    def process_item(self, item, spider):
        content = json.dumps(dict(item), ensure_ascii= False) + '\n'
        self.f.write(content)
        return item
    
    def close_spider(self,spider):
        self.f.close()
```

需要注意的是，如果需要存储图片，可以使用Scrapy自己的ImagesPipeline，或者自定义一个pipeline，以下是我使用ImagesPipeline处理图片的代码：

```python
import os
import scrapy
from settings import IMAGES_STORE as images_store
from scrapy.pipelines.images import ImagesPipeline

class MyPipeline(ImagesPipeline):
    def get_media_requests(self,item,info):
        image_link = item['imagelink']
        yield scrapy.Request(image_link)
    
    def item_completed(self,results,item,info):
        image_path = [x["path"] for ok, x in results if ok]
        os.rename(images_store + image_path[0], image_store + item["nickname"] + ".jpg")
        return item
```

使用`yield item`可以返回数据给pipeline，继续循环处理，或输出到屏幕，添加以下代码：

```python
import sys
reload(sys)
sys.setdefaultencoding("utf-8")
```

最后，如果是使用的自定义的pipeline，别忘了在settings.py中，将`ITEM_PIPELINES`的优先级数字设置在合适的范围内，执行顺序不能乱哦~
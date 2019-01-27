---
title: 解决 Scrapy 抓取中文内容显示不正常问题
tags: [python, crawler]
---

用 Scrapy 爬取中文内容保存到 JSON 文件时总是出现 Unicode 码，解决办法如下。

在 piplines.py 文件中插入如下代码：

```python
import json, codecs
class MyfirstPipeline(object):
    def __init__(self):
        self.file = codecs.open('result.json', 'w', encoding='utf-8')
    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + "\n"
        self.file.write(line.decode('unicode_escape'))
        return item
```

或者是：

```python
import json, codecs
class MyfirstPipeline(object):
    def __init__(self):
        self.file = codecs.open('result.json', 'w', encoding='utf-8')
    def process_item(self, item, spider):
        line = json.dumps(dict(item), ensure_ascii=False) + "\n"
        self.file.write(line)
        return item
```

在 setting.py 文件中插入如下代码：

```python
ITEM_PIPELINES = {
    'myfirst.pipelines.MyfirstPipeline': 800,
}
```

OK!
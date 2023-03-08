---
title: "Python将 PDF 转为图片和长图"
subtitle: "Python | PDF | 图片"
layout: post
author: "luoruiqing"
header-style: text
catalog:    true
tags:
  - Python
---



由于最近找工作, 发现很多 PDF 转图片都是收费的, 要么就是打水印的, 那只能利用程序能力解决了!

下面代码使用 Python 将 PDF 文件转为图片


#### 依赖库

- pillow
- pdf2image

## Demo


```py
from PIL import Image
import sys
from pdf2image import convert_from_path

pages = convert_from_path("./我的PDF文件.pdf")

images = []
for index, page in enumerate(pages):
    page.save(f"{index}.png", "png")
    images.append(Image.open(f"{index}.png"))


widths, heights = zip(*(i.size for i in images))

total_height = sum(heights)
max_width = max(widths)

new_im = Image.new('RGB', (max_width, total_height))

y_offset = 0
for image in images:
    new_im.paste(image, (0, y_offset))
    y_offset += image.size[1] + 20 # 间隔 20 像素, 若不需要删除即可

new_im.save('我的PNG文件.png')
```


这里的 demo 是垂直拼接图片, 水平也很简单, 上面代码简单改改即可.
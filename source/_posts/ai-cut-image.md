---
title: AI扣人像（image）
categories:
  - AI
excerpt: '通过AI实现扣人像的能力'
description: '通过AI实现扣人像的能力'
date: 2025-01-08 15:46:25
---

## 源码信息

requirements.txt

```txt
torch
torchvision
pillow
kornia
transformers
timm
```

main.py
```python
from PIL import Image
import matplotlib.pyplot as plt
import torch
from torchvision import transforms
from transformers import AutoModelForImageSegmentation

import time

# 开始计时
start_time = time.time()

model = AutoModelForImageSegmentation.from_pretrained(
    'briaai/RMBG-2.0', trust_remote_code=True
)
torch.set_float32_matmul_precision(['high', 'highest'][0])

# 检查CUDA是否可用，如果可用则将模型移动到GPU
if torch.cuda.is_available():
    model.to('cuda')
else:
    print("CUDA不可用，模型将在CPU上运行")
model.eval()

# Data settings
image_size = (1024, 1024)
transform_image = transforms.Compose(
  [
    transforms.Resize(image_size),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
  ]
)

"""
需要自己准备图片
"""
image = Image.open('./2.png')
# 将图像从RGBA模式转换为RGB模式
if image.mode == 'RGBA':
    image = image.convert('RGB')
input_images = transform_image(image).unsqueeze(0)

# Prediction
with torch.no_grad():
    preds = model(input_images)[-1].sigmoid().cpu()
pred = preds[0].squeeze()
pred_pil = transforms.ToPILImage()(pred)
mask = pred_pil.resize(image.size)
image.putalpha(mask)

image.save("no_bg_image_2.png")
# 计算处理时间
print(f"处理时间: {time.time() - start_time:.2f}秒")
```

## 效果

<div style="display: flex; align-items: center;">
  <div>
    运行前
    <img src="/image/ai-cut-image/2.png" alt="运行前" style="max-height: 750px; width: auto; margin-right: 10px;" />
  </div>
  <div>
    运行后
    <img src="/image/ai-cut-image/no_bg_image_2.png" alt="运行后" style="max-height: 750px; width: auto;" />
  </div>
</div>

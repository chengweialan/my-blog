---
title: VGGT学习记录
published: 2025-11-08
description: 'VGGT论文及代码解读'
image: './images/vggt/figure.png'
tags: ["人工智能", "Transformer"]
category: '人工智能'
draft: false 
lang: ''
series: '科研'
---

# VGGT论文阅读与代码解读(待续)

> 近期在用Transformer做3D视觉相关的工作，自然关注到2025 CVPR最佳论文 [VGGT: Visual Geometry Grounded Transformer](https://arxiv.org/abs/2503.11651)。VGGT能实现单次端到端从图像预测3D场景主要基本属性，其架构对我当前的Research有一定借鉴意义。借此机会记录论文及代码的阅读过程。


## 代码阅读

核心Module在```vggt/models/vggt.py```文件中，如下：

```python
class VGGT(nn.Module, PyTorchModelHubMixin):
    def __init__(self, img_size=518, patch_size=14, embed_dim=1024,
                 enable_camera=True, enable_point=True, enable_depth=True, enable_track=True):
        super().__init__()

        self.aggregator = Aggregator(img_size=img_size, patch_size=patch_size, embed_dim=embed_dim)

        self.camera_head = CameraHead(dim_in=2 * embed_dim) if enable_camera else None
        self.point_head = DPTHead(dim_in=2 * embed_dim, output_dim=4, activation="inv_log", conf_activation="expp1") if enable_point else None
        self.depth_head = DPTHead(dim_in=2 * embed_dim, output_dim=2, activation="exp", conf_activation="expp1") if enable_depth else None
        self.track_head = TrackHead(dim_in=2 * embed_dim, patch_size=patch_size) if enable_track else None
```

Aggregator是算法的主要模块，针对图片输入实现交替注意力机制（Alternating-attention），四个Head针对不同的下游任务进行设计，分别预测相机位姿、深度图像、3D point map和Track特征（用于后续实现Tracking）。

z g f h g f i yi j o o i o i j f tu g y g ku h k j h chen ge u o

<!-- ### 核心模块拆解
关键文件：`models/vgg_t.py`。以下是简化版代码解读：

```
```就、
#### Camera Head

AdaLN： 取消梯度
Small Transformer -->
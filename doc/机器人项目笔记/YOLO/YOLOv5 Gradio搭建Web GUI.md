# YOLOv5 Gradio搭建Web GUI实战

### 一、Gradio 简介与环境安装

#### 1. Gradio 核心优势

Gradio 是开源 Python 库，专为机器学习演示和 Web 应用开发设计，内置丰富交互组件，已封装前后端交互逻辑，无需额外编写复杂代码，仅需少量代码即可实现可视化 Web 界面。

#### 2. 环境安装（基于 yolov5 虚拟环境）

1. 打开终端，激活前文创建的 yolov5 虚拟环境：

    ```Bash
    conda activate yolov5
    ```
    
2. 执行安装命令，下载并安装 Gradio：

    ```Bash
    pip install gradio
    ```
    
3. 验证安装：安装完成后无报错，即可开始后续开发（国内源可加速下载，无需额外配置）。

### 二、完整 Web GUI 实现（集成全拓展功能）

以下为集成所有核心及拓展功能的完整代码，已通过注释标注关键功能点（实时更新、示例图片、公网分享等）。该代码一次性实现双输入源、参数调节、实时检测、示例预设、公网分享五大功能.

```Python
import torch
import gradio as gr

model = torch.hub.load("./","custom", path="runs/train/exp/weights/best.pt", source="local")

title = "YOLOv5 Object Detection"
description = "Upload an image to detect objects using a custom-trained YOLOv5 model.棒"

base_conf = 0.25
base_iou = 0.45

def det_image(img,conf,iou):
    model.conf = conf
    model.iou = iou
    results = model(img)
    return results.render()[0]

gr.Interface(inputs=[gr.Image(sources=["webcam", "upload"], label="选择输入源", type="numpy"),gr.Slider(minimum=0.0,maximum=1.0,value=base_conf),gr.Slider(minimum=0.0,maximum=1.0,value=base_iou)],
             outputs=["image"],
             fn=det_image,
             title=title,
             description=description,
             #实时更新
             live=True,
             #示例图片
             examples=[["./datasets/images/train/90.jpg",base_conf,base_iou],["./datasets/images/train/120.jpg",0.35,base_iou]]).launch(share=True)#公网
```

#### 代码结构与功能对应解析

代码按“模型加载→参数定义→检测函数→界面配置”逻辑组织，每处拓展功能均与注释精准对应，具体说明如下：

1. **模型加载**：通过`torch.hub.load` 加载本地训练好的模型，路径为 `runs/train/exp/weights/best.pt`，`source="local"` 指定从本地加载，避免网络请求，适配自定义训练模型。

2. **核心参数**：默认置信度阈值 `base_conf=0.25`、IOU 阈值 `base_iou=0.45`，与 YOLOv5 原生默认参数一致，确保检测效果基础稳定。

3. **检测函数**：`det_image` 函数接收图片、置信度、IOU 三个参数，动态更新模型阈值后执行检测，返回渲染后的检测结果（含目标标注框）。

4. **界面配置（拓展功能对应）**：
    双输入源：`gr.Image(sources=["webcam", "upload"])` 支持摄像头拍摄和本地图片上传两种方式，`type="numpy"` 适配模型输入格式，标签设为“选择输入源”，操作直观。

5. 参数调节：两个 `gr.Slider` 组件分别控制置信度、IOU 阈值，范围限定为 0.0-1.0，默认值关联前文定义的基础参数，支持实时微调检测精度。

6. 实时更新（注释对应）：`live=True` 开启动态触发机制，上传图片、调节滑动条或摄像头拍摄后，右侧结果将自动刷新，无需手动点击提交。

7. 示例图片（注释对应）：`examples` 预设两组示例数据，分别为 `./datasets/images/train/90.jpg`（默认阈值）和 `./datasets/images/train/120.jpg`（置信度阈值 0.35），用户可直接点击示例快速测试。

8. 公网分享（注释对应）：`launch(share=True)` 生成临时公网链接，支持异地访问，默认有效期 72 小时，适合演示和团队协作测试。

#### 运行与测试步骤

1. 保存代码：将上述代码保存为 `yolov5_gradio.py`，放置于 YOLOv5 根目录，确保与 `runs`、`datasets` 文件夹同级。

2. 启动程序：在激活的 yolov5 虚拟环境中，执行命令 `python yolov5_gradio.py`。

3. 访问界面：终端将输出两个地址，本地访问地址 `http://localhost:7860`（Ctrl+点击直接打开）、公网访问地址 `https://xxx.gradio.live`（72 小时有效）。

4. 功能测试：
    上传检测：点击“选择输入源”上传 `datasets/images/train` 下的图片，右侧自动显示标注结果，调节滑动条可过滤冗余框或补充漏检目标。

5. 摄像头检测：点击输入框下方“摄像头”按钮，授权后可实时拍摄并检测，适合动态场景测试。

6. 示例测试：直接点击界面下方预设示例，快速验证模型检测效果，无需手动上传。

### 三、整体流程回顾与优势总结

流程梳理：环境安装（Gradio）→ 编写完整代码（集成全功能）→ 运行测试 → 公网分享，全程无需编写前后端分离代码，仅通过 Gradio 组件配置即可实现 Web 可视化。

#### 运行与测试

1. 执行代码，终端将提示本地访问地址：`http://localhost:7860`。

2. 按 `Ctrl+点击` 地址打开浏览器，左侧上传图片（如死神VS火影数据集图片），右侧将自动显示检测结果。
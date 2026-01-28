# YOLOv5 修改网络结构（以C2f为例）

### 一、前置准备：借鉴YOLOv8代码

网络结构修改的核心是借鉴现有开源代码（YOLOv8），获取C2f模块相关实现，具体操作如下：

#### 1. 代码借鉴来源

```text

# YOLOv8 开源项目地址
https://github.com/ultralytics/ultralytics
```

#### 2. 代码获取与分析步骤

1. 访问上述 GitHub 地址，下载 YOLOv8 项目的 ZIP 压缩包，获取完整代码库。

2. 对比 YOLOv5 与 YOLOv8 项目结构差异：YOLOv8 为支持命令行执行方式，做了额外设计，源码结构与 YOLOv5 有所不同。

3. 定位 YOLOv8 网络结构定义位置：进入 YOLOv8 代码的 `ultralytics/nn/modules` 目录，该目录对应 YOLOv5 中的 `models/commons.py`，存储所有网络模块定义。

4. 提取核心模块：从 `ultralytics/nn/modules` 中复制 C2f相关模块代码，用于后续 YOLOv5 网络修改。

### 二、网络结构修改全流程（以C2f为例，4步完成）

核心修改顺序（从原文提取，无任何更改）：`models/commons.py` → 加入新增网络结构 → `models/yolo.py` → 设定网络结构的传参细节 → `models/yolov5*.yaml` → 修改现有模型结构配置文件 → `train.py` → 训练时指定模型结构配置文件

#### 第一步：修改 models/commons.py（加入C2f模块）

1. 打开 YOLOv5 项目的 `models/commons.py` 文件。

2. 将从 YOLOv8 `ultralytics/nn/modules` 中提取的 C2f（CRF）模块代码，复制到 `commons.py` 中（建议放在 C3 模块之前，便于后续替换）。

3. 参数适配与重命名：

    - C2f 模块中若包含 V5 中没有的 `K` 参数（V5 中仅有 `G` 和 `E` 参数），需从 YOLOv8 代码中复制 `K` 参数相关定义，一并粘贴到 `commons.py`。

    - 为避免覆盖 V5 原有模块，将新增的 C2f 模块重命名（如添加前缀 `C2F_`），确保命名符合项目规范，不与原有模块冲突。

#### 第二步：修改 models/yolo.py（设定传参细节）

1. 打开 `models/yolo.py` 文件，找到 `parse_model` 函数。

2. 传参适配：C2f 模块与 C3 模块的结构、参数完全一致，可直接参考 C3 模块的传参逻辑，在 `parse_model` 函数中添加 C2f 模块的传参处理：

#### 第三步：修改 models/yolov5*.yaml（替换网络模块）

1. 复制现有模型配置文件（如 `models/yolov5s.yaml`），重命名为 `yolov5s_c2f.yaml`。

2. 打开 `yolov5s_c2f.yaml` 文件，修改 backbone 部分：将所有 `C3` 模块替换为新增的 `C2f` 模块。

#### 第四步：修改 train.py（训练时指定配置文件）

训练时需指定修改后的配置文件，确保模型加载 C2f 模块，核心代码与操作如下：

在train.py的参数解析部分，修改--cfg参数的默认值，使其指向修改后的C2f模型配置文件，避免训练时手动指定路径，代码如下：

```Python

parser.add_argument('--cfg', type=str, default=ROOT / 'models/yolov5s_c2f.yaml', help='model.yaml path')
```
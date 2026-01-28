# YOLOv5 引入 SE 注意力机制

### 一、前置准备：借鉴SE模块开源代码

#### 1. 代码借鉴来源

```Plain Text

# SE注意力机制开源代码地址
https://github.com/Zhugekongan/Attention-mechanism-implementation
```

#### 2. 代码提取与核心分析

1. 访问上述 GitHub 地址，定位 `model` 目录下的 `SE block` 代码，复制完整模块实现（含导包语句）。

2. SE 模块核心功能：通过“平均池化→通道压缩→通道恢复→激活→与原始特征相乘”，强化关键通道特征，抑制无关信息。

3. SE 模块关键特性（与 C2f/C3 差异）：

    - 输入通道数 = 输出通道数（先压缩再恢复，通道数不变）。

    - 仅需 2 个参数：`in_channels`（输入通道数）、`ratio`（通道压缩比，决定中间隐层通道数）。

    - 中间隐层通道数 = `in_channels // ratio`（压缩比可自定义，如 2）。

### 二、引入SE机制全流程（4步完成，核心是层编号调整）

核心修改顺序：  

`models/commons.py` → 加入 SE 模块代码 → `models/yolov5*.yaml` → 修改配置文件（加 SE 层+调整层编号） → `models/yolo.py` → 配置 SE 传参细节 → `train.py` → 训练时指定自定义配置文件

#### 第一步：修改 models/[commons.py](commons.py)（添加SE模块）

1. 打开 YOLOv5 项目的 `models/commons.py` 文件。

2. 复制 SE 模块代码（含导包语句），粘贴到文件中（建议放在 C2f/C3 模块之前，避免冲突）：

3. 确保导包语句 `import torch.nn.functional as F` 已添加，避免“F未定义”报错。

#### 第二步：修改 models/yolov5*.yaml（添加SE层+调整层编号）

1. 复制现有模型配置文件（如 `models/yolov5s.yaml`），重命名为 `yolov5s_se.yaml`（标识含 SE 机制的模型）。

2. 添加 SE 层：选择在 backbone 最后一层加入 SE 模块（原文推荐位置），修改 backbone 部分配置：

```YAML

# 修改后的 backbone 配置（仅展示新增 SE 层部分，其余保持不变）
backbone:
  [[-1, 1, Conv, [64, 6, 2, 2]],
   [-1, 1, Conv, [128, 3, 2]],
   [-1, 3, C3, [128]],
   [-1, 1, Conv, [256, 3, 2]],
   [-1, 6, C3, [256]],
   [-1, 1, Conv, [512, 3, 2]],
   [-1, 9, C3, [512]],
   [-1, 1, Conv, [1024, 3, 2]],
   [-1, 3, C3, [1024]],
   [-1, 1, SPPF, [1024, 5]],
   [-1, 1, SE, [1024, 2]],  # 新增 SE 层：输入通道1024，压缩比2
  ]
```

1. 关键操作：调整后续层的 `from` 参数（新增 SE 层会改变原有层编号）：

    - 新增 SE 层为第 10 层（原 SPPF 为第 9 层），因此原第 10 层及以后的层编号均 +1。

    - 重点调整 head 部分依赖 backbone 层的 `from` 参数：

        - 原第 19 层（现第 20 层）：输入从「11, 14」改为「12, 15」（均 +1）。

        - 原 Detect 层输入「17, 20, 23」改为「18, 21, 24」（均 +1）。

    - 核心原则：新增层之后，所有引用该层及后续层的 `from` 参数，均需按新增层数累加调整。

#### 第三步：修改 models/[yolo.py](yolo.py)（配置SE模块传参细节）

1. 打开 `models/yolo.py` 文件，找到 `parse_model` 函数（模型解析核心函数）。

2. 在 `parse_model` 中添加 SE 模块的传参逻辑（新增 `elif` 分支，参考 C3/C2f 传参格式）：

```Python

elif m is SE:  # 新增 SE 模块传参逻辑
    c1 = ch[f]  # SE 输入通道 = 上一层输出通道
    c2 = args[0]    # SE 输出通道 = 输入通道（通道数不变）
    # 通道数按 width_multiple 缩放，且确保可被8整除
    if c2 != no:  # if not output
                c2 = make_divisible(c2 * gw, 8)
    args = [c1, *args[1:]]  # 传入 in_channels 和 ratio 参数
```

1. 关键说明：

    - SE 模块无需 `n`（重复次数），默认 `n=1`，无需额外处理。

    - 必须通过 `make_divisible` 确保通道数可被 8 整除，适配 GPU 加速。

    - `args` 仅保留 `in_channels` 和 `ratio`（从配置文件传入），符合 SE 模块初始化参数要求。

#### 第四步：修改 [train.py](train.py)（训练时指定配置文件）

1. 打开 `train.py`，修改 `--cfg` 参数的默认值，指向含 SE 机制的配置文件：

```Python

# 从原文提取的修改后代码（无任何更改）
parser.add_argument('--cfg', type=str, default=ROOT / 'models/yolov5s_se.yaml', help='model.yaml path')
```
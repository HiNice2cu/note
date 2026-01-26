# YOLOv5 Windows 11 环境安装

## 一、安装前准备

### 1. 系统与硬件要求

- 系统：Windows 11（课程基于该系统演示，Win10 可参考适配）

- CPU：无强制要求（示例为 i5-8300H）

- 显卡：NVIDIA 独立显卡（示例为 1050Ti，需匹配对应 CUDA 版本）

- 显卡驱动：需支持目标 PyTorch 版本（示例驱动版本 516.94）

- 网络：稳定网络（用于下载安装包与依赖，部分包体积较大）

### 2. 核心下载地址

- Miniconda（清华镜像）：用于创建隔离 Python 环境

- PyPI 国内源（清华大学）：加速 Python 包下载

- PyTorch 官网：获取指定版本安装命令

- YOLOv5 GitHub 仓库：下载 7.0 稳定版源码

## 二、分步安装流程

### 第一步：安装 Miniconda（轻量级环境管理器）

#### 目的

隔离不同项目的 Python 环境，避免依赖冲突，比 Anaconda 更轻量化（无需预制多余包）。

#### 操作步骤

1. 下载版本：访问Miniconda清华镜像具体地址 `https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/`，搜索 `PY38`，选择 `Miniconda3-py38_22.11.1-1-Windows-x86_64.exe`（Windows 64 位，Python 3.8 兼容版），其他系统需匹配对应版本。

2. 安装向导：

    - 双击安装包，点击「Next」→ 勾选「I agree」→ 选择安装类型「Just Me」（个人使用）。

    - 安装路径：优先选 C 盘（无空间可选其他盘），**路径必须纯英文无空格**（避免安装失败）。

    - 关键勾选：务必勾选「Add Miniconda3 to my PATH environment variable」（虽系统提示不推荐，但后续使用必需）。

    - 完成安装：点击「Install」→ 后续步骤取消两个默认勾选 → 点击「Finish」。

### 第二步：创建并激活 Python 虚拟环境

#### 核心命令

1. 打开 CMD：按 Win 键 + R，输入 `CMD` 打开命令提示符。

2. 创建环境：

    ```Bash
    
    conda create -n yolov5 python=3.8
    ```

    - 说明：`yolov5` 为环境名称（可自定义），`python=3.8` 为指定 Python 版本（必需，避免高版本兼容问题）。

3. 确认创建：输入 `y` 回车，等待环境创建完成。

4. 激活环境：

    ```Bash
    
    conda activate yolov5
    ```

    - 验证：命令行前缀显示 `(yolov5)` 即激活成功。

### 第三步：配置 PyPI 国内源（加速包下载）

#### 目的

解决 Python 官方仓库（国外）下载慢的问题，使用清华大学镜像。

#### 配置命令

在激活的 `yolov5` 环境中执行：

```Bash

pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

- 说明：命令会将国内源写入 `pip.ini` 文件，全局生效，后续 `pip install` 自动走国内源。

### 第四步：安装 PyTorch 1.8.2（YOLOv5 兼容版）

#### 版本选择理由

当前 PyTorch 已更新至 2.1.20，但 YOLOv5 7.0 对高版本兼容性不佳，**必须安装 1.8.2 版本**，避免训练时出现报错。

#### 按显卡型号选择 CUDA 版本

|显卡类型|推荐 CUDA 版本|原因|
|---|---|---|
|30 系（3050/3060 等）、40 系|11.1|不支持 CUDA 11 以下版本|
|16 系（1650/1660 等）|10.2|半精度训练兼容性要求|
|无显卡/AMD 显卡|CPU 版本|无需 CUDA，直接安装 CPU 版 PyTorch|
|老显卡（如 1050Ti）|11.1|兼容性更优|
#### 安装命令

1. 复制对应版本命令（从 PyTorch 官网「Previous Versions」获取）：

    - CUDA 11.1 版本（30/40 系、老显卡）：

        ```Bash
        
        pip install torch==1.8.2+cu111 torchvision==0.9.2+cu111 torchaudio==0.8.2 -f https://download.pytorch.org/whl/lts/1.8/torch_lts.html
        ```

    - CUDA 10.2 版本（16 系）：

        ```Bash
        
        pip install torch==1.8.2+cu102 torchvision==0.9.2+cu102 torchaudio==0.8.2 -f https://download.pytorch.org/whl/lts/1.8/torch_lts.html
        ```

    - CPU 版本（无 NVIDIA 显卡）：

        ```Bash
        
        pip install torch==1.8.2+cpu torchvision==0.9.2+cpu torchaudio==0.8.2 -f https://download.pytorch.org/whl/lts/1.8/torch_lts.html
        ```

2. 执行命令：在激活的 `yolov5` 环境中粘贴命令，回车安装。

    - 注意：安装包约 3.1G，首次下载可能较慢，需耐心等待；若之前安装过，会优先使用缓存加速。

#### 验证 GPU 可用性（关键步骤）

无需额外安装 CUDA 和 CUDNN，仅需 PyTorch 安装正确 + 显卡驱动兼容：

1. 在 CMD 中执行：

    ```Bash
    
    python
    import torch
    torch.cuda.is_available()  # 返回 True 则说明 GPU 可调用
    ```

2. 查看显卡驱动：桌面 → 右键 → 英伟达控制面板 → 系统信息 → 组件 → 查看 `NV CUDA 64.DLL` 版本（即驱动支持的最高 CUDA 版本，需 ≥ 安装的 CUDA 版本）。

    - 若驱动不兼容：升级显卡驱动后重试。

### 第五步：下载 YOLOv5 7.0 源码并配置依赖

#### 1. 下载源码

- 访问 YOLOv5 GitHub 仓库 → 点击「Releases」→ 选择「V7.0」→ 下载「Source code (zip)」。

- 解压：将 zip 文件解压到桌面，确保桌面生成 `yolov5-7.0` 文件夹（路径纯英文无空格）。

#### 2. 修改 requirements.txt（适配版本）

打开 `yolov5-7.0` 文件夹中的 `requirements.txt`，调整 3 处配置：

- `numpy==1.20.3`（指定版本，避免兼容问题）

- `pillow==8.3.0`（指定版本，适配 YOLOv5 7.0）

- 注释掉 `torch` 和 `torchvision` 相关行（已手动安装，避免重复安装为 CPU 版本）：

    ```Plain Text
    
    # torch>=1.7.0
    # torchvision>=0.8.1
    ```

#### 3. 安装 YOLOv5 依赖

1. 进入源码目录：

    - 方法 1：CMD 中执行 `cd Desktop\yolov5-7.0`（根据实际解压路径调整）。

    - 方法 2：打开 `yolov5-7.0` 文件夹，地址栏输入 `CMD` 回车（直接进入当前目录的 CMD）。

2. 激活环境（若已退出）：`conda activate yolov5`。

3. 安装依赖：

    ```Bash
    
    pip install -r requirements.txt
    ```

    - 验证：无报错且显示「Successfully installed」即依赖安装完成。

### 第六步：模型检测（验证环境是否生效）

#### 1. 下载预训练模型

- 首次运行 `detect.py` 时，系统会自动下载 `yolov5s.pt` 预训练模型（约 140M）。

- 加速方案：若终端下载慢，复制终端中显示的模型下载链接，用浏览器下载，下载后复制到 `yolov5-7.0` 根目录。

#### 2. 运行检测脚本

在 `yolov5-7.0` 目录的 CMD 中执行：

```Bash

python detect.py
```

#### 3. 验证结果

- 若终端显示 `GPU:0` 或 `CUDA:` 相关信息，说明 GPU 版本生效。

- 若显示 `CPU`，说明环境配置失败（需检查 PyTorch 安装版本与显卡适配性）。

- 检测结果：生成 `runs/detect/exp2` 文件夹，内含测试图片的检测结果（框选目标），即环境搭建成功。

## 三、关键注意事项

1. 路径规范：所有安装路径、文件夹名称必须纯英文无空格，否则会导致脚本运行失败。

2. 版本匹配：Python 3.8 + PyTorch 1.8.2 + YOLOv5 7.0 为稳定组合，不可随意升级版本。

3. 无需额外安装 CUDA/CUDNN：PyTorch 已内置对应版本，仅需显卡驱动支持即可调用 GPU。

4. 常见问题：

    - 环境激活失败：检查 Miniconda 是否添加到系统 PATH。

    - 依赖安装报错：删除 `requirements.txt` 中冲突的包版本，重新执行 `pip install`。

    - GPU 调用失败：确认显卡型号、CUDA 版本、驱动版本三者匹配，重新安装 PyTorch。
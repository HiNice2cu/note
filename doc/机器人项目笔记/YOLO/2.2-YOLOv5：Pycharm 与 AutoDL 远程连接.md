# YOLOv5：Pycharm 与 AutoDL 远程连接

### 一、Pycharm 安装与基础配置

#### 1. 安装关键注意事项

- **版本选择**：必须下载 **专业版**（Community 社区版不支持远程服务器连接功能），演示使用 2022.1.3 版本（需与实操一致可点击官网「Other Versions」选择历史版本，避免最新版兼容性问题）。

- **安装选项**：勾选「Update context menu」和「Add open folder as project」，支持右键文件夹直接打开为 Pycharm 项目，提升操作便捷性。

- **安装路径**：选择自定义路径（避免中文路径），按提示完成安装（无需额外复杂配置）。

#### 2. 本地项目打开与解释器配置

##### （1）打开本地 YOLOv5 项目

- 右键 YOLOv5 项目文件夹（如 `yolov5-master`），选择「Open folder as PyCharm Project」，直接在 Pycharm 中加载项目。

- 初始状态下右下角会提示「No Python interpreter configured」，需手动配置环境。

##### （2）配置 Conda 虚拟环境解释器

```Python
# 核心操作步骤（无代码，纯界面操作）
1. 打开 File → Settings → Project: yolov5-master → Python Interpreter
2. 点击右上角「+」号 → 选择「Conda Environment」→ 勾选「Existing environment」
3. 点击右侧「...」浏览，找到 yolov5 虚拟环境的 Python 可执行文件：
   路径示例：C:\Users\用户名\Miniconda3\envs\yolov5\python.exe
4. 可选勾选「Make available to all projects」（所有项目共享该解释器）
5. 点击「OK」→ 再次「OK」，等待右下角「Updating Python interpreter」进程完成
```

- **备选方案（未识别 Conda 环境时）**：若上述步骤未找到环境，选择「System Interpreter」→ 手动浏览至 yolov5 环境的 `python.exe` 文件，完成配置。

- **关键提醒**：配置后需等待「Updating indexes」进程结束（该进程运行时无法执行代码），仅「Updating Python interpreter」运行不影响操作。

#### 3. 终端设置（解决 Conda 环境激活问题）

##### （1）默认终端问题

Pycharm 默认终端为 PowerShell，无法正常激活 Conda 环境，导致 `pip install` 安装的包无法导入，需切换至 CMD 终端。

##### （2）终端切换步骤

```Python
# 核心操作步骤（无代码，纯界面操作）
1. 打开 File → Settings → Tools → Terminal
2. 在「Application settings」→「Shell path」中，选择「Command Prompt (CMD)」
3. 点击「Apply」→「OK」，关闭当前终端并重新打开
4. 新终端将自动激活 yolov5 环境（前缀显示 (yolov5)）
```

- **验证效果**：在新终端执行 `pip install jieba`，安装后在 Python 代码中 `import jieba` 无报错，说明配置成功。

### 二、Pycharm 连接 AutoDL 远程服务器

#### 1. 前置准备

- 启动 AutoDL 服务器实例（参考第九章），获取「SSH 登录指令」和「登录密码」（AutoDL 控制台可直接复制）。

- 确保 Pycharm 为专业版，已完成本地环境配置。

#### 2. SSH 连接配置

```Python
# 核心操作步骤（无代码，纯界面操作）
1. 打开 File → Settings → Tools → SSH Configurations
2. 点击「+」号新建配置，填写以下信息：
   - Host：从 AutoDL「SSH 登录指令」中提取（格式示例：region-xxx.autodl.com）
   - Port：从 SSH 登录指令中提取（格式示例：215583）
   - Username：默认 root
   - Password：点击「Set password」，粘贴 AutoDL 登录密码
3. 点击「Test Connection」，提示「Successfully connected」即为配置成功
4. 点击「OK」保存配置
```

#### 3. 远程解释器配置（关键步骤）

```Python
# 核心操作步骤（无代码，纯界面操作）
1. 打开 File → Settings → Project: yolov5-master → Python Interpreter
2. 点击「+」号 → 选择「SSH Interpreter」→ 勾选「Existing server configuration」
3. 选择上述新建的 AutoDL SSH 配置 → 点击「Next」
4. 配置远程 Python 解释器：
   - Interpreter：浏览选择服务器上 Conda 环境的 Python 路径（示例：/root/miniconda3/bin/python）
     （可通过 AutoDL Jupyter Lab 终端执行 `which python` 查找路径）
   - Sync folders（项目同步）：
     - Local path：本地 YOLOv5 项目路径（自动识别，无需修改）
     - Remote path：服务器上项目存储路径（建议新建文件夹：/root/yoloV5）
5. 点击「Finish」→「Apply」→「OK」，开始自动上传本地项目文件至服务器
```

- **上传提醒**：文件上传完成后将提示「286 个文件已传输」，需等待服务器端「Updating indexes」进程结束，再执行远程代码。

### 三、远程服务器项目运行与文件同步

#### 1. 远程代码运行

- 配置完成后，右键 `detect.py` → 选择「Run detect」，代码将在 AutoDL 服务器上执行（终端显示远程执行路径）。

- 代码修改同步：本地修改代码后（如修改检测目标为 `bus.jpg`），Pycharm 会自动上传更新至服务器（可通过「Tools → Deployment → Automatic Upload」开启/关闭该功能）。

#### 2. 服务器文件下载（训练结果同步至本地）

服务器端训练结果（如 `runs/train/EXP4`）不会自动同步至本地，需手动下载：

```Python
# 核心操作步骤（无代码，纯界面操作）
1. 打开 Tools → Deployment → Browse Remote Host（右侧弹出远程文件窗口）
2. 在远程窗口中找到目标文件夹（如 /root/优乐V5/runs/detect/EXP4）
3. 右键该文件夹 → 选择「Download from here」，指定本地保存路径
4. 等待下载完成，即可在本地查看服务器端检测/训练结果
```

#### 3. 远程终端使用

- 打开 Pycharm 终端，右上角选择「Remote Python 3.8（AutoDL-YOLOv5）」，即可进入服务器终端。

- 可直接在该终端执行服务器端命令（如 `python train.py`、`pip install 依赖包`），与 AutoDL Jupyter Lab 终端功能一致。

### 四、多解释器切换（本地 UI 与远程训练兼顾）

服务器端未安装 PySide6 等 UI 依赖，无法运行可视化界面代码，需切换至本地解释器运行：

```Python
# 核心操作步骤（无代码，纯界面操作）
1. 右键需运行的 UI 代码（如 base_ui.py）→ 选择「Edit Configurations」
2. 在「Python interpreter」中，选择本地 yolov5 环境解释器（而非远程解释器）
3. 点击「Apply」→「OK」，再点击「Run base_ui」，即可在本地启动可视化界面
```

- 优势：同一项目可灵活切换本地/远程解释器，远程用于训练（算力充足），本地用于 UI 演示（无需服务器依赖）。

### 五、关键注意事项

1. **远程连接前提**：必须使用 Pycharm 专业版，社区版无 SSH 连接功能；AutoDL 服务器需处于「运行中」状态。

2. **环境一致性**：服务器端需提前配置 YOLOv5 环境（参考第九章），确保依赖包与本地一致，避免运行报错。

3. **文件同步方向**：本地→服务器自动同步（开启 Automatic Upload 后），服务器→本地需手动下载，训练结果需及时下载备份。

4. **成本控制**：远程训练完成后，及时在 AutoDL 控制台关闭服务器，避免长时间计费；关闭后 SSH 连接自动断开，再次启动需重新配置（IP/端口可能变化）。

5. **依赖安装**：服务器端缺失的依赖包，需在远程终端执行 `pip install` 安装，不可在本地终端安装（本地安装仅作用于本地环境）。

### 六、整体流程回顾

1. Pycharm 安装（专业版+关键选项勾选）→ 本地项目打开→ 解释器配置（Conda/系统解释器）→ 终端切换（CMD 激活环境）。

2. AutoDL SSH 配置（Host/Port/账号密码）→ 远程解释器配置（服务器 Python 路径+项目同步）→ 本地文件自动上传。

3. 远程运行代码（训练/检测）→ 服务器结果手动下载→ 本地 UI 代码切换本地解释器运行。
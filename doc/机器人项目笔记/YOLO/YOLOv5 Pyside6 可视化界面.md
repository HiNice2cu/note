# YOLOv5 Pyside6 可视化界面

### 一、可视化界面环境安装

需安装 Pyside6 框架、QT Designer 设计工具及 VS Code 辅助插件，环境基于前文 yolov5 虚拟环境搭建。

#### 1. 核心依赖安装

在激活的 yolov5 虚拟环境中执行以下命令：

```Bash

# 安装 Pyside6（含 QT Designer）
pip install pyside6
# 安装 jupyter lab（辅助模型调用验证）
pip install jupyter lab
```

#### 2. QT Designer 定位与快捷方式创建

QT Designer 随 Pyside6 一并安装，需找到其路径并创建桌面快捷方式：

1. 查找 Python 安装路径：打开 CMD，输入 `where python`，获取当前 yolov5 环境的 Python 路径（如 `C:\Miniconda3\envs\yolov5\python.exe`）。

2. 定位 QT Designer：进入 Python 路径下的 `C:\Users\12804\miniconda3\envs\yolov5\Lib\site-packages\PySide6` 文件夹，找到 `designer.exe` 文件。

3. 创建快捷方式：右键 `designer.exe`，选择「发送到」→「桌面快捷方式」，便于后续快速启动。

#### 3. VS Code 插件安装（UI 编译支持）

打开 VS Code，在左侧扩展栏搜索「QT for Python」，点击安装，用于将 QT Designer 生成的 `.ui` 文件编译为 Python 代码。

### 二、QT Designer 界面设计（拖拽式可视化）

#### 1. 新建 UI 项目

1. 双击桌面 `designer.exe` 启动工具，选择「Main Window」→ 点击「创建」，生成空白窗口。

2. 移除默认菜单栏（无需保留），按以下规划设计界面布局。

#### 2. 界面组件拖拽与布局

#### 3. 保存 UI 文件

按 `Ctrl+S` 保存文件，路径选择 YOLOv5 根目录，文件名为 `m` `ain` `_window.ui`（后缀为 `.ui`，不可修改）。

### 三、UI 文件编译（.ui → .py）

通过 VS Code 插件将设计好的 `.ui` 文件编译为 Python 代码，便于后续逻辑编写：

1. 用 VS Code 打开 YOLOv5 根目录，找到 `main_window.ui` 文件。

2. 右键点击该文件，选择「Compile QT UI File」，自动生成 `main_window` `_ui` `.py` 文件（含界面组件的 Python 定义）。

### 四、核心代码实现（图片/视频检测逻辑）

新建 Python 文件 `base_ui.py`，导入编译后的 UI 模块，实现信号绑定、文件读取、模型调用、结果显示等逻辑，代码完全复刻实操演示：

```Python

# 导入核心依赖
import sys
import torch
import cv2
# 导入 PySide6 相关模块
from PySide6.QtWidgets import QMainWindow, QApplication, QFileDialog
from PySide6.QtGui import QPixmap,QImage
# 导入编译后的 UI 模块（同步修改为新编译文件名）
from main_window_ui import Ui_MainWindow
from PySide6.QtCore import QTimer

def convert2QImage(img):
    """将 OpenCV 图像转换为 QImage 格式"""
    height, width, channel = img.shape
    return QImage(img, width, height, width*channel, QImage.Format_RGB888)

class MainWindow(QMainWindow, Ui_MainWindow):
    def __init__(self):
        super(MainWindow, self).__init__()
        # 初始化 UI 界面
        self.setupUi(self)
        self.model = torch.hub.load("./","custom", path="runs/train/exp/weights/best.pt", source="local")
        self.timer = QTimer()
        self.timer.setInterval(1)
        self.video = None
        self.bind_slots()

    def image_pred(self,file_path):
        # 图像预测逻辑
        results = self.model(file_path)
        image = results.render()[0]  # 显示第一张图像的结果
        return convert2QImage(image)
    
    def open_image(self):
        self.timer.stop()  
        file_path=QFileDialog.getOpenFileName(self, dir="./datasets/images/train", filter="*.png *.jpg *.jpeg")
        if file_path[0]:
            file_path=file_path[0]
            qimage= self.image_pred(file_path)
            self.input.setPixmap(QPixmap(file_path))
            self.output.setPixmap(QPixmap.fromImage(qimage))

    def video_pred(self):
        ret, frame = self.video.read()
        if not ret:
            self.timer.stop()
        else:
            frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            self.input.setPixmap(QPixmap.fromImage(convert2QImage(frame)))
            results = self.model(frame)
            image = results.render()[0]
            self.output.setPixmap(QPixmap.fromImage(convert2QImage(image)))
    
    def open_video(self):
        file_path=QFileDialog.getOpenFileName(self, dir="./datasets", filter="*.mp4")
        if file_path[0]:
            file_path=file_path[0]
            self.video = cv2.VideoCapture(file_path) 
            self.timer.start()              

    def bind_slots(self):
        # 绑定按钮点击事件到相应的槽函数
        self.det_image.clicked.connect(self.open_image)
        self.det_video.clicked.connect(self.open_video)
        self.timer.timeout.connect(self.video_pred)
        
# 程序入口
if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    app.exec()
```

### 五、关键功能逻辑解析

#### 1. 核心组件与信号绑定

- 信号与槽机制：通过`clicked.connect()` 将按钮点击事件与函数绑定（如代码中 `det_image` 绑定 `open_image`、`det_video` 绑定 `open_video`）。

- 计时器：解决 QT 界面阻塞问题，视频检测时通过 `QTimer` 定时读取帧，设置 1 毫秒间隔，避免循环阻塞界面刷新，同时初始化 `self.video = None` 防止资源残留。

#### 2. 图片检测核心流程

1. 文件选择：通过 `QFileDialog.getOpenFileName()` 打开图片，默认路径为 `./datasets/images/train`（同步代码中 datasets 复数格式），过滤 `.png/.jpg/.jpeg` 格式。

2. 模型检测：调用训练好的 `best.pt` 模型（路径为 `runs/train/exp/weights/best.pt`），通过 `results.render()` 提取检测结果（array 格式）。

3. 格式转换：代码中 `convert2QImage()` 函数实现格式转换，先适配图像宽高通道，直接将处理后图像转为 QImage 格式，确保 QT 正常显示。

#### 3. 视频检测核心流程

1. 视频初始化：`cv2.VideoCapture()` 读取视频文件，默认打开路径为 `./datasets`（复数格式），计时器启动后按 1 毫秒间隔读取帧，保证流畅播放。

2. 帧处理：循环读取视频帧，先将 BGR 格式转为 RGB 格式，再执行「原始帧显示→模型检测→检测结果显示」流程，逻辑与图片检测一致。

3. 终止条件：视频读取完毕（`ret=False`）时，停止计时器释放资源，避免后台占用。

#### 4. 冲突优化

- 图片检测时调用 `self.timer.stop()`，停止视频检测计时器，避免两者同时运行冲突，同时清空之前的视频资源。

- 计时器间隔固定为 1 毫秒（代码中已设置），无需额外调整，可直接实现视频流畅播放。

### 六、界面运行与测试

#### 1. 运行程序

在 yolov5 虚拟环境中，进入代码所在目录，执行（同步修改为新文件名）：

```Bash
python base_ui.py
```

#### 2. 功能测试

1. 图片检测：点击「图片检测」按钮→ 选择 `datasets/images/train`（复数格式）下的图片（如 30.jpg）→ 左侧显示原始图片，右侧显示检测结果（标注daitu、mingren）。

2. 视频检测：点击「视频检测」按钮→ 选择 `datasets`（复数格式）下的 MP4 视频（如 BVN.mp4）→ 左侧播放原始视频，右侧实时显示检测结果。

3. 终止操作：视频检测时按 `Ctrl+C` 终止程序（不可直接关闭窗口），避免资源泄露。

### 七、常见问题与注意事项

1. 界面无响应：视频检测时未使用计时器，或未初始化 `self.video`，导致循环阻塞 QT 事件队列，需确保 `video_pred` 由 `QTimer` 触发，且初始化视频资源。

2. 图片显示异常：检查`convert2QImage()` 函数参数，确保图像宽、高、通道数适配，且格式设为 `QImage.Format_RGB888`。

3. 模型加载失败：检查 `best.pt` 路径是否正确（需与代码一致，为 `runs/train/exp/weights/best.pt`），同时确认模型与 YOLOv5 版本兼容。

4. 依赖冲突：安装 Pyside6 后若报错，执行`pip uninstall pywin32` 卸载冲突包，重新启动程序即可。

### 八、整体流程回顾

1. 环境搭建：安装 Pyside6、QT Designer 及辅助工具，创建设计工具快捷方式，确保环境与 yolov5 虚拟环境一致。

2. UI 设计：通过 QT Designer 拖拽组件、设置属性，保存为 `main_window.ui` 文件，编译为 `main_window_ui.py` 模块（同步代码导入路径）。

3. 逻辑编写：新建`base_ui.py` 文件，导入编译后的 UI 模块，实现文件选择、模型调用、格式转换、视频帧处理等核心功能，通过信号与槽绑定 `det_image`、`det_video` 按钮事件。

4. 测试优化：运行 `python base_ui.py` 测试功能，确保图片/视频检测流畅，无界面阻塞、格式异常等问题。
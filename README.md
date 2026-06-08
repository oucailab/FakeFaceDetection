# FakeFaceDetection

FakeFaceDetection 是一个用于检测图片或视频中人脸伪造概率的深度学习项目。项目会从视频中抽取帧，检测画面中的人脸区域，再通过人脸伪造检测模型输出伪造概率。

本项目适用于 AI 换脸识别、视频内容审核、身份真实性辅助判断等场景。检测结果应作为辅助参考，不能作为唯一判定依据。

本仓库提供一个可直接用于测试的训练权重文件和10个用于视频检测的样例，可在release页下载。

## 项目功能

- 支持对 `mp4` 视频进行人脸伪造概率检测
- 支持按固定帧数抽样，降低长视频推理成本
- 自动检测视频帧中的人脸并裁剪人脸区域
- 输出视频级别的伪造概率分数
- 支持基于公开数据集进行评估
- 支持重新训练模型并保存训练权重

## 工作流程

1. 输入待检测视频文件
2. 从视频中抽取指定数量的视频帧
3. 对每帧进行人脸检测
4. 裁剪并归一化人脸图像
5. 将人脸图像输入检测模型
6. 汇总多帧预测结果
7. 输出最终伪造概率

示例输出：

```text
fakeness: 0.8735
```

`fakeness` 越接近 `1`，表示模型越倾向于判断该视频包含伪造人脸；越接近 `0`，表示越倾向于真实人脸。

## 项目结构

```text
FakeFaceDetection/
├── README.md
├── requirements.txt              # Python 依赖
├── build.sh                      # Docker 镜像构建脚本
├── exec.sh                       # Docker 容器启动脚本
├── data/                         # 数据集、测试视频或样例数据
├── output/                       # 训练输出、日志和模型权重
├── dockerfiles/
│   └── Dockerfile                # Docker 环境配置
└── src/
    ├── model.py                  # 检测模型结构
    ├── train.py                  # 模型训练入口
    ├── configs/                  # 训练配置
    ├── inference/                # 视频、图片和数据集推理代码
    ├── preprocess/               # 数据预处理与人脸裁剪代码
    └── utils/                    # 训练、增强、日志等工具函数
```

## 环境要求

推荐使用支持 CUDA 的 NVIDIA GPU 环境运行本项目。

主要依赖：

- Python 3.8
- PyTorch
- TorchVision
- OpenCV
- NumPy
- scikit-learn
- dlib
- retina-face
- efficientnet-pytorch

建议先创建 Python 3.8 环境，再安装依赖：

```bash
pip install -r requirements.txt
```

如果 PyTorch CUDA 版本安装失败，请根据本机 CUDA 版本到 PyTorch 官方安装说明中选择匹配命令。

## Docker 使用

构建镜像：

```bash
bash build.sh
```

启动容器前，请先将 `exec.sh` 中的 `/path/to/this/repository` 替换为本项目在你电脑上的绝对路径。

启动容器：

```bash
bash exec.sh
```

## 视频检测

准备一个待检测视频，例如：

```text
data/sample.mp4
```

运行视频推理：

```bash
python src/inference/inference_video.py -w output/weights/FFc23.tar -i data/sample.mp4 -n 32
```

参数说明：

| 参数 | 说明 |
| --- | --- |
| `-w` | 模型权重文件路径 |
| `-i` | 输入视频路径 |
| `-n` | 抽取的视频帧数量，默认值为 `32` |

示例输出：

```text
fakeness: 0.8735
```

会读取到这些文件。

### 完整训练流程

1. 下载 `FaceForensics++`真实视频

将训练视频放到：

```text
data/FaceForensics++/original_sequences/youtube/c23/videos/
```

同时确认 `train.json`、`val.json`、`test.json` 位于：

```text
data/FaceForensics++/
```

2. 确认人脸关键点模型存在

训练预处理需要使用：

```text
src/preprocess/shape_predictor_81_face_landmarks.dat
```

如果该文件不存在，从[此处](https://github.com/codeniko/shape_predictor_81_face_landmarks)下载好该模型文件并放到上述路径。

3. 提取视频帧和 dlib 人脸关键点

```bash
python src/preprocess/crop_dlib_ff.py -d Original -c c23
```

该步骤会从视频中抽帧，并生成：

```text
data/FaceForensics++/original_sequences/youtube/c23/frames/
data/FaceForensics++/original_sequences/youtube/c23/landmarks/
```

4. 提取 RetinaFace 检测框

```bash
python src/preprocess/crop_retina_ff.py -d Original -c c23
```

该步骤会生成：

```text
data/FaceForensics++/original_sequences/youtube/c23/retina/
```

5. 检查预处理结果

训练前请确认每个参与训练的视频目录下同时存在：

```text
frames/<video_id>/*.png
landmarks/<video_id>/*.npy
retina/<video_id>/*.npy
```

如果缺少 `landmarks/` 或 `retina/`，对应帧会在训练数据加载时被跳过。

6. 运行训练

默认配置文件：

```text
src/configs/sbi/base.json
```

运行训练：

```bash
python src/train.py src/configs/sbi/base.json -n fake_face_detection
```

训练结果会保存到 `output/` 目录，包含日志文件和模型权重文件。

输出目录示例：

```text
output/fake_face_detection_base_<时间戳>/
├── logs/
│   └── losses.logs
└── weights/
    └── *_val.tar
```

其中模型权重位于：

```text
output/fake_face_detection_base_<时间戳>/weights/
```

## 输出说明

| 输出字段 | 说明 |
| --- | --- |
| `fakeness` | 人脸被模型判断为伪造内容的概率分数 |
| `AUC` | 数据集评估时的分类性能指标 |




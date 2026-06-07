# FakeFaceDetection

FakeFaceDetection 是一个用于检测视频中人脸伪造概率的项目。用户输入一段 `mp4` 格式视频后，系统会自动抽取视频帧，识别人脸区域，并给出视频中人脸为伪造、换脸或深度伪造内容的概率评估。

本项目适用于深度伪造检测、视频内容审核、身份真实性辅助判断等场景。

## 项目功能

- 支持输入 `mp4` 格式视频
- 自动抽取视频关键帧
- 自动检测视频画面中的人脸
- 对检测到的人脸进行真假概率分析
- 输出视频级别的伪造概率结果
- 可用于辅助识别 AI 换脸、Deepfake、人脸合成等伪造内容

## 工作流程

1. 用户上传或指定一段 `mp4` 视频
2. 系统按固定间隔抽取视频帧
3. 对每一帧进行人脸检测与裁剪
4. 将人脸图像输入伪造检测模型
5. 汇总多帧检测结果
6. 输出最终判断结果，例如：

```text
Fake probability: 87.35%
Result: Likely Fake
```

## 项目结构

```text
FakeFaceDetection/
├── README.md
├── data/                 # 存放测试视频或样例数据
├── models/               # 存放训练好的检测模型
├── src/                  # 核心代码
│   ├── video_loader.py   # 视频读取与抽帧
│   ├── face_detector.py  # 人脸检测
│   ├── predictor.py      # 真假概率预测
│   └── main.py           # 程序入口
├── outputs/              # 检测结果输出
└── requirements.txt      # Python 依赖
```

## 安装方式

克隆仓库：

```bash
git clone https://github.com/your-username/FakeFaceDetection.git
cd FakeFaceDetection
```

安装依赖：

```bash
pip install -r requirements.txt
```

## 使用方法

运行检测程序：

```bash
python src/main.py --video data/sample.mp4
```

示例输出：

```text
Video: data/sample.mp4
Detected faces: 42
Fake probability: 87.35%
Result: Likely Fake
```

## 输出说明

| 输出字段 | 说明 |
| --- | --- |
| `Fake probability` | 视频中人脸为伪造内容的概率 |

默认判断规则：

| 概率范围 | 判断结果 |
| --- | --- |
| `0% - 40%` | Likely Real |
| `40% - 70%` | Uncertain |
| `70% - 100%` | Likely Fake |


## 注意事项

- 检测结果仅作为辅助判断，不应作为唯一结论
- 视频清晰度、压缩程度、光照、人脸角度都会影响检测准确率
- 如果视频中没有检测到人脸，系统无法给出有效的人脸伪造概率
- 建议结合多帧、多模型或人工复核提高可靠性

## 后续计划

- 增加 Web 上传界面
- 支持更多视频格式
- 增加检测结果可视化
- 支持逐帧伪造概率曲线
- 支持模型训练与微调

## License

This project is licensed under the MIT License.


---
title: Heygem技术架构
tags:
  - 面试
  - Heygem
categories:
  - 面试
date: 2026-03-05 14:25:00
typora-root-url: ../..
---

Heygem技术架构

## 📊 项目技术架构总结

### 🎯 **项目概述**

这是一个基于**深度学习**的**AI 数字人视频生成系统**，能够将静态视频与音频结合，生成逼真的数字人说话视频。项目采用 Python 开发，运行在 Windows 平台上。

------

### 🏗️ **整体架构分层**

#### **1. 应用层 (Application Layer)**

- **[`app.py`](javascript:void(0))** - 主入口文件
- **[`digital_human_streaming.py`](javascript:void(0))** - 数字人流式处理核心逻辑
- **[`rtmp_cached.py`](javascript:void(0))** - RTMP 推流器（带人脸特征缓存优化）
- **[`digital_human_test.py`](javascript:void(0))** - 测试脚本

#### **2. 服务层 (Service Layer)**

- [`service/trans_dh_service.py`](javascript:void(0))

   

  \- 数字人转换服务核心业务逻辑

  - 音频特征提取
  - 视频帧处理
  - 批量推理生成

#### **3. 核心算法层 (Core Algorithm Layer)**

##### **3.1 音频处理模块**

- **Wenet 语音识别引擎** ([`wenet/compute_ctc_att_bnf.py`](javascript:void(0)))
  - 使用 Conformer 编码器架构
  - 提取 CTC-Attention 瓶颈特征 (BNF)
  - 支持 FP16 半精度推理加速
  - 采样率：16kHz, 80 维 mel 频谱特征
- **音频分离模型** ([`audio-separator-models/`](javascript:void(0)))
  - RoFormer 架构
  - 人声/伴奏分离
  - 44.1kHz 采样率处理

##### **3.2 人脸检测与处理模块**

- **人脸检测** ([`face_detect_utils/face_detect.py`](javascript:void(0)))
  - SCRFD 模型 (500M/10G 版本)
  - ONNX Runtime 推理加速
  - 支持 GPU/CPU 模式
- **人脸关键点检测**
  - PFPFLD 模型 (68 点)
  - 用于精确定位人脸特征
- **人脸对齐与裁剪** ([`face_lib/face_detect_and_align/`](javascript:void(0)))
  - InsightFace 集成
- **人脸修复增强** ([`face_lib/face_restore/gfpgan_onnx/`](javascript:void(0)))
  - GFPGAN v1.4
  - 提升生成画质

##### **3.3 姿态估计模块**

- 头部姿态估计

   

  (

  `face_detect_utils/head_pose.py`

  )

  - 检测偏航角、俯仰角、翻滚角
  - 姿态阈值过滤 ([-70°,50°], [-100°,100°], [-70°,70°])

##### **3.4 数字人生成核心模型** ([`landmark2face_wy/`](javascript:void(0)))

- L2FaceAudioModel

   

  \- 音频驱动的人脸动画模型

  - **MappingNet**: 3DMM 系数→描述子映射
  - **WarpingNet**: 图像变形网络 (ADAINHourglass 架构)
  - **EditingNet**: 精细编辑网络

- 模型特性

  :

  - 输入尺寸：512×512
  - 支持动态融合 (blend_dynamic)
  - 超分能力 (chaofen_before=1 启用 GFPGAN)
  - 掩码融合策略
  - 半精度推理 (FP16)

#### **4. 数据处理层 (Data Processing Layer)**

- **[`preprocess_audio_and_3dmm.py`](javascript:void(0))** - 音频和 3DMM 特征预处理
  - 多进程并行处理
  - 人脸检测与裁剪
  - 3DMM 边界框计算
- **数据集类** ([`landmark2face_wy/data/`](javascript:void(0)))
  - FaceRealA3DMMExp 数据集
  - 音频 - 人脸配对数据加载

#### **5. 工具与基础设施层 (Utilities & Infrastructure)**

- **[`h_utils/`](javascript:void(0))** - 通用工具类
  - 自定义错误处理
  - 请求工具
  - 模型下载管理
- **[`y_utils/`](javascript:void(0))** - 项目专用工具
  - 配置管理 (GlobalConfig)
  - 日志系统
- **模型库封装** ([`model_lib/base_wrapper/`](javascript:void(0)))
  - 统一的模型接口封装

------

### 🔧 **技术栈**

| 类别             | 技术选型                                         |
| :--------------- | :----------------------------------------------- |
| **编程语言**     | Python 3.9 (内嵌于 [`py39/`](javascript:void(0)) |
| **深度学习框架** | PyTorch (支持 CUDA)                              |
| **推理加速**     | ONNX Runtime, TensorRT(可选)                     |
| **计算机视觉**   | OpenCV, InsightFace                              |
| **音频处理**     | Librosa, Wenet                                   |
| **流媒体**       | FFmpeg (RTMP 推流)                               |
| **界面**         | Gradio (可选)                                    |
| **部署环境**     | Windows, CUDA 11.x                               |

------

### 📦 **核心依赖模型**

1. **SCRFD** - 人脸检测 (ONNX)
2. **PFPFLD** - 人脸关键点 (ONNX)
3. **Wenet Conformer** - 语音特征提取 (PyTorch)
4. **RoFormer** - 音频分离 (PyTorch)
5. **GFPGAN** - 人脸修复 (ONNX)
6. **L2FaceAudio** - 数字人生成 (PyTorch 自定义)

------

### ⚙️ **工作流程**

```
音频输入 → 音频分离 → Wenet 特征提取 → 音频特征序列
                              ↓
视频输入 → 人脸检测 → 人脸对齐 → 3DMM 参数提取 → 参考图像
                              ↓
                    数字人模型推理 (L2FaceAudio)
                              ↓
                    人脸融合 + GFPGAN 增强
                              ↓
                    视频帧合成 → RTMP 推流/保存
```

------

### 🚀 **性能优化策略**

1. **模型层面**:
   - FP16 半精度推理
   - ONNX Runtime 加速
   - 批量推理 (batch processing)
2. **系统层面**:
   - 多进程数据处理 ([`multiprocessing.dummy`](javascript:void(0)))
   - 人脸特征缓存机制 ([`rtmp_cached.py`](javascript:void(0)))
   - 流式生成与推流并行
3. **内存管理**:
   - `torch.cuda.empty_cache()` 定期清理
   - 分块处理大音频文件

------

### 📝 **配置文件**

- **[`config/config.json`](javascript:void(0))** - 运行时配置
- **[`config/config.ini`](javascript:void(0))** - 初始化参数
- **[`开始.bat`](javascript:void(0))** - Windows 启动脚本 (设置环境变量、CUDA 路径等)

------

### 🎬 **应用场景**

- AI 虚拟主播生成
- 数字人视频定制
- 实时数字人直播
- 教育/培训视频制作

这是一个**工业级的数字人视频生成系统**,集成了当前主流的深度学习模型，具备完整的音视频处理能力和实时推流功能。
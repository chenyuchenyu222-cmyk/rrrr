# ASR语音识别服务
这是一个基于VOSK和VAD的完整语音识别服务系统，支持唤醒词检测、语音活动检测和实时语音识别。

## 环境安装
pip install -r requirements.txt -i https://pypi.doubanio.com/simple
如果出现cuda错误需要安装torch
pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu129

### rk3588安装
```shell
onnxruntime-gpu
https://elinux.org/Jetson_Zoo#ONNX_Runtime 下载1.19.0
pyaudio 报错
sudo apt update
sudo apt install portaudio19-dev
修复 libffi 库问题
sudo apt update
sudo apt install --reinstall libffi-dev libffi8 libasound2-dev libasound2 alsa-utils
sudo ldconfig
CTranslate2需要安装cpu版本
sudo apt-get install libomp-dev
whisper device 要选cpu



sherpa-onnx        1.12.13
librknnrt.so
https://github.com/airockchip/rknn-toolkit2/blob/v2.2.0/rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so
```

## 使用方法

### 添加权限
```shell
 chmod +x start.sh
 chmod +x stop.sh
```

### 执行脚本

sh start.sh
sh stop.sh

## Web状态监控页面

**注意**: Web状态页面现在可以直接在浏览器中打开使用，无需单独启动Web服务器。

1. 浏览器打开asr_service/data/asr_web_monitor.html
2. 输入框中输入服务器ws地址点击连接。
3. 点击连接查看最新状态


## 常见问题

### 系统状态运行中，但是唤醒词服务未启动。
１ 可能 robot-core-ai 未启动
２ 可能录音未找到设备，参考Log
### 无法唤醒，唤醒词服务未启动未找到dov设备或系统dov未设置为默认服务
1 设置dov为默认输入源
2 dov设备灯闪烁，重新插拔dov设备，先插入sub口，再插入串口。sudo lsusb  可查看设备
### 麦克风方向无数据
1 是否安装ch340驱动
检查驱动是否已加载：
lsmod | grep ch34
ls /dev/tty*
连接后出现 /dev/ttyCH341USB0
# sudo usermod -a -G dialout $USER  将当前用户添加到 dialout 组
# sudo chmod 666 /dev/ttyCH341USB0  临时更改设备权限

```shell
不传参数（使用配置文件中的默认串口号）
python test_serial.py
传递串口号参数
python test_serial.py /dev/ttyCH341USB2


orin串口已绑定至/dev/hx_MicDirection
ls -l /dev/hx*
```


设置音量
`chmod +x audio_setup.sh`
## 使用默认音量400%
`./audio_setup.sh`
## 使用自定义音量
`./audio_setup.sh 150%`



## whisper 模型下载地址
```shell
https://huggingface.co/collections/openai/whisper-release


##orin sherpa-onnx安装
pip install sherpa-onnx sherpa-onnx-bin

pip install sherpa-onnx==1.12.14+cuda -f https://k2-fsa.github.io/sherpa/onnx/cuda.html
python3 -c "import sherpa_onnx; print(sherpa_onnx.__version__)"


pip install sherpa-ncnn sherpa-ncnn-bin
```

## 配置文件说明，
config.env : windows启动配置文件
data/orin.config.env : orin启动配置文件，新orin环境只需要配置的ip.

DEFAULT_ONLINE_ASR_SERVICE # 在线的识别引擎目前只有DashScopeAsr
DEFAULT_LOCAL_ASR_SERVICE # 离线的识别引擎，以下4种，orin上配置NcnnZipformerAsr
    OnnxSenseVoiceAsr orin上onnx不可用，不劫持onnx
    NccnSenseVoiceAsr 
    NcnnZipformerAsr 
ASR_AUDIO_PATH=保存的音频路径
SQL_LIST_PATH=robot-ai.db文件路径
SAMPLE_RATE=16000 # 固定16k

KWS_MODEL_BASE # 唤醒词模型路径
KWS_MODEL_NAME # 唤醒词模型路径有多个子模型可选

DOV_MIC_PORT # 麦克风方向的串口名称
DOV_MIC_RECONNECT_INTERVAL # 麦克风方向的串口如果断开多久重连秒

ROBOT_CLIENT_URL # java工程接口
ROBOT_CLIENT_RECONNECT_INTERVAL # 重连时间秒

nccn 框架sense voice参数，orin平台用
NCCN_SENSE_VOICE_MODEL_DIR # 模型路径
NCCN_SENSE_VOICE_TOKENS # 分词文件
NCCN_SENSE_VOICE_NUM_THREADS
NCCN_SENSE_VOICE_USE_ITN
NCCN_SENSE_VOICE_DEBUG

onnx框架sense voice参数
ONNX_SENSE_VOICE_MODEL # 模型路径
ONNX_SENSE_VOICE_TOKENS # 分词文件
ONNX_SENSE_VOICE_HR_RULE_FSTS=  # 配置参数暂时为空
ONNX_SENSE_VOICE_HR_LEXICON=  # 配置参数暂时为空

silero_vad分词配置
SILERO_VAD_MODEL= # 模型路径
SILERO_VAD_SAMPLE_RATE # 固定16000
SILERO_VAD_BUFFER_SIZE_IN_SECONDS  # 缓存音频长度

在线语音识别
DASHSCOPE_ASR_MODEL # 模型名称paraformer-realtime-v2
DASHSCOPE_ASR_VOCABULARY_ID #热词VOCABULARY_ID
DASHSCOPE_ASR_SAMPLE_RATE=16000 #固定16000

NCNN Zipformer Configuration
NCNN_ZIPFORMER_MODEL_BASE_PATH= # 模型路径
NCNN_ZIPFORMER_NUM_THREADS=4   # 推理线程数量
NCNN_ZIPFORMER_ENABLE_ENDPOINT_DETECTION=true  #启动模型自带端点识别，否则无法分句。
NCNN_ZIPFORMER_RULE1_MIN_TRAILING_SILENCE=2.0
NCNN_ZIPFORMER_RULE2_MIN_TRAILING_SILENCE=1.0
NCNN_ZIPFORMER_RULE3_MIN_UTTERANCE_LENGTH=300.0
NCNN_ZIPFORMER_DECODING_METHOD=modified_beam_search  # 多路径搜索，可以提交准确率
NCNN_ZIPFORMER_NUM_ACTIVE_PATHS=8  # 热词路径数
NCNN_ZIPFORMER_HOTWORDS_SCORE=1.5   # 热词分数


SPEAKER_RECOGNITION_ENABLE=False  # Ture False是否启用说话人识别
SPEAKER_RECOGNITION_MODEL=# 模型路径
SPEAKER_RECOGNITION_THRESHOLD=0.5 # 大于此值才发送

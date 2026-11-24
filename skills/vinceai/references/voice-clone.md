# 授权音色配音

不出镜、只有脚本或需要讲解旁白时，可以把授权的克隆音色接到 vinceAI。已有批准的音轨时直接进入剪辑和字幕对齐。复用已获授权的音色配置前，确认当前项目仍在授权范围内。

## 用 Qwen3-TTS 本地合成

- 准备 Qwen3-TTS Python 环境、FFmpeg 和已下载的 [Qwen3-TTS-12Hz-0.6B-Base](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-0.6B-Base)。下面示例走支持 BF16 的 CUDA 显卡和 SDPA。安装与替代运行方式见 [Qwen3-TTS 官方文档](https://github.com/QwenLM/Qwen3-TTS)。
- 准备一段本人或已授权、清楚且无混入其他说话人的干声，再准备逐字一致的 UTF-8 转写。把干声、转写和脚本保存在本项目约定的私有工作区。先合成一小段，核对音色、普通话读法和停连。
- 将下方示例保存为 `synthesize.py`，给出本地模型目录、参考音频、参考转写、口播稿和输出文件。口播稿较长时按完整语义切成数句，保持同一个音色提示，逐段检查并拼接。品牌、数值和量词可在合成稿里改成正确读音，屏幕字幕保留原写法。

```python
import sys
from pathlib import Path

import soundfile
import torch
from qwen_tts import Qwen3TTSModel

model_dir, ref_audio, ref_text, script, output = (
    Path(arg).expanduser().resolve() for arg in sys.argv[1:]
)
if output.exists():
    raise FileExistsError(output)

model = Qwen3TTSModel.from_pretrained(
    str(model_dir),
    device_map="cuda:0",
    dtype=torch.bfloat16,
    attn_implementation="sdpa",
)
with torch.inference_mode():
    voice = model.create_voice_clone_prompt(
        ref_audio=str(ref_audio),
        ref_text=ref_text.read_text(encoding="utf-8-sig").strip(),
        x_vector_only_mode=False,
    )
    waves, sample_rate = model.generate_voice_clone(
        text=script.read_text(encoding="utf-8-sig").strip(),
        language="Chinese",
        voice_clone_prompt=voice,
    )
soundfile.write(output, waves[0], sample_rate, subtype="PCM_16")
```

```shell
python synthesize.py ./model ./reference.wav ./reference.txt ./script.txt ./narration-raw.wav
ffmpeg -n -i narration-raw.wav -af atempo=1.1 -ar 48000 -c:a pcm_s24le narration.wav
```

视频工程统一使用最终配音做字幕时间轴。按用户确认的语速使用保留音高的变速，已调速的音轨直接接入。试听检查爆音、重读和句尾，按听感调人声增益并保留峰值余量。记录模型、已应用的语速和试听范围。同一段口播的多个画幅共用 WAV 和字幕时间轴，并按各画幅检查断行和安全区。

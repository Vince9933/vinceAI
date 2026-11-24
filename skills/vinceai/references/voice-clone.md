# 授权音色配音

只有脚本或需要克隆旁白时，使用本人或已授权的参考音色。有已确认的旁白时先核对文案、语速与增益，再接入剪辑。参考录音、逐字转写和音色配置留在授权范围内的工作区。

## 合成自然口播

1. 参考干声选清楚、安静、语气贴合当期内容的一段，听得见原说话人的停连、轻重和换气。给参考录音配逐字一致的 UTF-8 转写，保留实际念出来的字。沿用已确认的音色提示，试听音色和讲话方式。
2. 把口播稿整理成连续的完整语义段。让设问和回应、观点和解释、数字和量词尽量留在同一段。用标点和段落表达语气与停顿。字幕按自己的字符限制另行分屏。发现错读或语气跳变时，优先重生成受影响的完整语义段，模型漏词或截尾时再按句意拆短。
3. 合成稿和字幕稿分别核对。把容易读错的人名、品牌、日期、金额、百分比和单位写成已确认的口读形式。合成后逐处听核，字幕保留约定的书写形式。
4. 听单段也听相邻段。根据设问、转折、句尾和换气收放停顿。保留有表达作用的尾音、呼吸和停连，在实际接缝试听首尾是否连贯。

## 用 Qwen3-TTS 本地合成

准备 Qwen3-TTS Python 环境、FFmpeg 和 [支持参考音频的 Base 模型](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-0.6B-Base)。示例使用支持 BF16 的 CUDA 显卡和 SDPA，一次合成一个完整语义段。批量制作可复用同一个 `voice_clone_prompt`，逐段听核后拼接干声。安装和接口见 [Qwen3-TTS 官方文档](https://github.com/QwenLM/Qwen3-TTS)。

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
soundfile.write(output, waves[0], sample_rate, subtype="FLOAT")
```

```shell
python synthesize.py ./model ./reference.wav ./reference.txt ./script.txt ./narration-raw.wav
```

## 变速、增益与试听

- 口播定稿后先做 **1.1 倍**保音高变速。按变速后实际峰值留余量，再以浮点精度对人声施加 **+7 dB**。先测电平，需要时在 +7 dB 前调低基础电平，使限幅器只保护偶发尖峰。记录基础调整、增益级和最终实测电平。已经处理过的音轨只补缺少的步骤。
- 下方命令先检查经过变速、重采样的干声。以成品峰值上限 −1.5 dBTP、计划增益 +7 dB、额外余量 1 dB 为例，变速后干声峰值的目标是 −9.5 dBTP。实测为 −6.5 dBTP 时，可在 `volume=7dB` 前加入 `volume=-3dB:precision=double`，把峰值降到目标。限幅器关闭自动补偿，输出后复测真峰值。若能听出字头变钝、尾音被压或音量起伏，检查原段和增益前余量。
- 试听新旧配音时，用同一段文案、相同播放响度的纯人声片段对比，听重音、音色、停连和接缝。匹配响度只用于试听副本。用户只要人声时输出独立 WAV，跳过 BGM。做成片时配乐独立混音，以最终剪好的人声对齐字幕和画面。

```shell
ffmpeg -hide_banner -i narration-raw.wav -af "atempo=1.1,aresample=48000,ebur128=peak=true" -f null -
ffmpeg -n -i narration-raw.wav -af "atempo=1.1,aresample=48000,volume=7dB:precision=double,alimiter=limit=0.84:attack=5:release=68:level=0:latency=1" -c:a pcm_s24le narration.wav
ffmpeg -hide_banner -i narration.wav -af "ebur128=peak=true" -f null -
```

记录参考音色、所用模型、实际处理参数和试听范围。多个画幅共用人声和字幕时间轴，分别检查字幕断行与安全区。

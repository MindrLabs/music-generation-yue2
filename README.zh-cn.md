[English](README.md) · [한국어](README.ko.md) · [中文](README.zh-cn.md)

# music-generation-yue2

这个服务接收风格提示词和歌词，返回一首 48 kHz 立体声歌曲，以及演唱这首歌所依据的乐谱。

## 模型

[`m-a-p/YuE2-3B`](https://huggingface.co/m-a-p/YuE2-3B) 是一个开放的音乐生成模型，能把歌词和风格提示词变成一首带人声和伴奏的完整歌曲。
它可以在生成音频之前先写出 ABC 乐谱。

## 演示

![在 gradio 界面中输入风格和歌词，运行后得到乐谱和歌曲](docs/images/music-generation-yue2.gif)

## 快速开始

需要 [model-compose](https://github.com/hanyeol/model-compose) 0.4.111 或更高版本。

用 [uv](https://docs.astral.sh/uv/) 安装：

```bash
uv pip install model-compose
```

或者用 pip 安装：

```bash
pip install model-compose
```

克隆本仓库：

```bash
git clone https://github.com/MindrLabs/music-generation-yue2
cd music-generation-yue2
```

运行：

```bash
model-compose up
```

gradio 界面在 `http://localhost:8081` 打开，HTTP API 在 `http://localhost:8080/api`。
可以用 `SERVER_PORT` 和 `PORT` 分别修改端口。

| 首次运行 | 发生的事 |
| :---: | --- |
| 虚拟环境 | model-compose 在 `.venv/composer` 创建环境，并安装 `torch==2.10.0` 和 `yue2-infer` 0.1.6 wheel |
| 检查点 | 下载 7.30 GB 的 `m-a-p/YuE2-3B`，以及组件默认使用的解码器 `m-a-p/YuE2-Vae`（0.53 GB） |

`DEVICE` 默认为 `cuda`。
在 MacBook 上，需要给 `composer` 组件加上 `cpu_offload: vae`，并用 `DEVICE=mps model-compose up` 启动。
性能表中的 MacBook M1 一行就是用这个配置测量的。
由于 `max_concurrent_count: 1`，第二个请求会等到第一个请求结束后才开始。

## 性能

| 设备 | 运行时 | 条目数 | 冷启动 (s) | 单项首个输出 (s) | 单项端到端 (s) | 总计 (s) | 输出 RTF | 峰值 VRAM (GiB) | 峰值 RSS (GiB) | 相对 RTX 4090 | 准确度 | 判定 |
| :---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RTX 4090 | model-compose + pytorch 2.10.0+cu128 | 20 | 10.9 | 23.7 | 23.9 | 492 | 0.361 | 9.47 | 15.5 | 1.00× | — | 基准 |
| DGX Spark | model-compose + pytorch 2.10.0+cu130 | 20 | 14.4 | 96.0 | 97.1 | 2029 | 1.46 | 9.80 | 56.8 | 4.06× | — | 通过 |
| MacBook M1 | model-compose + pytorch 2.10.0 | 1 | 9.01 | 378 | 379 | 379 | 13.0 | — | 4.33 | 15.8× | — | 通过 |

单项数值为中位数，总计为从就绪到最后一个条目的时间。
比值为各行单项端到端时间除以最快一行的值。
各行运行的条目数不同，因此总计不能在行之间比较。
速度差异中除硬件外，还混有各行不同的运行时的影响。
VRAM 为空表示运行器无法读取该设备的加速器，并不代表未使用显存。
DGX Spark, MacBook M1 的处理器与加速器各自共用同一内存池，因此其 VRAM 与 RSS 峰值会把同一部分内存计算两次。

> [!NOTE]
> RTX 4090 的输出 RTF 为 0.361，意味着一首 1 分钟的歌大约 22 秒就能生成，比播放还快。
> DGX Spark 的 1.46 意味着一首 1 分钟的歌大约需要 88 秒，生成速度慢于播放速度。
> MacBook M1 的 13.0 意味着一首 1 分钟的歌大约需要 13 分钟，而且这只是 1 个条目的测量值。

### 测量条件

| 项目 | 值 |
| :---: | --- |
| 数据集 | 为本基准编写的 20 个条目 |
| 指标 | 用领域工具测量设备之间的距离 |
| 检查点 | [m-a-p/YuE2-3B@14fc6c6f146441b1dd6363fcb2e01e82a6914cb7](https://huggingface.co/m-a-p/YuE2-3B/tree/14fc6c6f146441b1dd6363fcb2e01e82a6914cb7) |
| 准确度 | 设备间距离 |
| 公开分数 | 未与公开分数比较，只检查设备之间是否一致 |

## 限制

准确度仅检查了设备之间是否一致，未与公开分数比较。

## 许可证

| 适用对象 | 许可证 | 商业使用 |
| :---: | --- | :---: |
| 本服务 | [`LICENSE`](LICENSE) | ✓ |
| `m-a-p/YuE2-3B` 权重 | [`MODEL_LICENSE`](https://github.com/multimodal-art-projection/YuE/blob/main/MODEL_LICENSE) | ✗ |
| 个人创作者用这些权重生成的歌曲 | [`MODEL_LICENSE`](https://github.com/multimodal-art-projection/YuE/blob/main/MODEL_LICENSE) | ✓ |

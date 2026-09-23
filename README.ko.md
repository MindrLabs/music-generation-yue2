[English](README.md) · [한국어](README.ko.md) · [中文](README.zh-cn.md)

# music-generation-yue2

스타일 프롬프트와 가사를 넣으면 48 kHz 스테레오 노래와 그 노래를 부른 악보를 함께 돌려주는 서비스입니다.

## 모델

[`m-a-p/YuE2-3B`](https://huggingface.co/m-a-p/YuE2-3B)는 가사와 스타일 프롬프트로 보컬과 반주가 있는 노래 한 곡을 만드는 공개 음악 생성 모델입니다.
오디오를 만들기 전에 ABC 악보를 먼저 쓸 수 있습니다.

## 데모

![gradio 화면에서 스타일과 가사를 넣고 실행해 악보와 노래를 받는 모습](docs/images/music-generation-yue2.gif)

## 빠른 시작

[model-compose](https://github.com/hanyeol/model-compose) 0.4.111 이상이 필요합니다.

[uv](https://docs.astral.sh/uv/)로 설치:

```bash
uv pip install model-compose
```

또는 pip로 설치:

```bash
pip install model-compose
```

이 저장소 클론:

```bash
git clone https://github.com/MindrLabs/music-generation-yue2
cd music-generation-yue2
```

실행:

```bash
model-compose up
```

gradio 화면은 `http://localhost:8081`, HTTP API는 `http://localhost:8080/api`에서 열립니다.
`SERVER_PORT`와 `PORT`로 각 포트를 바꿀 수 있습니다.

| 첫 실행 | 하는 일 |
| :---: | --- |
| 가상 환경 | model-compose가 `.venv/composer`에 환경을 만들고 `torch==2.10.0`과 `yue2-infer` 0.1.6 휠을 설치합니다 |
| 체크포인트 | `m-a-p/YuE2-3B` 7.30 GB와 컴포넌트가 기본으로 쓰는 디코더 `m-a-p/YuE2-Vae` 0.53 GB를 내려받습니다 |

`DEVICE`의 기본값은 `cuda`입니다.
MacBook에서는 `composer` 컴포넌트에 `cpu_offload: vae`를 추가하고 `DEVICE=mps model-compose up`으로 실행합니다.
성능 표의 MacBook M1 행은 이 설정으로 측정했습니다.
`max_concurrent_count: 1`이므로 두 번째 요청은 첫 요청이 끝날 때까지 기다립니다.

## 성능

| 기기 | 런타임 | 항목 수 | 콜드 스타트 (s) | 항목당 첫 출력 (s) | 항목당 전체 (s) | 합계 (s) | 출력 RTF | 최대 VRAM (GiB) | 최대 RSS (GiB) | RTX 4090 대비 | 정확도 | 판정 |
| :---: | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| RTX 4090 | model-compose + pytorch 2.10.0+cu128 | 20 | 10.9 | 23.7 | 23.9 | 492 | 0.361 | 9.47 | 15.5 | 1.00× | — | 기준 |
| DGX Spark | model-compose + pytorch 2.10.0+cu130 | 20 | 14.4 | 96.0 | 97.1 | 2029 | 1.46 | 9.80 | 56.8 | 4.06× | — | 통과 |
| MacBook M1 | model-compose + pytorch 2.10.0 | 1 | 9.01 | 378 | 379 | 379 | 13.0 | — | 4.33 | 15.8× | — | 통과 |

항목당 값은 중앙값이며 합계는 준비 완료 시점부터 마지막 항목까지의 시간입니다.
비율은 각 행의 항목당 전체 시간을 가장 빠른 행의 값으로 나눈 것입니다.
행마다 실행한 항목 수가 달라 합계는 행끼리 비교할 수 없습니다.
속도 차이에는 행마다 다른 런타임의 영향이 하드웨어 차이와 함께 섞여 있습니다.
빈 VRAM 칸은 러너가 그 기기의 가속기를 읽지 못했다는 뜻입니다. 사용량이 0이었다는 뜻은 아닙니다.
DGX Spark, MacBook M1에서는 프로세서와 가속기가 메모리 하나를 함께 쓰므로 VRAM과 RSS 최대값에 같은 바이트가 두 번 들어갑니다.

> [!NOTE]
> RTX 4090의 출력 RTF 0.361은 1분짜리 노래를 약 22초에 만든다는 뜻이라 재생 시간보다 빨리 나옵니다.
> DGX Spark의 1.46은 1분짜리 노래에 약 88초가 걸린다는 뜻이라 노래가 재생 시간보다 늦게 나옵니다.
> MacBook M1의 13.0은 1분짜리 노래에 약 13분이 걸린다는 뜻이고, 항목 1개만 잰 값입니다.

### 측정 조건

| 항목 | 값 |
| :---: | --- |
| 데이터셋 | 이 벤치마크를 위해 쓴 항목 20개 |
| 지표 | 도메인 도구로 기기 간 거리를 측정 |
| 체크포인트 | [m-a-p/YuE2-3B@14fc6c6f146441b1dd6363fcb2e01e82a6914cb7](https://huggingface.co/m-a-p/YuE2-3B/tree/14fc6c6f146441b1dd6363fcb2e01e82a6914cb7) |
| 정확도 | 기기 간 거리 |
| 공개 점수 | 공개 점수와 비교하지 않았으며 기기 간 일치만 확인 |

## 한계

정확도는 기기 간 일치로만 확인했으며 공개 점수와는 비교하지 않았습니다.

## 라이선스

| 대상 | 라이선스 | 상업적 이용 |
| :---: | --- | :---: |
| 이 서비스 | [`LICENSE`](LICENSE) | ✓ |
| `m-a-p/YuE2-3B` 가중치 | [`MODEL_LICENSE`](https://github.com/multimodal-art-projection/YuE/blob/main/MODEL_LICENSE) | ✗ |
| 개인 창작자가 가중치로 만든 노래 | [`MODEL_LICENSE`](https://github.com/multimodal-art-projection/YuE/blob/main/MODEL_LICENSE) | ✓ |

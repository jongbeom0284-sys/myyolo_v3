# YOLOv3 Docker Lab

**이미지 URL을 입력하면 YOLOv3 객체 탐지를 실행하고, 결과 이미지를 호스트 폴더에 저장하는 Docker 실습입니다.**

국민대학교 차량소프트웨어엔지니어링 과제 · 윤종범
`Docker` · `Linux / WSL` · `Bash` · `Darknet / YOLOv3`

## 구현한 실행 흐름

```text
이미지 URL → wget으로 input.jpg 저장 → Darknet YOLOv3 추론
                                         ↓
                                /output/predictions.jpg
                                         ↓ volume mount
                                   사용자 PC의 결과 폴더
```

| 파일 | 역할 |
|---|---|
| [`Dockerfile`](Dockerfile) | Ubuntu 20.04에 빌드 도구 설치 → 공개 Darknet clone·빌드 → YOLOv3 가중치 준비 → 실행 스크립트 등록 |
| [`entrypoint.sh`](entrypoint.sh) | 첫 번째 인자로 받은 URL의 이미지를 내려받고 탐지 결과를 `/output`으로 이동 |

기존 YOLOv3 모델을 새로 만든 프로젝트가 아니라, **모델 실행 환경과 입출력을 컨테이너로 묶는 과제**입니다.

## 빌드와 실행

Docker 엔진이 실행 중인 Linux / WSL 셸 기준입니다. 이미지 빌드에는 인터넷 연결과 가중치 다운로드가 필요합니다.

```bash
docker build -t myyolo-v3:coursework .
mkdir -p output
docker run --rm -v "$(pwd)/output:/output" myyolo-v3:coursework "<image-url>"
```

정상적으로 실행되면 `output/predictions.jpg`에서 탐지 이미지를 확인할 수 있습니다. Windows PowerShell에서는 bind mount의 호스트 경로를 해당 셸의 절대 경로로 바꿔야 합니다.

## 과제에서 배운 점

- **실행 환경의 코드화**: `FROM`, `RUN`, `COPY`, `ENTRYPOINT`로 설치부터 실행까지의 흐름을 Dockerfile에 담았습니다.
- **Windows와 Linux 줄바꿈 차이**: 당시 보고서에 CRLF/LF 차이와 실행 오류를 정리했고, Dockerfile에 `sed -i 's/\r$//' entrypoint.sh` 처리를 넣었습니다. 실행 오류에는 다른 원인도 있으므로 모든 `exec format error`가 줄바꿈 때문이라고 일반화하지 않습니다.
- **컨테이너와 호스트 연결**: `/output`에 volume mount를 걸어 컨테이너 안에서 생성한 결과를 호스트에 남깁니다.
- **역할 분리**: Dockerfile은 실행 환경, entrypoint는 입력 이미지와 추론 실행 흐름을 담당합니다.

## 작업 기록과 확인 범위

WSL에 남아 있던 `myyolo_v3` 작업본과 이 저장소의 Dockerfile·실행 스크립트가 같은 내용임을 확인했습니다. 이번 정리에서는 README와 파일 관리 설정을 보강했으며 모델이나 추론 코드를 새로 개발한 것으로 표현하지 않습니다.

실행 스크립트의 Bash 구문을 검사했습니다. 정리 시점에는 Docker 엔진이 실행 중이 아니어서 이미지 재빌드·추론을 다시 수행하지 않았습니다. URL 입력 검사·다운로드 실패 처리, upstream commit·가중치 checksum 고정은 현재 실습 코드에 없습니다.

기반은 [pjreddie/Darknet](https://github.com/pjreddie/darknet)입니다. Darknet 전체 소스와 가중치는 저장소에 복사하지 않았으며, Dockerfile이 원 출처에서 받도록 되어 있습니다. 관련 수업의 로컬 LLM·빌드 실습은 [차량소프트웨어엔지니어링 실습 기록](https://github.com/jongbeom0284-sys/Vehicle-Software-Engineering-Labs)에 연결했습니다.

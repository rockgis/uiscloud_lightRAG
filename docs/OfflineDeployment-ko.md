# LightRAG 오프라인 배포 가이드

이 가이드는 인터넷 접근이 제한되거나 불가능한 환경에서 LightRAG를 배포하기 위한 종합 지침을 제공합니다.

Docker를 사용하여 LightRAG를 배포하는 경우 이 문서를 참조할 필요가 없습니다. LightRAG Docker 이미지는 오프라인 작동을 위해 사전 설정되어 있기 때문입니다.

> `transformers`, `torch` 또는 `cuda`가 필요한 소프트웨어 패키지는 오프라인 의존성 그룹에 포함되지 않습니다. 따라서 Docling 같은 문서 추출 도구나 HuggingFace, LMDeploy 같은 로컬 LLM 모델은 오프라인 설치 지원 범위 밖입니다. Docling은 독립 서비스로 분리되어 배포될 예정입니다.

## 목차

- [개요](#개요)
- [빠른 시작](#빠른-시작)
- [계층적 의존성](#계층적-의존성)
- [Tiktoken 캐시 관리](#tiktoken-캐시-관리)
- [완전한 오프라인 배포 워크플로우](#완전한-오프라인-배포-워크플로우)
- [문제 해결](#문제-해결)

## 개요

LightRAG는 파일 유형 및 설정에 따라 선택적 기능을 위해 동적 패키지 설치(`pipmaster`)를 사용합니다. 오프라인 환경에서는 이러한 동적 설치가 실패합니다. 이 가이드는 필요한 모든 의존성과 캐시 파일을 사전 설치하는 방법을 보여줍니다.

### 동적으로 설치되는 항목

LightRAG는 다음을 위해 패키지를 동적으로 설치합니다:

- **스토리지 백엔드**: `redis`, `neo4j`, `pymilvus`, `pymongo`, `asyncpg`, `qdrant-client`
- **LLM 제공자**: `openai`, `anthropic`, `ollama`, `zhipuai`, `aioboto3`, `voyageai`, `llama-index`, `lmdeploy`, `transformers`, `torch`
- **Tiktoken 모델**: OpenAI CDN에서 다운로드되는 BPE 인코딩 모델

**참고**: 문서 처리 의존성(`pypdf`, `python-docx`, `python-pptx`, `openpyxl`)은 현재 `api` extras 그룹에 사전 설치되어 있으며, 더 이상 동적 설치가 필요하지 않습니다.

## 빠른 시작

### 옵션 1: pip와 오프라인 Extras 사용

```bash
# 온라인 환경: 모든 오프라인 의존성 설치
pip install lightrag-hku[offline]

# tiktoken 캐시 다운로드
lightrag-download-cache

# 오프라인 패키지 생성
pip download lightrag-hku[offline] -d ./offline-packages
tar -czf lightrag-offline.tar.gz ./offline-packages ~/.tiktoken_cache

# 오프라인 서버로 전송
scp lightrag-offline.tar.gz user@offline-server:/path/to/

# 오프라인 환경: 설치
tar -xzf lightrag-offline.tar.gz
pip install --no-index --find-links=./offline-packages lightrag-hku[offline]
export TIKTOKEN_CACHE_DIR=~/.tiktoken_cache
```

### 옵션 2: 요구사항 파일 사용

```bash
# 온라인 환경: 패키지 다운로드
pip download -r requirements-offline.txt -d ./packages

# 오프라인 서버로 전송
tar -czf packages.tar.gz ./packages
scp packages.tar.gz user@offline-server:/path/to/

# 오프라인 환경: 설치
tar -xzf packages.tar.gz
pip install --no-index --find-links=./packages -r requirements-offline.txt
```

## 계층적 의존성

LightRAG는 다양한 사용 사례를 위한 유연한 의존성 그룹을 제공합니다:

### 사용 가능한 의존성 그룹

| 그룹 | 설명 | 사용 사례 |
| ----- | ----------- | -------- |
| `api` | API 서버 + 문서 처리 | PDF, DOCX, PPTX, XLSX 지원 FastAPI 서버 |
| `offline-storage` | 스토리지 백엔드 | Redis, Neo4j, MongoDB, PostgreSQL 등 |
| `offline-llm` | LLM 제공자 | OpenAI, Anthropic, Ollama 등 |
| `offline` | 완전한 오프라인 패키지 | API + 스토리지 + LLM (모든 기능) |

**참고**: 문서 처리(PDF, DOCX, PPTX, XLSX)는 더 나은 통합을 위해 `api` extras 그룹에 포함됩니다. 이전 `offline-docs` 그룹은 `api`로 통합되었습니다.

### 설치 예제

```bash
# 문서 처리 포함 API 설치
pip install lightrag-hku[api]

# API 및 스토리지 백엔드 설치
pip install lightrag-hku[api,offline-storage]

# 모든 오프라인 의존성 설치 (오프라인 배포에 권장)
pip install lightrag-hku[offline]
```

### 개별 요구사항 파일 사용

```bash
# 스토리지 백엔드만
pip install -r requirements-offline-storage.txt

# LLM 제공자만
pip install -r requirements-offline-llm.txt

# 모든 오프라인 의존성
pip install -r requirements-offline.txt
```

## Tiktoken 캐시 관리

Tiktoken은 처음 사용 시 BPE 인코딩 모델을 다운로드합니다. 오프라인 환경에서는 이러한 모델을 사전 다운로드해야 합니다.

### CLI 명령 사용

LightRAG 설치 후 내장 명령을 사용하세요:

```bash
# 기본 위치에 다운로드 (출력에서 정확한 경로 확인)
lightrag-download-cache

# 특정 디렉토리에 다운로드
lightrag-download-cache --cache-dir ./tiktoken_cache

# 특정 모델만 다운로드
lightrag-download-cache --models gpt-4o-mini gpt-4
```

### 기본 다운로드 모델

- `gpt-4o-mini` (LightRAG 기본값)
- `gpt-4o`
- `gpt-4`
- `gpt-3.5-turbo`
- `text-embedding-ada-002`
- `text-embedding-3-small`
- `text-embedding-3-large`

### 오프라인 환경에서 캐시 위치 설정

```bash
# 옵션 1: 환경 변수 (임시)
export TIKTOKEN_CACHE_DIR=/path/to/tiktoken_cache

# 옵션 2: ~/.bashrc 또는 ~/.zshrc에 추가 (영구)
echo 'export TIKTOKEN_CACHE_DIR=~/.tiktoken_cache' >> ~/.bashrc
source ~/.bashrc

# 옵션 3: 기본 위치에 복사
cp -r /path/to/tiktoken_cache ~/.tiktoken_cache/
```

## 완전한 오프라인 배포 워크플로우

### 1단계: 온라인 환경에서 준비

```bash
# 1. 오프라인 의존성을 포함한 LightRAG 설치
pip install lightrag-hku[offline]

# 2. tiktoken 캐시 다운로드
lightrag-download-cache --cache-dir ./offline_cache/tiktoken

# 3. 모든 Python 패키지 다운로드
pip download lightrag-hku[offline] -d ./offline_cache/packages

# 4. 전송용 아카이브 생성
tar -czf lightrag-offline-complete.tar.gz ./offline_cache

# 5. 내용 확인
tar -tzf lightrag-offline-complete.tar.gz | head -20
```

### 2단계: 오프라인 환경으로 전송

```bash
# scp 사용
scp lightrag-offline-complete.tar.gz user@offline-server:/tmp/

# 또는 USB/물리적 미디어 사용
# lightrag-offline-complete.tar.gz를 USB 드라이브에 복사
```

### 3단계: 오프라인 환경에서 설치

```bash
# 1. 아카이브 압축 해제
cd /tmp
tar -xzf lightrag-offline-complete.tar.gz

# 2. Python 패키지 설치
pip install --no-index \
    --find-links=/tmp/offline_cache/packages \
    lightrag-hku[offline]

# 3. tiktoken 캐시 설정
mkdir -p ~/.tiktoken_cache
cp -r /tmp/offline_cache/tiktoken/* ~/.tiktoken_cache/
export TIKTOKEN_CACHE_DIR=~/.tiktoken_cache

# 4. 영구적용을 위해 셸 프로파일에 추가
echo 'export TIKTOKEN_CACHE_DIR=~/.tiktoken_cache' >> ~/.bashrc
```

### 4단계: 설치 확인

```bash
# Python 임포트 테스트
python -c "from lightrag import LightRAG; print('✓ LightRAG 임포트 성공')"

# tiktoken 테스트
python -c "from lightrag.utils import TiktokenTokenizer; t = TiktokenTokenizer(); print('✓ Tiktoken 작동 중')"

# 선택적 의존성 테스트 (설치된 경우)
python -c "import docling; print('✓ Docling 사용 가능')"
python -c "import redis; print('✓ Redis 사용 가능')"
```

## 문제 해결

### 문제: 네트워크 오류로 Tiktoken 실패

**증상**: `Unable to load tokenizer for model gpt-4o-mini`

**해결책**:
```bash
# TIKTOKEN_CACHE_DIR 설정 확인
echo $TIKTOKEN_CACHE_DIR

# 캐시 파일 존재 여부 확인
ls -la ~/.tiktoken_cache/

# 비어 있으면 온라인 환경에서 먼저 캐시를 다운로드해야 합니다
```

### 문제: 동적 패키지 설치 실패

**증상**: `Error installing package xxx`

**해결책**:
```bash
# 필요한 특정 패키지를 사전 설치하세요
# 문서 처리 포함 API:
pip install lightrag-hku[api]

# 스토리지 백엔드:
pip install lightrag-hku[offline-storage]

# LLM 제공자:
pip install lightrag-hku[offline-llm]
```

### 문제: 런타임에 의존성 누락

**증상**: `ModuleNotFoundError: No module named 'xxx'`

**해결책**:
```bash
# 설치된 항목 확인
pip list | grep -i xxx

# 누락된 구성 요소 설치
pip install lightrag-hku[offline]  # 모든 오프라인 의존성 설치
```

### 문제: tiktoken 캐시 권한 거부

**증상**: `PermissionError: [Errno 13] Permission denied`

**해결책**:
```bash
# 캐시 디렉토리의 올바른 권한 확인
chmod 755 ~/.tiktoken_cache
chmod 644 ~/.tiktoken_cache/*

# 또는 사용자 쓰기 가능한 디렉토리 사용
export TIKTOKEN_CACHE_DIR=~/my_tiktoken_cache
mkdir -p ~/my_tiktoken_cache
```

## 모범 사례

1. **온라인 환경에서 먼저 테스트**: 오프라인으로 전환하기 전에 항상 온라인 환경에서 완전한 설정을 테스트하세요.

2. **캐시 최신 상태 유지**: 새 모델이 출시될 때 오프라인 캐시를 주기적으로 업데이트하세요.

3. **설정 문서화**: 실제로 필요한 선택적 의존성을 메모해 두세요.

4. **버전 고정**: 프로덕션에서 특정 버전 고정을 고려하세요:
   ```bash
   pip freeze > requirements-production.txt
   ```

5. **최소 설치**: 필요한 것만 설치하세요:
   ```bash
   # 문서 처리 포함 API만 필요한 경우
   pip install lightrag-hku[api]
   # 그런 다음 특정 LLM 수동 추가: pip install openai
   ```

## 추가 자료

- [LightRAG GitHub 저장소](https://github.com/HKUDS/LightRAG)
- [Docker 배포 가이드](./DockerDeployment-ko.md)
- [API 서버 문서](./LightRAG-API-Server-ko.md)

## 지원

이 가이드에서 다루지 않은 문제가 발생하면:

1. [GitHub Issues](https://github.com/HKUDS/LightRAG/issues)를 확인하세요
2. [프로젝트 문서](../README-ko.md)를 검토하세요
3. 오프라인 배포 세부 정보를 포함하여 새 이슈를 생성하세요

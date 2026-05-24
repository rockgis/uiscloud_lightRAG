# LightRAG Docker 배포

다양한 LLM 백엔드를 지원하는 경량 지식 그래프 검색 증강 생성 시스템입니다.

## 🚀 준비 사항

### 저장소 클론:

```bash
# Linux/MacOS
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG
```
```powershell
# Windows PowerShell
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG
```

### 환경 설정:

```bash
# Linux/MacOS
cp .env.example .env
# 원하는 설정으로 .env 편집
```
```powershell
# Windows PowerShell
Copy-Item .env.example .env
# 원하는 설정으로 .env 편집
```

LightRAG는 `.env` 파일의 환경 변수로 설정할 수 있습니다:

**서버 설정**

- `HOST`: 서버 호스트 (기본값: 0.0.0.0)
- `PORT`: 서버 포트 (기본값: 9621)

**LLM 설정**

- `LLM_BINDING`: 사용할 LLM 백엔드 (lollms/ollama/openai)
- `LLM_BINDING_HOST`: LLM 서버 호스트 URL
- `LLM_MODEL`: 사용할 모델 이름

**임베딩 설정**

- `EMBEDDING_BINDING`: 임베딩 백엔드 (lollms/ollama/openai)
- `EMBEDDING_BINDING_HOST`: 임베딩 서버 호스트 URL
- `EMBEDDING_MODEL`: 임베딩 모델 이름

**RAG 설정**

- `MAX_ASYNC`: 최대 비동기 작업 수
- `MAX_TOKENS`: 최대 토큰 크기
- `EMBEDDING_DIM`: 임베딩 차원

## 🐳 Docker 배포

Docker 지침은 Docker Desktop이 설치된 모든 플랫폼에서 동일하게 작동합니다.

### 빌드 최적화

Dockerfile은 BuildKit 캐시 마운트를 사용하여 빌드 성능을 크게 향상시킵니다:

- **자동 캐시 관리**: `# syntax=docker/dockerfile:1` 지시어로 BuildKit이 자동 활성화됩니다
- **빠른 재빌드**: `uv.lock` 또는 `bun.lock` 파일이 변경될 때만 의존성을 새로 다운로드합니다
- **효율적인 패키지 캐싱**: UV 및 Bun 패키지 다운로드가 빌드 간에 캐시됩니다
- **별도 설정 불필요**: Docker Compose 및 GitHub Actions에서 기본 작동합니다

### LightRAG 서버 시작:

```bash
docker compose up -d
```

대화형 설정을 사용한 경우 생성된 스택으로 시작:

```bash
docker compose -f docker-compose.final.yml up -d
```

대화형 설정은 `.env`를 호스트에서 사용 가능한 상태로 유지합니다. `postgres` 또는 `host.docker.internal` 같은 컨테이너 전용 호스트명과 `/app/data/certs/` 하위의 SSL 경로는 `.env`가 아닌 생성된 `docker-compose.final.yml`의 `lightrag` 서비스에 주입됩니다.

생성된 스택에 로컬 Milvus가 포함된 경우, Docker Compose는 시작 시 저장소 `.env` 또는 내보낸 셸 환경에서 `MINIO_ACCESS_KEY_ID`와 `MINIO_SECRET_ACCESS_KEY`를 가져옵니다. 둘 중 하나라도 없으면 즉시 종료됩니다.

localhost 외부에 스택을 노출하기 전에 다음을 실행하세요:

```bash
make env-security-check
```

이 명령은 인증 누락, 안전하지 않은 화이트리스트 설정, 약한 JWT 시크릿 등의 보안 위험을 파일 변경 없이 감사합니다.

LightRAG 서버는 다음 경로를 데이터 저장에 사용합니다:

```
data/
├── rag_storage/    # RAG 데이터 영속성
└── inputs/         # 입력 문서
```

### 선택사항: 로컬 vLLM 임베딩 및 리랭커

vLLM으로 임베딩 및/또는 리랭킹을 로컬에서 실행하려면 `make env-base`를 실행하고 Docker를 통해 임베딩 및 리랭크 서비스를 로컬에서 실행할지 묻는 질문에 `yes`로 답하세요.

GPU 호스트용 `docker-compose.override.yml` 예제 (임베딩 + 리랭커):

```yaml
services:
  vllm-embed:
    image: vllm/vllm-openai:latest
    runtime: nvidia
    command: >
      --model BAAI/bge-m3
      --port 8001
      --dtype float16
    ports:
      - "8001:8001"
    volumes:
      - ./data/hf-cache:/root/.cache/huggingface
    ipc: host
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  vllm-rerank:
    image: vllm/vllm-openai:latest
    runtime: nvidia
    command: >
      --model BAAI/bge-reranker-v2-m3
      --port 8000
      --dtype float16
    ports:
      - "8000:8000"
    volumes:
      - ./data/hf-cache:/root/.cache/huggingface
    ipc: host
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

CPU 전용 호스트의 경우 공식 CPU 이미지 사용:

```yaml
services:
  vllm-embed:
    image: vllm/vllm-openai-cpu:latest
    command: >
      --model BAAI/bge-m3
      --port 8001
      --dtype float32
    ports:
      - "8001:8001"
    volumes:
      - ./data/hf-cache:/root/.cache/huggingface

  vllm-rerank:
    image: vllm/vllm-openai-cpu:latest
    command: >
      --model BAAI/bge-reranker-v2-m3
      --port 8000
      --dtype float32
    ports:
      - "8000:8000"
    volumes:
      - ./data/hf-cache:/root/.cache/huggingface
```

`.env`에 임베딩 및 리랭크 설정 추가:

```bash
EMBEDDING_BINDING=openai
EMBEDDING_MODEL=BAAI/bge-m3
EMBEDDING_DIM=1024
EMBEDDING_BINDING_HOST=http://localhost:8001/v1
EMBEDDING_BINDING_API_KEY=local-key

RERANK_BINDING=cohere
RERANK_MODEL=BAAI/bge-reranker-v2-m3
RERANK_BINDING_HOST=http://localhost:8000/rerank
RERANK_BINDING_API_KEY=local-key
```

LightRAG가 Docker에서 실행되고 vLLM이 호스트에서 실행되는 경우, 생성된 컴포즈 파일은 엔드포인트를 다음으로 재작성합니다:

```bash
EMBEDDING_BINDING_HOST=http://host.docker.internal:8001/v1
RERANK_BINDING_HOST=http://host.docker.internal:8000/rerank
```

GPU 사용 시:
```bash
VLLM_EMBED_DEVICE=cuda
VLLM_RERANK_DEVICE=cuda
```

NVIDIA Container Toolkit이 설치되어 있고 호스트에 CUDA 드라이버가 있어야 합니다.

### SSL 인증서

설정 마법사는 컴포즈 파일 생성 전에 TLS 인증서 파일을 `./data/certs/` 하위에 준비합니다.

### PostgreSQL 이미지

대화형 설정은 기본적으로 PostgreSQL을 `gzdaniel/postgres-for-rag:16.6`으로 설정합니다. 이 이미지는 Apache AGE와 pgvector를 함께 제공하므로, 별도 확장 설정 없이 `PGGraphStorage` 및 `PGVectorStorage`와 함께 작동합니다.

### 업데이트

Docker 컨테이너 업데이트:
```bash
docker compose pull
docker compose down
docker compose up
```

### 오프라인 배포

`transformers`, `torch` 또는 `cuda`가 필요한 소프트웨어 패키지는 Docker 이미지에 사전 설치되지 않습니다. Docling 같은 문서 추출 도구나 HuggingFace, LMDeploy 같은 로컬 LLM 모델은 오프라인 환경에서 사용할 수 없습니다. Docling은 독립 서비스로 분리되어 배포될 예정입니다.

## 📦 Docker 이미지 빌드

### 로컬 개발 및 테스트용

```bash
# Docker Compose로 빌드 및 실행 (BuildKit 자동 활성화)
docker compose up --build

# 또는 명시적으로 BuildKit 활성화
DOCKER_BUILDKIT=1 docker compose up --build
```

### 프로덕션 릴리스용

**멀티 아키텍처 빌드 및 푸시**:

```bash
# 제공된 빌드 스크립트 사용
./docker-build-push.sh
```

**빌드 스크립트는 다음을 수행합니다**:

- Docker 레지스트리 로그인 상태 확인
- buildx 빌더 자동 생성/사용
- AMD64 및 ARM64 아키텍처 모두 빌드
- GitHub Container Registry (ghcr.io)에 푸시
- 멀티 아키텍처 매니페스트 검증

**사전 요구 사항**:

- Docker 20.10+ (Buildx 지원 포함)
- 충분한 디스크 공간 (오프라인 이미지의 경우 20GB 이상 권장)
- 레지스트리 접근 자격증명 (이미지 푸시 시)

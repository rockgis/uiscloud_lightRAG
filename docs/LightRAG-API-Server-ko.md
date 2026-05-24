# LightRAG 서버 및 WebUI

LightRAG 서버는 Web UI와 API 지원을 제공합니다. Web UI는 문서 인덱싱, 지식 그래프 탐색, 간단한 RAG 쿼리 인터페이스를 제공합니다. 또한 Ollama 호환 인터페이스를 제공하여 Open WebUI 같은 AI 챗봇이 LightRAG에 쉽게 접근할 수 있습니다.

## 시작하기

### 설치

* PyPI에서 설치

```bash
### uv를 사용하여 LightRAG 서버 설치 (권장)
uv tool install "lightrag-hku[api]"

### 또는 pip 사용
# python -m venv .venv
# source .venv/bin/activate  # Windows: .venv\Scripts\activate
# pip install "lightrag-hku[api]"
```

* 소스에서 설치

```bash
# 저장소 클론
git clone https://github.com/HKUDS/lightrag.git
cd lightrag

# 개발 환경 초기화 (권장)
make dev
source .venv/bin/activate  # Linux/macOS
# Windows: .venv\Scripts\activate

# uv를 사용한 수동 단계
uv sync --extra test --extra offline
source .venv/bin/activate

# 프론트엔드 빌드
cd lightrag_webui
bun install --frozen-lockfile
bun run build
cd ..
```

### LightRAG 서버 시작 전

LightRAG는 문서 인덱싱 및 쿼리 작업을 위해 LLM과 임베딩 모델이 모두 필요합니다. LightRAG는 다양한 LLM/임베딩 백엔드와 바인딩을 지원합니다:

* ollama
* lollms
* openai 또는 openai 호환
* azure_openai
* aws_bedrock
* gemini

설정을 위해 환경 변수 사용을 권장합니다. 프로젝트 루트 디렉토리에 `env.example` 파일이 있습니다. 이 파일을 시작 디렉토리에 `.env`로 복사한 후 LLM 및 임베딩 모델 관련 파라미터를 수정하세요.

**LLM 및 임베딩 설정 예제:**

* OpenAI LLM + Ollama 임베딩:

```
LLM_BINDING=openai
LLM_MODEL=gpt-4o
LLM_BINDING_HOST=https://api.openai.com/v1
LLM_BINDING_API_KEY=your_api_key

EMBEDDING_BINDING=ollama
EMBEDDING_BINDING_HOST=http://localhost:11434
EMBEDDING_MODEL=bge-m3:latest
EMBEDDING_DIM=1024
```

* Ollama LLM + Ollama 임베딩:

```
LLM_BINDING=ollama
LLM_MODEL=mistral-nemo:latest
LLM_BINDING_HOST=http://localhost:11434
### Ollama 서버 컨텍스트 길이 (MAX_TOTAL_TOKENS+2000보다 커야 함)
OLLAMA_LLM_NUM_CTX=16384

EMBEDDING_BINDING=ollama
EMBEDDING_BINDING_HOST=http://localhost:11434
EMBEDDING_MODEL=bge-m3:latest
EMBEDDING_DIM=1024
```

> **중요**: 임베딩 모델은 문서 인덱싱 전에 결정해야 하며, 쿼리 단계에서도 동일한 모델을 사용해야 합니다. 임베딩 모델 변경 시 벡터 관련 테이블/저장소를 삭제하고 새 차원으로 재생성해야 합니다.

### 설정 마법사로 .env 파일 생성

```bash
make env-base           # 필수 첫 단계: LLM, 임베딩, 리랭커
make env-storage        # 선택: 스토리지 백엔드 및 데이터베이스 서비스
make env-server         # 선택: 서버 포트, 인증, SSL
make env-security-check # 선택: 현재 .env 보안 감사
```

### LightRAG 서버 시작

LightRAG 서버는 두 가지 운영 모드를 지원합니다:
* 간단하고 효율적인 Uvicorn 모드:

```
lightrag-server
```

* 멀티프로세스 Gunicorn + Uvicorn 모드 (프로덕션 모드, Windows 미지원):

```
lightrag-gunicorn --workers 4
```

LightRAG를 시작할 때 현재 작업 디렉토리에 `.env` 설정 파일이 있어야 합니다. `.env` 파일을 수정한 후에는 터미널을 다시 열어야 새 설정이 적용됩니다.

시작 시 `.env` 파일의 설정은 커맨드라인 파라미터로 재정의할 수 있습니다:

- `--host`: 서버 수신 주소 (기본값: 0.0.0.0)
- `--port`: 서버 수신 포트 (기본값: 9621)
- `--timeout`: LLM 요청 타임아웃 (기본값: 150초)
- `--log-level`: 로그 레벨 (기본값: INFO)
- `--working-dir`: 데이터베이스 영속성 디렉토리 (기본값: ./rag_storage)
- `--input-dir`: 업로드된 파일 디렉토리 (기본값: ./inputs)
- `--workspace`: 워크스페이스 이름 (다중 인스턴스 간 데이터 논리적 격리)

### Docker로 LightRAG 서버 시작

Docker Compose를 사용하는 것이 가장 편리한 배포 방법입니다:

1. 프로젝트 디렉토리 생성
2. LightRAG 저장소에서 `docker-compose.yml` 파일 복사
3. `.env` 파일 준비: `env.example`을 복사하고 LLM 및 임베딩 파라미터 설정
4. 다음 명령으로 시작:

```shell
docker compose up
# 백그라운드 실행: docker compose up -d
```

### Nginx 리버스 프록시 설정

Nginx를 리버스 프록시로 사용할 때 `/documents/upload` 엔드포인트에 `client_max_body_size`를 설정해야 합니다:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    # 글로벌 기본값: 긴 컨텍스트가 있는 LLM 쿼리에 8MB
    client_max_body_size 8M;

    # 업로드 엔드포인트: 대용량 파일 업로드에 100MB
    location /documents/upload {
        client_max_body_size 100M;

        proxy_pass http://localhost:9621;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 대용량 파일 업로드를 위한 타임아웃 증가
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;
    }

    # 스트리밍 엔드포인트: LLM 응답 스트리밍
    location ~ ^/(query/stream|api/chat|api/generate) {
        gzip off;  # 스트리밍 응답에 압축 비활성화

        proxy_pass http://localhost:9621;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # LLM 생성을 위한 긴 타임아웃
        proxy_read_timeout 300s;
    }

    # 기타 엔드포인트
    location / {
        proxy_pass http://localhost:9621;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 여러 LightRAG 인스턴스 시작

**방법 1**: 각 인스턴스에 완전히 독립적인 작업 환경 설정 (별도 작업 디렉토리 및 `.env` 파일)

**방법 2**: 모든 인스턴스가 동일한 `.env` 파일 공유, 커맨드라인 인수로 포트와 워크스페이스 지정:

```
# 인스턴스 1 시작
lightrag-server --port 9621 --workspace space1

# 인스턴스 2 시작
lightrag-server --port 9622 --workspace space2
```

> **중요**: 다른 인스턴스 간에는 `workspace` 파라미터가 달라야 합니다. 그렇지 않으면 데이터 충돌 및 손상이 발생합니다.

### LightRAG 인스턴스 간 데이터 격리

워크스페이스 구현 방식은 스토리지 유형에 따라 다릅니다:

- **로컬 파일 기반**: 워크스페이스 서브디렉토리로 격리 (`JsonKVStorage`, `NetworkXStorage`, `NanoVectorDBStorage`, `FaissVectorDBStorage`)
- **컬렉션 기반**: 컬렉션 이름에 워크스페이스 접두사 추가 (`RedisKVStorage`, `MilvusVectorDBStorage`, `MongoKVStorage` 등)
- **Qdrant**: 페이로드 기반 파티셔닝으로 격리 (`QdrantVectorDBStorage`)
- **관계형 DB**: 논리적 데이터 분리를 위한 `workspace` 필드 추가 (`PGKVStorage`, `PGVectorStorage`, `PGDocStatusStorage`)
- **그래프 DB**: 레이블을 통한 논리적 격리 (`Neo4JStorage`, `MemgraphStorage`)
- **OpenSearch**: 인덱스 이름 접두사로 격리 (`OpenSearchKVStorage`, `OpenSearchVectorDBStorage`)

### Gunicorn + Uvicorn 멀티 워커

LightRAG 서버는 `Gunicorn + Uvicorn` 프리로드 모드로 운영될 수 있습니다. Gunicorn의 멀티 워커(멀티프로세스) 기능은 문서 인덱싱 작업이 RAG 쿼리를 차단하는 것을 방지합니다.

```
### 워커 프로세스 수, (2 × 코어 수) + 1 이하
WORKERS=2
### 한 번에 병렬로 처리할 파일 수
MAX_PARALLEL_INSERT=2
### LLM에 대한 최대 동시 요청 수
MAX_ASYNC=4
```

### Linux 서비스로 LightRAG 설치

샘플 파일 `lightrag.service.example`에서 서비스 파일 `lightrag.service`를 생성하세요:

```text
Environment="PATH=/home/netman/lightrag-xyj/venv/bin"
WorkingDirectory=/home/netman/lightrag-xyj
ExecStart=/home/netman/lightrag-xyj/venv/bin/lightrag-gunicorn
```

Ubuntu 시스템에서 서비스 설치:

```shell
sudo cp lightrag.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start lightrag.service
sudo systemctl status lightrag.service
sudo systemctl enable lightrag.service
```

---

## Ollama 에뮬레이션

LightRAG를 Ollama 챗 모델로 에뮬레이션하는 Ollama 호환 인터페이스를 제공합니다. Open WebUI 같은 Ollama 지원 AI 챗 프론트엔드가 LightRAG에 쉽게 접근할 수 있습니다.

### Open WebUI를 LightRAG에 연결

lightrag-server를 시작한 후 Open WebUI 관리자 패널에서 Ollama 유형 연결을 추가하면 `lightrag:latest` 모델이 Open WebUI 모델 관리 인터페이스에 나타납니다.

### 채팅에서 쿼리 모드 선택

기본 쿼리 모드는 `hybrid`입니다. 쿼리 접두사로 모드를 선택할 수 있습니다:

```
/local
/global
/hybrid
/naive
/mix

/bypass
/context
/localcontext
/globalcontext
/hybridcontext
/naivecontext
/mixcontext
```

예시: `/mix LightRAG가 무엇인가요?` - mix 모드 쿼리를 트리거합니다.

- `/bypass`: LightRAG 쿼리 모드가 아니며, 쿼리를 채팅 히스토리와 함께 기반 LLM에 직접 전달합니다.
- `/context`: LightRAG 쿼리 모드가 아니며, LLM을 위해 준비된 컨텍스트 정보만 반환합니다.

### 채팅에서 사용자 프롬프트 추가

쿼리 접두사에 대괄호를 추가하여 LLM에 사용자 프롬프트를 제공할 수 있습니다:

```
/[다이어그램에 mermaid 형식 사용] 스크루지의 인물 관계도를 그려주세요
/mix[다이어그램에 mermaid 형식 사용] 스크루지의 인물 관계도를 그려주세요
```

---

## API 키 및 인증

### API 키

```
LIGHTRAG_API_KEY=your-secure-api-key-here
WHITELIST_PATHS=/health,/api/*
```

API 키는 요청 헤더 `X-API-Key`로 전달됩니다:

```
curl -X 'POST' \
  'http://localhost:9621/documents/scan' \
  -H 'accept: application/json' \
  -H 'X-API-Key: your-secure-api-key-here-123' \
  -d ''
```

### 계정 자격증명 (JWT 기반 인증)

```bash
AUTH_ACCOUNTS='admin:{bcrypt}$2b$12$replace-with-generated-hash,user1:pass456'
TOKEN_SECRET='your-key'
TOKEN_EXPIRE_HOURS=4
```

bcrypt 비밀번호를 생성하는 가장 쉬운 방법:

```bash
lightrag-hash-password --username admin
```

---

## Azure OpenAI 백엔드

Azure CLI를 사용한 Azure OpenAI API 생성:

```bash
RESOURCE_GROUP_NAME=LightRAG
LOCATION=swedencentral
RESOURCE_NAME=LightRAG-OpenAI

az login
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION
az cognitiveservices account create --name $RESOURCE_NAME --resource-group $RESOURCE_GROUP_NAME --kind OpenAI --sku S0 --location swedencentral
az cognitiveservices account deployment create --resource-group $RESOURCE_GROUP_NAME --model-format OpenAI --name $RESOURCE_NAME --deployment-name gpt-4o --model-name gpt-4o --model-version "2024-08-06" --sku-capacity 100 --sku-name "Standard"
az cognitiveservices account deployment create --resource-group $RESOURCE_GROUP_NAME --model-format OpenAI --name $RESOURCE_NAME --deployment-name text-embedding-3-large --model-name text-embedding-3-large --model-version "1" --sku-capacity 80 --sku-name "Standard"
az cognitiveservices account show --name $RESOURCE_NAME --resource-group $RESOURCE_GROUP_NAME --query "properties.endpoint"
az cognitiveservices account keys list --name $RESOURCE_NAME -g $RESOURCE_GROUP_NAME
```

`.env` 파일에서 Azure OpenAI 설정:

```
LLM_BINDING=azure_openai
LLM_BINDING_HOST=your-azure-endpoint
LLM_MODEL=your-model-deployment-name
LLM_BINDING_API_KEY=your-azure-api-key
AZURE_OPENAI_API_VERSION=2024-08-01-preview

EMBEDDING_BINDING=azure_openai
EMBEDDING_MODEL=your-embedding-deployment-name
```

---

## LightRAG 서버 상세 설정

API 서버는 두 가지 방법으로 설정할 수 있습니다 (우선순위 높은 순):

1. 커맨드라인 인수
2. 환경 변수 또는 .env 파일

### 지원되는 스토리지 유형

LightRAG는 4가지 유형의 스토리지를 사용합니다:

* **KV_STORAGE**: LLM 응답 캐시, 텍스트 청크, 문서 정보
* **VECTOR_STORAGE**: 엔티티 벡터, 관계 벡터, 청크 벡터
* **GRAPH_STORAGE**: 지식 그래프
* **DOC_STATUS_STORAGE**: 문서 처리 상태

스토리지 선택 환경 변수:

```bash
LIGHTRAG_KV_STORAGE=JsonKVStorage          # 기본값
LIGHTRAG_VECTOR_STORAGE=NanoVectorDBStorage # 기본값
LIGHTRAG_GRAPH_STORAGE=NetworkXStorage      # 기본값
LIGHTRAG_DOC_STATUS_STORAGE=JsonDocStatusStorage # 기본값
```

### 최대 출력 토큰 설정

LLM 응답의 과도하게 길거나 무한 루프 출력을 **방지**하기 위해 max_tokens를 설정하세요:

```
# vLLM/SGLang 배포 모델 또는 대부분의 OpenAI 호환 API 제공자
OPENAI_LLM_MAX_TOKENS=9000

# Ollama 배포 모델
OLLAMA_LLM_NUM_PREDICT=9000

# OpenAI o1-mini 또는 최신 모델
OPENAI_LLM_MAX_COMPLETION_TOKENS=9000
```

### 엔티티 추출 설정

```
ENABLE_LLM_CACHE_FOR_EXTRACT=true  # 기본값, 테스트 환경에서 LLM 비용 절감
```

---

## REST API 엔드포인트

### 문서 API

| 메서드 | 경로 | 설명 |
|--------|------|------|
| `POST` | `/documents/upload` | 파일 업로드 |
| `POST` | `/documents/upload_uris` | URI 기반 문서 수집 |
| `GET` | `/documents/paginated` | 문서 목록 (페이지네이션) |
| `GET` | `/documents/{doc_id}` | 특정 문서 조회 |
| `DELETE` | `/documents/{doc_id}` | 문서 삭제 |
| `GET` | `/documents/pipeline_status` | 처리 파이프라인 상태 |
| `POST` | `/documents/rebuild` | 지식 그래프 재구축 |

### 쿼리 API

| 메서드 | 경로 | 설명 |
|--------|------|------|
| `POST` | `/query` | 표준 RAG 쿼리 |
| `POST` | `/query/stream` | 실시간 스트리밍 쿼리 |
| `POST` | `/query/data` | 원시 검색 데이터 (LLM 생성 없음) |

### 그래프 API

| 메서드 | 경로 | 설명 |
|--------|------|------|
| `GET` | `/graph/nodes` | 엔티티 노드 조회 |
| `GET` | `/graph/edges` | 관계 엣지 조회 |
| `POST` | `/graph/query` | 그래프 순회 검색 |
| `GET` | `/graph/statistics` | 그래프 통계 |
| `DELETE` | `/graph/reset` | 지식 그래프 초기화 |

### 쿼리 파라미터 예제

```json
{
  "query": "LightRAG란 무엇인가요?",
  "mode": "mix",
  "stream": false,
  "top_k": 60,
  "chunk_top_k": 20,
  "max_entity_tokens": 6000,
  "max_relation_tokens": 8000,
  "max_total_tokens": 30000,
  "enable_rerank": true,
  "user_prompt": "한국어로 답변해주세요"
}
```

# LightRAG 아키텍처 문서

> 버전: 1.4.16 | API 버전: 0291 | 작성일: 2026-05-24 | 최종 업데이트: 2026-05-26

---

## 목차

0. [현재 운영 환경 (스냅샷)](#0-현재-운영-환경-스냅샷)
1. [프로젝트 개요](#1-프로젝트-개요)
2. [전체 아키텍처 구조](#2-전체-아키텍처-구조)
3. [핵심 컴포넌트](#3-핵심-컴포넌트)
4. [스토리지 레이어](#4-스토리지-레이어)
5. [LLM 프로바이더 레이어](#5-llm-프로바이더-레이어)
6. [문서 처리 파이프라인](#6-문서-처리-파이프라인)
7. [쿼리 처리 파이프라인](#7-쿼리-처리-파이프라인)
8. [API 서버](#8-api-서버)
9. [WebUI](#9-webui)
10. [인증 및 보안](#10-인증-및-보안)
11. [워크스페이스 격리](#11-워크스페이스-격리)
12. [비동기 처리 아키텍처](#12-비동기-처리-아키텍처)
13. [설정 관리](#13-설정-관리)
14. [배포 옵션](#14-배포-옵션)
15. [데이터 흐름 다이어그램](#15-데이터-흐름-다이어그램)

---

## 0. 현재 운영 환경 스냅샷

> 기준일: 2026-05-26 | 상태: 정상 운영 중

### 0.1 전체 서비스 구성도

```
┌─────────────────────────────────────────────────────────────────────┐
│                        로컬 개발 머신 (macOS)                         │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Docker Desktop                             │   │
│  │                                                              │   │
│  │  ┌─────────────────────────────┐                            │   │
│  │  │  uiscloud_lightrag 스택      │                            │   │
│  │  │  ┌───────────────────────┐  │                            │   │
│  │  │  │ uiscloud_lightrag-    │  │  포트 9621                 │   │
│  │  │  │ lightrag-1            │◄─┼──────────────────────────  │   │
│  │  │  │ (ghcr.io/hkuds/       │  │                            │   │
│  │  │  │  lightrag:latest      │  │                            │   │
│  │  │  │  v1.4.16)             │  │                            │   │
│  │  │  └───────────────────────┘  │                            │   │
│  │  └─────────────────────────────┘                            │   │
│  │                                                              │   │
│  │  ┌─────────────────────────────┐                            │   │
│  │  │  HypermakinaRAG 스택        │                            │   │
│  │  │  ├─ frontend    (포트 80)   │                            │   │
│  │  │  ├─ app         (포트 8085) │                            │   │
│  │  │  ├─ docreader   (포트 50051)│                            │   │
│  │  │  ├─ postgres    (내부)      │                            │   │
│  │  │  └─ redis       (내부)      │                            │   │
│  │  └─────────────────────────────┘                            │   │
│  │                                                              │   │
│  │  ┌─────────────────────────────┐                            │   │
│  │  │  client-services 스택       │                            │   │
│  │  │  ├─ kb-search    (포트 8001)│                            │   │
│  │  │  └─ error-guide  (포트 8002)│                            │   │
│  │  └─────────────────────────────┘                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Ollama (네이티브)  ──────────────────────── 포트 11434             │
│    ├─ bge-m3:latest     (임베딩, 1024dim)                           │
│    ├─ qwen3:14b         (로컬 LLM)                                  │
│    └─ exaone3.5:7.8b    (로컬 LLM)                                  │
└─────────────────────────────────────────────────────────────────────┘
                │ LiteLLM Proxy API (sk-dgx-proxy)
                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   DGX 서버 (192.168.0.210)                           │
│                                                                     │
│  LiteLLM Proxy (포트 4000)                                          │
│    ├─ Qwen3-30B-A3B       (LLM, thinking 비활성화)                  │
│    ├─ bge-m3              (임베딩)                                   │
│    └─ bge-reranker-v2-m3  (리랭킹)                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 0.2 uiscloud LightRAG 서비스 현황

| 항목 | 값 |
|------|-----|
| **컨테이너명** | `uiscloud_lightrag-lightrag-1` |
| **이미지** | `ghcr.io/hkuds/lightrag:latest` |
| **코어 버전** | v1.4.16 |
| **API 버전** | 0291 |
| **상태** | Up 45시간 이상 / healthy |
| **WebUI** | http://localhost:9621 (접속 가능) |
| **API 문서** | http://localhost:9621/docs |
| **재시작 횟수** | 0회 (안정 운영) |
| **인증** | 비활성화 (로컬 개발 모드) |
| **인덱싱된 문서** | 0건 (초기 상태) |

### 0.3 LLM / Embedding 설정 (`.env` 현행)

```bash
# LLM: LiteLLM Proxy → Qwen3-30B-A3B (thinking 비활성화)
LLM_BINDING=openai
LLM_BINDING_HOST=http://192.168.0.210:4000/v1
LLM_BINDING_API_KEY=sk-dgx-proxy
LLM_MODEL=Qwen3-30B-A3B
LLM_TIMEOUT=180
OPENAI_LLM_EXTRA_BODY='{"chat_template_kwargs": {"enable_thinking": false}}'
OPENAI_LLM_MAX_TOKENS=4096

# Embedding: Ollama bge-m3:latest (Docker 내부에서 host.docker.internal 경유)
EMBEDDING_BINDING=ollama
EMBEDDING_BINDING_HOST=http://host.docker.internal:11434
EMBEDDING_MODEL=bge-m3:latest
EMBEDDING_DIM=1024
OLLAMA_EMBEDDING_NUM_CTX=8192
```

### 0.4 스토리지 설정 (파일 기반)

| 스토리지 | 구현체 | 경로 |
|---------|--------|------|
| KV | `JsonKVStorage` | `./data/rag_storage/` |
| Vector | `NanoVectorDBStorage` | `./data/rag_storage/` |
| Graph | `NetworkXStorage` | `./data/rag_storage/graph_chunk_entity_relation.graphml` |
| DocStatus | `JsonDocStatusStorage` | `./data/rag_storage/` |

### 0.5 Docker 볼륨 마운트

```
호스트 경로                                          컨테이너 경로
./data/rag_storage                    →  /app/data/rag_storage
./data/inputs                         →  /app/data/inputs
./.env                                →  /app/.env
./config.ini                          →  /app/config.ini
```

### 0.6 전체 Docker 컨테이너 목록

| 컨테이너 | 이미지 | 포트 | 상태 |
|---------|--------|------|------|
| `uiscloud_lightrag-lightrag-1` | `ghcr.io/hkuds/lightrag:latest` | 9621 | Up (healthy) |
| `HypermakinaRAG-frontend` | `knowwheresoft/hypermakinarag-ui:latest` | 80 | Up |
| `HypermakinaRAG-app` | `knowwheresoft/hypermakinarag-app:latest` | 8085 | Up (healthy) |
| `HypermakinaRAG-docreader` | `knowwheresoft/hypermakinarag-docreader:latest` | 50051 | Up (healthy) |
| `HypermakinaRAG-postgres` | `paradedb/paradedb:v0.22.2-pg17` | 5432 (내부) | Up (healthy) |
| `HypermakinaRAG-redis` | `redis:7.0-alpine` | 6379 (내부) | Up |
| `client-services-kb-search-1` | `client-services-kb-search` | 8001 | Up (healthy) |
| `client-services-error-guide-1` | `client-services-error-guide` | 8002 | Up (healthy) |

### 0.7 쿼리 설정 현행값

| 파라미터 | 값 | 설명 |
|---------|-----|------|
| `TOP_K` | 40 | KG 엔티티/관계 검색 수 |
| `CHUNK_TOP_K` | 20 | 텍스트 청크 검색 수 |
| `MAX_ENTITY_TOKENS` | 6,000 | 엔티티 컨텍스트 최대 토큰 |
| `MAX_RELATION_TOKENS` | 8,000 | 관계 컨텍스트 최대 토큰 |
| `MAX_TOTAL_TOKENS` | 30,000 | 전체 컨텍스트 최대 토큰 |
| `COSINE_THRESHOLD` | 0.2 | 벡터 유사도 최소 임계값 |
| `ENABLE_LLM_CACHE` | true | 쿼리 LLM 캐시 활성화 |
| `ENABLE_LLM_CACHE_FOR_EXTRACT` | true | 추출 LLM 캐시 활성화 |
| `SUMMARY_LANGUAGE` | Korean | 요약 언어 |
| `CHUNK_SIZE` | 1,200 tokens | 청크 크기 |
| `CHUNK_OVERLAP_SIZE` | 100 tokens | 청크 오버랩 |

### 0.8 Git 저장소

| 항목 | 값 |
|------|-----|
| **브랜치** | `main` |
| **Remote** | `https://github.com/rockgis/uiscloud_lightRAG.git` |
| **최신 커밋** | `99b39c51` docs(ko): add Korean translations |

---

## 1. 프로젝트 개요

LightRAG는 **지식 그래프(Knowledge Graph) 기반 RAG(Retrieval-Augmented Generation)** 프레임워크다. 일반적인 벡터 검색 기반 RAG와 달리, 문서에서 엔티티(Entity)와 관계(Relation)를 추출해 그래프로 구성하고, 이를 다중 검색 모드(local / global / hybrid / mix / naive)로 질의에 활용한다.

**핵심 특징:**
- 지식 그래프 + 벡터 검색 결합
- 5가지 검색 모드 지원
- 12종 스토리지 백엔드 플러그인
- 12종 LLM 프로바이더 지원
- FastAPI 기반 REST API + Ollama 호환 API
- React 19 WebUI 내장
- 멀티 워크스페이스 격리

---

## 2. 전체 아키텍처 구조

```
┌──────────────────────────────────────────────────────────────────────┐
│                          클라이언트 레이어                             │
│   WebUI (React 19)  │  REST API Client  │  Ollama-Compatible Client  │
└──────────────────────────────┬───────────────────────────────────────┘
                               │ HTTP
┌──────────────────────────────▼───────────────────────────────────────┐
│                       API 서버 레이어 (FastAPI)                        │
│  lightrag/api/lightrag_server.py                                     │
│  ┌─────────────┐ ┌──────────────┐ ┌─────────────┐ ┌──────────────┐  │
│  │ /documents  │ │   /query     │ │   /graph    │ │  /api (Ollama│  │
│  │ 문서 관리   │ │  쿼리 처리   │ │  그래프 관리 │ │  호환 API)  │  │
│  └─────────────┘ └──────────────┘ └─────────────┘ └──────────────┘  │
│  JWT 인증 / API Key 인증 / CORS / 멀티 워크스페이스 헤더              │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────┐
│                       핵심 엔진 레이어                                 │
│                   lightrag/lightrag.py (LightRAG)                    │
│                                                                      │
│   ainsert() ──► 문서 처리 파이프라인 ──► 스토리지 저장               │
│   aquery()  ──► 쿼리 처리 파이프라인 ──► LLM 응답 생성               │
└────────┬─────────────────────┬────────────────────────────────────────┘
         │                     │
┌────────▼──────────┐  ┌───────▼──────────────────────────────────────┐
│  operate.py       │  │              스토리지 레이어                   │
│ - chunking        │  │                                              │
│ - extract_entities│  │  KV Storage   Vector Storage  Graph Storage  │
│ - merge_nodes     │  │  (청크/캐시)  (임베딩 검색)   (그래프 탐색)   │
│ - kg_query        │  │                                              │
│ - naive_query     │  │  DocStatus Storage (문서 처리 상태 추적)      │
└────────┬──────────┘  └───────────────┬──────────────────────────────┘
         │                             │
┌────────▼─────────────────────────────▼──────────────────────────────┐
│                       LLM / Embedding 레이어                          │
│   lightrag/llm/                                                      │
│   OpenAI │ Anthropic │ Gemini │ Ollama │ Bedrock │ HuggingFace ...   │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3. 핵심 컴포넌트

### 3.1 LightRAG 클래스 (`lightrag/lightrag.py`)

메인 오케스트레이터. `@dataclass`로 정의된 4,456줄 규모의 클래스다.

**초기화 필수 흐름:**
```python
rag = LightRAG(...)
await rag.initialize_storages()   # 반드시 호출
# ... 사용
await rag.finalize_storages()     # 정리
```

**주요 메서드:**

| 메서드 | 설명 |
|--------|------|
| `ainsert(content, ids, file_paths)` | 문서 비동기 삽입 |
| `ainsert_custom_chunks(chunks)` | 사전 분할된 청크 삽입 |
| `ainsert_custom_kg(kg)` | 커스텀 지식 그래프 삽입 |
| `aquery(query, param)` | 비동기 RAG 쿼리 |
| `aquery_data(query, param)` | 컨텍스트 데이터 반환 쿼리 |
| `adelete_by_doc_id(doc_id)` | 문서 ID로 삭제 |
| `adelete_by_entity(entity_name)` | 엔티티 삭제 |
| `aedit_entity / aedit_relation` | 엔티티/관계 편집 |
| `acreate_entity / acreate_relation` | 엔티티/관계 생성 |
| `amerge_entities(entities)` | 엔티티 병합 |
| `aexport_data(output_path)` | 데이터 내보내기 |
| `get_knowledge_graph(label)` | 지식 그래프 조회 |

**주요 설정 필드:**

```python
@dataclass
class LightRAG:
    working_dir: str                    # 작업 디렉토리
    workspace: str                      # 워크스페이스 격리 키
    llm_model_func: Callable            # LLM 함수
    embedding_func: EmbeddingFunc       # 임베딩 함수

    # 스토리지 선택
    kv_storage: str = "JsonKVStorage"
    vector_storage: str = "NanoVectorDBStorage"
    graph_storage: str = "NetworkXStorage"
    doc_status_storage: str = "JsonDocStatusStorage"

    # 추출 설정
    entity_types: list[str]             # 추출할 엔티티 타입
    max_parallel_insert: int = 2        # 병렬 삽입 수
    max_async: int                      # LLM 동시 요청 수
    summary_language: str = "English"   # 요약 언어

    # 쿼리 설정
    top_k: int = 40
    chunk_top_k: int = 20
    max_total_tokens: int = 30000
```

---

### 3.2 operate.py - 핵심 처리 로직

추출과 쿼리의 핵심 알고리즘이 구현된 5,197줄 모듈.

**문서 처리:**

| 함수 | 역할 |
|------|------|
| `chunking_by_token_size()` | 토큰 기반 문서 분할 |
| `extract_entities()` | LLM으로 엔티티/관계 추출 |
| `_handle_single_entity_extraction()` | 단일 엔티티 파싱 |
| `_handle_single_relationship_extraction()` | 단일 관계 파싱 |
| `merge_nodes_and_edges()` | 기존 그래프와 병합 |

**쿼리 처리:**

| 함수 | 역할 |
|------|------|
| `kg_query()` | 지식 그래프 기반 쿼리 (local/global/hybrid/mix) |
| `naive_query()` | 직접 벡터 검색 쿼리 |
| `rebuild_knowledge_from_chunks()` | 청크에서 지식 재구성 |

---

### 3.3 base.py - 추상 기반 클래스

스토리지 인터페이스를 정의하는 추상 클래스 모음.

```
StorageNameSpace (ABC)
├── BaseKVStorage (ABC)          # get_by_id, upsert, delete
├── BaseVectorStorage (ABC)      # query, upsert, delete
├── BaseGraphStorage (ABC)       # upsert_node, upsert_edge, get_node, get_edge
└── DocStatusStorage(BaseKVStorage, ABC)  # get_docs_by_status
```

**주요 데이터 클래스:**

```python
@dataclass
class QueryParam:
    mode: Literal["local","global","hybrid","naive","mix","bypass"] = "mix"
    top_k: int = 40
    chunk_top_k: int = 20
    max_entity_tokens: int = 6000
    max_relation_tokens: int = 8000
    max_total_tokens: int = 30000
    enable_rerank: bool = False
    stream: bool = False
    conversation_history: list[dict] = []

class DocStatus(Enum):
    PENDING = "pending"
    PROCESSING = "processing"
    PROCESSED = "processed"
    FAILED = "failed"

@dataclass
class DocProcessingStatus:
    id: str
    content: str
    status: DocStatus
    chunks_list: list[str]
    file_path: str
    created_at: datetime
    updated_at: datetime
```

---

### 3.4 namespace.py - 스토리지 네임스페이스

```python
class NameSpace:
    # KV 스토리지 네임스페이스
    KV_STORE_FULL_DOCS = "full_docs"           # 전체 문서 원본
    KV_STORE_TEXT_CHUNKS = "text_chunks"        # 분할된 청크
    KV_STORE_LLM_RESPONSE_CACHE = "llm_response_cache"  # LLM 캐시
    KV_STORE_FULL_ENTITIES = "full_entities"    # 엔티티 메타데이터
    KV_STORE_FULL_RELATIONS = "full_relations"  # 관계 메타데이터
    KV_STORE_ENTITY_CHUNKS = "entity_chunks"    # 엔티티-청크 매핑
    KV_STORE_RELATION_CHUNKS = "relation_chunks" # 관계-청크 매핑

    # 벡터 스토리지 네임스페이스
    VECTOR_STORE_ENTITIES = "entities"          # 엔티티 임베딩
    VECTOR_STORE_RELATIONSHIPS = "relationships" # 관계 임베딩
    VECTOR_STORE_CHUNKS = "chunks"              # 청크 임베딩

    # 그래프 스토리지 네임스페이스
    GRAPH_STORE_CHUNK_ENTITY_RELATION = "chunk_entity_relation"

    # 문서 상태 네임스페이스
    DOC_STATUS = "doc_status"
```

---

### 3.5 utils.py - 유틸리티

2,856줄 이상의 핵심 유틸리티 모음.

| 클래스/함수 | 역할 |
|-------------|------|
| `EmbeddingFunc` | 임베딩 함수 래퍼 (embedding_dim, max_token_size 속성 부여) |
| `priority_limit_async_func_call` | 우선순위 큐 기반 LLM 요청 제한기 |
| `handle_cache / save_to_cache` | LLM 응답 캐싱 |
| `pick_by_weighted_polling` | 가중치 폴링 기반 청크 선택 |
| `pick_by_vector_similarity` | 벡터 유사도 기반 청크 선택 |
| `process_chunks_unified` | 리랭킹 포함 통합 청크 처리 |
| `Tokenizer / TiktokenTokenizer` | 토큰 카운팅 |
| `compute_mdhash_id` | MD5 해시 기반 ID 생성 |
| `CacheData` | LLM 캐시 데이터 구조 |
| `TokenTracker` | 토큰 사용량 추적 |

---

## 4. 스토리지 레이어

### 4.1 스토리지 타입 매트릭스

| 스토리지 타입 | 역할 | 기본값 |
|---------------|------|--------|
| `KV_STORAGE` | 문서, 청크, LLM 캐시, 엔티티/관계 메타데이터 | `JsonKVStorage` |
| `VECTOR_STORAGE` | 엔티티/관계/청크 임베딩 벡터 검색 | `NanoVectorDBStorage` |
| `GRAPH_STORAGE` | 엔티티-관계 그래프 구조 | `NetworkXStorage` |
| `DOC_STATUS_STORAGE` | 문서 처리 상태 추적 | `JsonDocStatusStorage` |

### 4.2 스토리지 구현체 목록

**KV 스토리지:**
| 구현체 | 백엔드 | 환경변수 |
|--------|--------|---------|
| `JsonKVStorage` | 로컬 JSON 파일 | 없음 |
| `RedisKVStorage` | Redis | `REDIS_URI` |
| `PGKVStorage` | PostgreSQL | `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DATABASE` |
| `MongoKVStorage` | MongoDB | `MONGO_URI`, `MONGO_DATABASE` |
| `OpenSearchKVStorage` | OpenSearch | `OPENSEARCH_HOSTS` |

**벡터 스토리지:**
| 구현체 | 백엔드 | 환경변수 |
|--------|--------|---------|
| `NanoVectorDBStorage` | 인메모리 (파일 영속) | 없음 |
| `MilvusVectorDBStorage` | Milvus | `MILVUS_URI`, `MILVUS_DB_NAME` |
| `PGVectorStorage` | PostgreSQL + pgvector | `POSTGRES_*` |
| `FaissVectorDBStorage` | FAISS | 없음 |
| `QdrantVectorDBStorage` | Qdrant | `QDRANT_URL` |
| `MongoVectorDBStorage` | MongoDB Atlas | `MONGO_URI` |
| `OpenSearchVectorDBStorage` | OpenSearch | `OPENSEARCH_HOSTS` |

**그래프 스토리지:**
| 구현체 | 백엔드 | 환경변수 |
|--------|--------|---------|
| `NetworkXStorage` | 인메모리 NetworkX | 없음 |
| `Neo4JStorage` | Neo4j | `NEO4J_URI`, `NEO4J_USERNAME`, `NEO4J_PASSWORD` |
| `PGGraphStorage` | PostgreSQL + Apache AGE | `POSTGRES_*` |
| `MongoGraphStorage` | MongoDB | `MONGO_URI` |
| `MemgraphStorage` | Memgraph | `MEMGRAPH_URI` |
| `OpenSearchGraphStorage` | OpenSearch | `OPENSEARCH_HOSTS` |

**DocStatus 스토리지:**
| 구현체 | 백엔드 |
|--------|--------|
| `JsonDocStatusStorage` | 로컬 JSON 파일 |
| `RedisDocStatusStorage` | Redis |
| `PGDocStatusStorage` | PostgreSQL |
| `MongoDocStatusStorage` | MongoDB |
| `OpenSearchDocStatusStorage` | OpenSearch |

### 4.3 스토리지 레지스트리 (`lightrag/kg/__init__.py`)

동적 모듈 로딩 방식으로 필요한 스토리지만 임포트:

```python
STORAGES = {
    "NetworkXStorage": ".kg.networkx_impl",
    "JsonKVStorage": ".kg.json_kv_impl",
    "NanoVectorDBStorage": ".kg.nano_vector_db_impl",
    "Neo4JStorage": ".kg.neo4j_impl",
    "PGKVStorage": ".kg.postgres_impl",
    # ...
}
```

---

## 5. LLM 프로바이더 레이어

### 5.1 지원 프로바이더

| 파일 | 프로바이더 | LLM | 임베딩 |
|------|-----------|-----|--------|
| `openai.py` | OpenAI / OpenAI 호환 | ✅ | ✅ |
| `azure_openai.py` | Azure OpenAI | ✅ | ✅ |
| `anthropic.py` | Anthropic Claude | ✅ | ❌ |
| `gemini.py` | Google Gemini | ✅ | ✅ |
| `ollama.py` | Ollama (로컬) | ✅ | ✅ |
| `bedrock.py` | AWS Bedrock | ✅ | ✅ |
| `hf.py` | HuggingFace | ✅ | ✅ |
| `zhipu.py` | ZhipuAI | ✅ | ✅ |
| `nvidia_openai.py` | NVIDIA NIM | ✅ | ✅ |
| `lmdeploy.py` | LMDeploy | ✅ | ❌ |
| `lollms.py` | LoLLMS | ✅ | ✅ |
| `jina.py` | Jina AI | ❌ | ✅ (리랭킹 포함) |
| `llama_index_impl.py` | LlamaIndex | ✅ | ✅ |

### 5.2 임베딩 함수 패턴

```python
from lightrag.utils import wrap_embedding_func_with_attrs

@wrap_embedding_func_with_attrs(embedding_dim=1536, max_token_size=8192)
async def custom_embed(texts: list[str]) -> np.ndarray:
    return await openai_embed.func(texts, model="text-embedding-3-large")
```

### 5.3 LLM 캐싱

LLM 응답은 `llm_response_cache` 네임스페이스에 자동 캐싱된다. 캐시 키는 입력 프롬프트의 해시값으로 생성된다. 쿼리 모드(`local`, `global` 등)와 캐시 타입을 조합해 키를 구분한다.

---

## 6. 문서 처리 파이프라인

### 6.1 전체 흐름

```
입력 문서 (텍스트 / 파일)
        │
        ▼
[1] 문서 큐 등록 (apipeline_enqueue_documents)
    - MD5 해시로 doc_id 생성
    - DocStatus: PENDING
        │
        ▼
[2] 청크 분할 (chunking_by_token_size)
    - 기본 토큰 크기: 1200 tokens
    - 오버랩: 100 tokens
    - 각 청크에 순서 인덱스 부여
        │
        ▼
[3] 엔티티/관계 추출 (extract_entities) ← LLM 호출
    - entity_extraction_system_prompt 사용
    - 최대 1회 gleaning (추가 추출)
    - 추출 결과 파싱: entity / relation 라인 분리
        │
        ▼
[4] 그래프 병합 (merge_nodes_and_edges)
    - 동일 엔티티: 설명 병합, LLM 요약 (8개 이상 시)
    - 관계: 키워드/설명 병합
    - 소스 ID 추적 (FIFO/KEEP 방식)
        │
        ▼
[5] 스토리지 저장 (병렬)
    ├── KV: 원본 문서, 청크, 엔티티 메타, 관계 메타
    ├── Vector: 엔티티 임베딩, 관계 임베딩, 청크 임베딩
    └── Graph: 엔티티 노드, 관계 엣지
        │
        ▼
[6] 상태 업데이트
    - DocStatus: PROCESSED
    - 실패 시: FAILED (재시도 가능)
```

### 6.2 청크 분할 상세

```python
def chunking_by_token_size(
    content: str,
    chunk_token_size: int = 1200,
    chunk_overlap_token_size: int = 100,
    tiktoken_model: str = "gpt-4o"
) -> list[TextChunkSchema]:
    # TextChunkSchema: {tokens, content, full_doc_id, chunk_order_index}
```

### 6.3 엔티티 추출 프롬프트

LLM에 다음 포맷으로 출력을 요청:
```
entity<|#|>entity_name<|#|>entity_type<|#|>entity_description
relation<|#|>source_entity<|#|>target_entity<|#|>keywords<|#|>description
<|COMPLETE|>
```

기본 추출 엔티티 타입: `Person, Creature, Organization, Location, Event, Concept, Method, Content, Data, Artifact, NaturalObject`

---

## 7. 쿼리 처리 파이프라인

### 7.1 쿼리 모드 비교

| 모드 | 검색 방식 | 최적 사용 케이스 |
|------|-----------|----------------|
| `local` | 엔티티 중심 → 연관 청크 | 특정 엔티티에 대한 상세 질문 |
| `global` | 커뮤니티/관계 중심 → 광범위 지식 | 전반적 주제/트렌드 질문 |
| `hybrid` | local + global 결합 | 일반적 질문 |
| `naive` | 청크 벡터 검색만 | 단순 키워드 검색 |
| `mix` | KG(local+global) + 벡터 검색 | **권장** (리랭커 함께 사용) |
| `bypass` | 검색 없이 LLM 직접 호출 | 지식베이스 불필요한 질문 |

### 7.2 kg_query 상세 흐름 (mix 모드)

```
질문 입력
    │
    ▼
[1] 키워드 추출 (LLM)
    - high_level_keywords: 개념/주제 키워드
    - low_level_keywords: 구체적 엔티티 키워드
    │
    ├──────────────────────────────────────────────────────┐
    │ [Local 경로]                           [Global 경로] │
    ▼                                                      ▼
[2L] 엔티티 벡터 검색              [2G] 관계 벡터 검색
     (ll_keywords → VECTOR_STORE_ENTITIES)   (hl_keywords → VECTOR_STORE_RELATIONSHIPS)
    │                                                      │
    ▼                                                      ▼
[3L] 엔티티 이웃 관계 조회         [3G] 관계 연결 엔티티 조회
     (Graph 탐색)                       (Graph 탐색)
    │                                                      │
    ▼                                                      ▼
[4L] 엔티티 연관 청크 수집          [4G] 관계 연관 청크 수집
    │                                                      │
    └──────────────────────┬───────────────────────────────┘
                           │
                           ▼
              [5] 청크 벡터 검색 결과 추가
                  (VECTOR_STORE_CHUNKS)
                           │
                           ▼
              [6] 리랭킹 (선택사항)
                  - 모든 청크를 질문과 유사도 재평가
                           │
                           ▼
              [7] 토큰 예산 내 컨텍스트 구성
                  - max_entity_tokens: 6,000
                  - max_relation_tokens: 8,000
                  - max_total_tokens: 30,000
                           │
                           ▼
              [8] LLM 최종 응답 생성
                  (rag_response 프롬프트)
```

### 7.3 청크 선택 알고리즘

두 가지 방식 중 선택 (`KG_CHUNK_PICK_METHOD`):

- **`VECTOR`** (기본): 벡터 유사도 기반 선택
- **`WEIGHTED_POLLING`**: 가중치 폴링 - 여러 엔티티가 참조하는 청크에 높은 가중치 부여, 라운드 로빈으로 균형 있게 선택

---

## 8. API 서버

### 8.1 구조

```
lightrag/api/
├── lightrag_server.py      # FastAPI 앱 팩토리 (create_app)
├── config.py               # CLI 인자 + 환경변수 파싱
├── auth.py                 # JWT 인증 로직
├── passwords.py            # bcrypt 패스워드 처리
├── utils_api.py            # API 유틸리티
├── runtime_validation.py   # 런타임 설정 검증
├── gunicorn_config.py      # Gunicorn 멀티워커 설정
├── run_with_gunicorn.py    # Gunicorn 진입점
├── routers/
│   ├── document_routes.py  # 문서 관리 라우터
│   ├── query_routes.py     # 쿼리 라우터
│   ├── graph_routes.py     # 그래프 관리 라우터
│   └── ollama_api.py       # Ollama 호환 API
└── webui/                  # React SPA 빌드 결과물
```

### 8.2 주요 엔드포인트

**인증:**
```
POST /login                  JWT 토큰 발급
GET  /auth-status            인증 상태 확인
```

**문서 관리 (`/documents`):**
```
POST   /documents/upload         파일 업로드 (PDF, DOCX, PPTX, XLSX, TXT 등)
POST   /documents/text           텍스트 직접 삽입
GET    /documents                문서 목록 조회
GET    /documents/{id}           특정 문서 조회
DELETE /documents/{id}           문서 삭제
GET    /documents/status/{id}    처리 상태 조회
POST   /documents/scan           디렉토리 스캔 및 인덱싱
POST   /documents/pipeline/enqueue   파이프라인 큐 등록
POST   /documents/pipeline/run       파이프라인 실행
```

**쿼리 (`/query`):**
```
POST /query          RAG 쿼리 (스트리밍 지원)
POST /query/data     컨텍스트 데이터 반환 쿼리
```

**그래프 (`/graph`):**
```
GET    /graph/label              그래프 레이블 목록
GET    /graph/nodes/{label}      노드 목록 조회
GET    /graph/edges/{label}      엣지 목록 조회
GET    /graph/node/{id}          노드 상세 조회
GET    /graph/edge/{src}/{tgt}   엣지 상세 조회
PUT    /graph/node/{id}          노드 수정
PUT    /graph/edge/{src}/{tgt}   엣지 수정
DELETE /graph/node/{id}          노드 삭제
DELETE /graph/edge/{src}/{tgt}   엣지 삭제
POST   /graph/nodes/merge        노드 병합
```

**Ollama 호환 (`/api`):**
```
POST /api/chat              Ollama chat API 에뮬레이션
POST /api/generate          Ollama generate API 에뮬레이션
GET  /api/tags              모델 목록
GET  /api/version           버전 정보
```

### 8.3 멀티 워크스페이스 지원

HTTP 헤더로 워크스페이스 지정:
```
LIGHTRAG-WORKSPACE: my_workspace
```

서버는 요청마다 해당 워크스페이스의 스토리지를 동적으로 로드한다.

### 8.4 지원 문서 포맷

| 확장자 | 처리 방식 |
|--------|---------|
| `.txt`, `.md` | 직접 읽기 |
| `.pdf` | pypdf 또는 Docling |
| `.docx` | python-docx |
| `.pptx` | python-pptx |
| `.xlsx` | openpyxl |
| 기타 | Docling (선택적 설치) |

---

## 9. WebUI

### 9.1 기술 스택

| 항목 | 기술 |
|------|------|
| 프레임워크 | React 19 |
| 언어 | TypeScript |
| 빌드 도구 | Vite + **Bun** |
| 스타일링 | Tailwind CSS |
| 그래프 시각화 | Sigma.js |
| 상태 관리 | React Hooks + Context |
| 국제화 | i18next (다국어 지원) |
| 패키지 매니저 | **Bun** (npm/yarn 사용 금지) |

### 9.2 주요 기능 화면

```
lightrag_webui/src/
├── features/
│   ├── GraphViewer.tsx        # 지식 그래프 인터랙티브 시각화
│   ├── DocumentManager.tsx    # 문서 업로드/관리/상태 확인
│   ├── RetrievalTesting.tsx   # RAG 쿼리 테스트 인터페이스
│   └── ApiSite.tsx            # Swagger UI 통합
├── components/
│   ├── AppSettings.tsx        # 앱 설정 패널
│   ├── documents/             # 문서 관련 컴포넌트
│   ├── graph/                 # 그래프 관련 컴포넌트
│   ├── retrieval/             # 검색 관련 컴포넌트
│   └── status/                # 상태 표시 컴포넌트
├── locales/                   # 다국어 번역 파일
├── stores/                    # 전역 상태
├── hooks/                     # 커스텀 React 훅
└── services/                  # API 통신 서비스
```

---

## 10. 인증 및 보안

### 10.1 인증 방식

두 가지 인증 방식 지원:

1. **JWT 토큰 인증** (`auth.py`)
   - `POST /login` → JWT 토큰 발급
   - `Authorization: Bearer <token>` 헤더로 전달
   - 토큰 자동 갱신: `X-New-Token` 응답 헤더로 새 토큰 전달
   - 알고리즘 혼동 공격 방지: `none` 알고리즘 완전 차단

2. **API Key 인증**
   - `LIGHTRAG_API_KEY` 환경변수 설정
   - `Authorization: Bearer <api-key>` 헤더로 전달

### 10.2 보안 설정

```
# 환경변수 설정
AUTH_ACCOUNTS=admin:hashed_password,user:hashed_password
TOKEN_SECRET=your-secret-key  # 기본값 사용 금지 (필수)
LIGHTRAG_API_KEY=your-api-key
```

- bcrypt 기반 패스워드 해싱 (`lightrag-hash-password` CLI 도구)
- JWT 알고리즘 `none` bypass 공격 방지 (CVE: GHSA-8ffj-4hx4-9pgf)
- CORS 설정 (`CORS_ORIGINS` 환경변수)
- SSL/TLS 지원

---

## 11. 워크스페이스 격리

동일한 LightRAG 인스턴스에서 여러 독립 데이터 공간을 관리한다.

### 11.1 스토리지별 격리 방식

| 스토리지 타입 | 격리 방식 |
|---------------|---------|
| 파일 기반 (JSON, NanoVDB) | 하위 디렉토리 생성 |
| 컬렉션 기반 (Milvus, Qdrant) | 컬렉션 이름 접두사 |
| 관계형 DB (PostgreSQL) | workspace 컬럼 필터링 |
| Qdrant | 페이로드 기반 파티셔닝 |
| MongoDB | 컬렉션 이름 접두사 |

### 11.2 사용 방법

```python
# Python API
rag = LightRAG(workspace="project_alpha", ...)

# REST API (HTTP 헤더)
headers = {"LIGHTRAG-WORKSPACE": "project_alpha"}
```

---

## 12. 비동기 처리 아키텍처

### 12.1 LLM 요청 제한 (`priority_limit_async_func_call`)

```
LLM 요청 큐
├── 우선순위 큐 (heapq 기반)
├── 동시 실행 제한 (max_async)
├── 타임아웃 처리
└── 재시도 로직 (tenacity)
```

### 12.2 문서 삽입 병렬성

```python
max_parallel_insert = 2   # 동시 삽입 문서 수 (권장 최대: 10)
max_async = 16            # LLM 동시 요청 수
```

### 12.3 공유 스토리지 (`kg/shared_storage.py`)

Gunicorn 멀티워커 환경에서 프로세스 간 상태 공유:
- `get_namespace_data()` - 공유 데이터 접근
- `get_namespace_lock()` - 네임스페이스별 락
- `get_data_init_lock()` - 초기화 락

### 12.4 동시성 제어

```python
# 엔티티/관계 쓰기 시 sorted key로 데드락 방지
sorted_key = f"{sorted([src, tgt])[0]}-{sorted([src, tgt])[1]}"
async with get_storage_keyed_lock(sorted_key):
    await upsert_edge(...)
```

---

## 13. 설정 관리

### 13.1 설정 우선순위

```
OS 환경변수 > .env 파일 > CLI 인자 > 코드 기본값
```

### 13.2 주요 환경변수

```bash
# LLM 설정
LLM_BINDING=openai                    # openai|ollama|gemini|azure_openai|aws_bedrock|lollms
LLM_BINDING_HOST=https://api.openai.com/v1
LLM_MODEL=gpt-4o-mini
LLM_BINDING_API_KEY=sk-...

# 임베딩 설정
EMBEDDING_BINDING=openai
EMBEDDING_MODEL=text-embedding-3-large
EMBEDDING_DIM=1536
EMBEDDING_MAX_TOKEN_SIZE=8192

# 스토리지 설정
KV_STORAGE=JsonKVStorage
VECTOR_STORAGE=NanoVectorDBStorage
GRAPH_STORAGE=NetworkXStorage
DOC_STATUS_STORAGE=JsonDocStatusStorage

# 쿼리 기본값
TOP_K=40
CHUNK_TOP_K=20
MAX_ENTITY_TOKENS=6000
MAX_RELATION_TOKENS=8000
MAX_TOTAL_TOKENS=30000

# 리랭킹
ENABLE_RERANK=false
RERANK_BINDING=null
RERANK_MODEL=BAAI/bge-reranker-v2-m3

# 인증
AUTH_ACCOUNTS=admin:hashed_pw
TOKEN_SECRET=your-secret-key
LIGHTRAG_API_KEY=your-api-key

# 서버
HOST=0.0.0.0
PORT=9621
WORKERS=2
WORKSPACE=default
WORKING_DIR=./rag_storage
```

### 13.3 CLI 엔트리포인트

```bash
lightrag-server          # uvicorn 단일 프로세스
lightrag-gunicorn        # Gunicorn 멀티워커
lightrag-hash-password   # 패스워드 해시 생성
lightrag-download-cache  # 다운로드 캐시
lightrag-clean-llmqc     # LLM 쿼리 캐시 정리
```

---

## 14. 배포 옵션

### 14.1 개발 환경 (로컬 직접 실행)

```bash
# 의존성 설치
uv sync --extra api
uv pip install ollama          # Ollama 바인딩 별도 설치 필요

# WebUI 빌드 (bun 사용, npm/yarn 금지)
cd lightrag_webui
bun install --frozen-lockfile
bun run build
cd ..

# 서버 실행
lightrag-server
```

### 14.2 Docker 단일 컨테이너 (현재 운영 방식)

**`docker-compose.yml`** — 현재 사용 중인 구성:

```yaml
services:
  lightrag:
    image: ghcr.io/hkuds/lightrag:latest
    ports:
      - "${HOST:-0.0.0.0}:${PORT:-9621}:9621"
    volumes:
      - ./data/rag_storage:/app/data/rag_storage
      - ./data/inputs:/app/data/inputs
      - ./config.ini:/app/config.ini
      - ./.env:/app/.env                   # 설정 파일 마운트
    deploy:
      restart_policy:
        condition: on-failure
        max_attempts: 10
    extra_hosts:
      - "host.docker.internal:host-gateway" # 컨테이너 → 호스트 Ollama 접근
    environment:
      WORKING_DIR: "/app/data/rag_storage"
      INPUT_DIR: "/app/data/inputs"
      HOST: "0.0.0.0"
      PORT: "9621"
```

```bash
# 실행
docker-compose up -d

# 로그 확인
docker logs uiscloud_lightrag-lightrag-1 -f

# 재시작 (설정 변경 후)
docker-compose restart
```

> **주의**: `.env` 파일 수정 후 반드시 컨테이너를 재시작해야 새 설정이 적용됩니다.
> LightRAG는 `.env`를 **시작 시 1회** 읽으며 런타임에 재읽기하지 않습니다.

### 14.3 Docker Full Stack (GPU 포함)

**`docker-compose-full.yml`** — 8개 서비스 전체 구성:

```
서비스 구성:
├─ lightrag        (포트 9621) - LightRAG API + WebUI
├─ milvus          (포트 19530) - 벡터 DB (GPU)
├─ milvus-etcd     (내부) - Milvus 메타데이터
├─ milvus-minio    (포트 9000/9001) - Milvus 오브젝트 스토리지
├─ neo4j           (포트 7474/7687) - 그래프 DB
├─ postgres        (포트 5432) - PostgreSQL + AGE + pgvector
├─ vllm-embed      (포트 8001) - BAAI/bge-m3 임베딩
└─ vllm-rerank     (포트 8000) - bge-reranker-v2-m3 리랭킹
```

```bash
docker-compose -f docker-compose-full.yml up -d
```

### 14.4 Kubernetes

```
k8s-deploy/
├── lightrag/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
├── databases/
├── install_lightrag.sh
└── install_lightrag_dev.sh
```

### 14.5 Gunicorn 멀티워커 (프로덕션)

```bash
lightrag-gunicorn
# gunicorn_config.py: preload_app=True, workers=2
# 공유 스토리지를 통한 프로세스 간 상태 공유
```

### 14.6 현재 운영 환경의 LiteLLM Proxy 연동

DGX 서버의 LiteLLM Proxy를 통해 여러 모델을 단일 OpenAI 호환 엔드포인트로 제공:

```
LiteLLM Proxy (192.168.0.210:4000)
├─ Qwen3-30B-A3B      (LLM, API Key: sk-dgx-proxy)
├─ bge-m3             (임베딩)
└─ bge-reranker-v2-m3 (리랭킹)
```

Docker 컨테이너에서 호스트 Ollama 접근은 `host.docker.internal:11434`를 통해 이루어지며,
`extra_hosts: host.docker.internal:host-gateway` 설정으로 활성화됩니다.

---

## 15. 데이터 흐름 다이어그램

### 15.1 문서 삽입 흐름

```
사용자 입력
    │ "텍스트 또는 파일"
    ▼
LightRAG.ainsert()
    │
    ├─► MD5 해시 → doc_id 생성
    ├─► 중복 문서 체크 (DocStatus 스토리지)
    │
    ▼
apipeline_enqueue_documents()
    │ DocStatus: PENDING
    ▼
apipeline_process_enqueue_documents()
    │
    ├─► chunking_by_token_size()
    │       └─► [chunk_1, chunk_2, ..., chunk_n]
    │
    ├─► 청크별 병렬 처리 (max_parallel_insert)
    │       ├─► extract_entities() ← LLM 호출
    │       └─► merge_nodes_and_edges() ← LLM 호출 (필요시)
    │
    └─► 스토리지 일괄 저장
            ├─► KV: full_docs, text_chunks, full_entities, full_relations
            ├─► Vector: entities, relationships, chunks (임베딩 생성)
            └─► Graph: 노드/엣지 upsert

DocStatus: PROCESSED
```

### 15.2 쿼리 흐름 (mix 모드)

```
사용자 질문
    │
    ▼
LightRAG.aquery(mode="mix")
    │
    ▼
kg_query()
    │
    ├─[1]─► 키워드 추출 (LLM)
    │           ├─► high_level_keywords
    │           └─► low_level_keywords
    │
    ├─[2L]─► 엔티티 벡터 검색 (ll_keywords)
    │            └─► top_k 엔티티 후보
    │
    ├─[2G]─► 관계 벡터 검색 (hl_keywords)
    │            └─► top_k 관계 후보
    │
    ├─[3]──► 청크 벡터 검색 (질문 직접)
    │            └─► chunk_top_k 청크 후보
    │
    ├─[4]──► 그래프 탐색
    │            ├─► 엔티티 이웃 관계 조회
    │            └─► 관계 연결 청크 수집
    │
    ├─[5]──► 리랭킹 (enable_rerank=True 시)
    │            └─► 모든 청크 재정렬
    │
    ├─[6]──► 토큰 예산 내 컨텍스트 구성
    │            ├─► 엔티티 컨텍스트 (max_entity_tokens: 6000)
    │            ├─► 관계 컨텍스트 (max_relation_tokens: 8000)
    │            └─► 청크 컨텍스트 (잔여 토큰)
    │
    └─[7]──► LLM 최종 응답 생성
                 └─► 스트리밍 또는 일반 응답
```

---

## 부록: 주요 파일 참조

| 파일 | 역할 | 줄 수 |
|------|------|-------|
| `lightrag/lightrag.py` | 메인 오케스트레이터 | 4,456 |
| `lightrag/operate.py` | 핵심 처리 로직 | 5,197 |
| `lightrag/utils.py` | 유틸리티 | 2,856+ |
| `lightrag/api/lightrag_server.py` | API 서버 | 1,548 |
| `lightrag/api/routers/document_routes.py` | 문서 라우터 | ~2,100 |
| `lightrag/kg/postgres_impl.py` | PostgreSQL 구현체 | ~4,700 |
| `lightrag/base.py` | 추상 기반 클래스 | ~900 |
| `lightrag/prompt.py` | LLM 프롬프트 템플릿 | ~400 |
| `lightrag/constants.py` | 설정 상수 | ~150 |
| `lightrag/namespace.py` | 스토리지 네임스페이스 | ~30 |

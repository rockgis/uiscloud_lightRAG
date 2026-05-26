# uiscloud LightRAG 사용자 가이드

> 버전: 1.4.16 | 작성일: 2026-05-26

---

## 목차

1. [서비스 소개](#1-서비스-소개)
2. [접속 방법](#2-접속-방법)
3. [문서 관리](#3-문서-관리)
4. [질의 검색](#4-질의-검색)
5. [쿼리 모드 상세](#5-쿼리-모드-상세)
6. [지식 그래프 탐색](#6-지식-그래프-탐색)
7. [HypermakinaRAG UI 연동](#7-hypermakinarag-ui-연동)
8. [API 직접 사용](#8-api-직접-사용)
9. [운영 현황](#9-운영-현황)
10. [자주 묻는 질문](#10-자주-묻는-질문)

---

## 1. 서비스 소개

**uiscloud LightRAG**는 지식 그래프(Knowledge Graph) 기반의 RAG(Retrieval-Augmented Generation) 시스템입니다.

### 일반 벡터 RAG와의 차이점

| 항목 | 일반 벡터 RAG | LightRAG |
|------|-------------|---------|
| 검색 방식 | 청크 유사도 검색 | 엔티티·관계 그래프 + 벡터 통합 |
| 맥락 이해 | 단편적 | 개념 간 연결 관계 추론 |
| 복잡한 질문 | 약함 | 강함 (다단계 추론 가능) |
| 인덱싱 속도 | 빠름 | 느림 (LLM 추출 필요) |
| 시각화 | 없음 | 지식 그래프 시각화 |

### 핵심 개념

```
문서 업로드
    │
    ▼
[LLM이 엔티티·관계 추출]
    예) "홍길동(사람) → 근무 → ABC회사(조직)"
    │
    ▼
[지식 그래프 구성]
    노드: 엔티티 (사람, 조직, 장소, 개념 등)
    엣지: 관계 (근무, 소속, 영향, 연관 등)
    │
    ▼
[질의 시 그래프 탐색 + 벡터 검색]
    질문 키워드 → 관련 엔티티 탐색 → 연결된 청크 수집 → LLM 답변 생성
```

---

## 2. 접속 방법

### LightRAG 전용 WebUI

| 항목 | 주소 |
|------|------|
| **WebUI** | http://localhost:9621 |
| **API 문서** | http://localhost:9621/docs |
| **인증** | 없음 (로컬 개발 모드) |

### HypermakinaRAG UI (통합 채팅)

| 항목 | 주소 |
|------|------|
| **HypermakinaRAG UI** | http://localhost |
| **이메일** | hypermakina@gmail.com |
| **비밀번호** | Admin123! |

---

## 3. 문서 관리

### 3.1 WebUI에서 문서 업로드

1. http://localhost:9621 접속
2. 상단 탭 **Documents** 클릭
3. 우측 상단 **Upload** 버튼 클릭
4. 파일 선택 (지원 형식: PDF, DOCX, PPTX, XLSX, TXT, MD)

### 3.2 텍스트 직접 입력

1. Documents 탭 → **Text** 버튼 클릭
2. 제목과 내용 입력
3. **Insert** 클릭

### 3.3 폴더 스캔 방식 (대량 업로드 권장)

```bash
# 파일을 inputs 폴더에 복사
cp 문서파일.pdf /Volumes/dev/workspace/uiscloud_workspace/uiscloud_lightRAG/data/inputs/

# WebUI에서 Scan 클릭 또는 API 호출
curl -X POST http://localhost:9621/documents/scan
```

### 3.4 문서 처리 상태

업로드 후 LLM이 엔티티·관계를 추출하는 과정이 자동으로 진행됩니다.

| 상태 | 의미 |
|------|------|
| **PENDING** | 처리 대기 중 |
| **PROCESSING** | LLM이 엔티티·관계 추출 중 |
| **PROCESSED** | 처리 완료 (질의 가능) |
| **FAILED** | 처리 실패 (재처리 가능) |

> **처리 시간**: 문서 크기·복잡도에 따라 수 분 소요됩니다. `PROCESSED` 상태가 될 때까지 기다린 후 질의하세요.

### 3.5 처리 실패 문서 재처리

Documents 탭 하단 **Reprocess Failed** 버튼 클릭, 또는:

```bash
curl -X POST http://localhost:9621/documents/reprocess_failed
```

### 3.6 문서 삭제

- 특정 문서: Documents 탭에서 문서 선택 → 삭제 아이콘
- 전체 삭제: **Clear** 버튼 (주의: 지식 그래프 전체 초기화)

---

## 4. 질의 검색

### 4.1 WebUI Retrieval 탭

1. 상단 탭 **Retrieval** 클릭
2. 하단 입력창에 질문 입력
3. **Enter** 또는 **Send** 버튼 클릭

```
입력 단축키:
  Enter       → 질문 전송
  Shift+Enter → 줄바꿈 (여러 줄 질문)
```

### 4.2 슬래시 모드 접두사

질문 앞에 슬래시 명령어를 붙이면 즉시 검색 모드를 변경할 수 있습니다:

```
/mix 질문     → KG + 벡터 통합 검색 (기본값, 권장)
/hybrid 질문  → local + global 결합 검색
/local 질문   → 특정 엔티티 중심 검색
/global 질문  → 전체 주제·트렌드 파악
/naive 질문   → 단순 벡터 유사도 검색
/bypass 질문  → RAG 없이 LLM에 직접 질문
```

**예시:**
```
/local 홍길동의 직책과 담당 업무는 무엇인가요?
/global 이 문서들의 핵심 주제와 전반적인 경향은?
/naive 2024년 매출 현황
```

### 4.3 우측 Query Settings 패널

| 설정 항목 | 기본값 | 설명 |
|---------|--------|------|
| **User Prompt** | 없음 | LLM에 추가 지시사항 입력 |
| **Query Mode** | mix | 검색 모드 선택 (슬래시로도 변경 가능) |
| **Top K** | 40 | KG 엔티티·관계 검색 수 |
| **Chunk Top K** | 20 | 텍스트 청크 검색 수 |
| **Max Entity Tokens** | 6,000 | 엔티티 컨텍스트 최대 토큰 |
| **Max Relation Tokens** | 8,000 | 관계 컨텍스트 최대 토큰 |
| **Max Total Tokens** | 30,000 | 전체 컨텍스트 최대 토큰 |
| **Enable Rerank** | OFF | 리랭킹 적용 여부 |
| **Only Need Context** | OFF | LLM 없이 검색 컨텍스트만 반환 |
| **Stream** | OFF | 스트리밍 응답 활성화 |

**User Prompt 활용 예:**
```
항상 한국어로 답변하고, 근거가 된 문서 제목을 인용해줘.
답변은 bullet point 형식으로 구성해줘.
```

---

## 5. 쿼리 모드 상세

### 5.1 모드별 비교

| 모드 | 검색 방식 | 최적 활용 상황 | 속도 |
|------|---------|--------------|------|
| **mix** | KG + 벡터 통합 | 일반적인 모든 질문 (권장) | 중간 |
| **hybrid** | local + global 결합 | 개념 + 전체 맥락이 모두 필요한 질문 | 느림 |
| **local** | 엔티티 중심 그래프 탐색 | "A는 무엇인가?", "A와 B의 관계?" | 빠름 |
| **global** | 커뮤니티·관계 전체 분석 | "전반적인 주제는?", "주요 트렌드는?" | 느림 |
| **naive** | 단순 청크 벡터 검색 | 키워드 검색, 특정 문장 찾기 | 매우 빠름 |
| **bypass** | RAG 없이 LLM 직접 호출 | 지식베이스와 무관한 일반 질문 | 빠름 |

### 5.2 모드 선택 기준

```
질문 유형별 추천 모드:

"X란 무엇인가?" / "X에 대해 설명해줘"
  → local 또는 mix

"X와 Y의 관계는?" / "X가 Y에 미치는 영향은?"
  → local 또는 hybrid

"전체적인 주제는?" / "핵심 내용을 요약해줘"
  → global 또는 mix

"2024년 매출이 얼마야?" (특정 수치 검색)
  → naive 또는 mix

"오늘 날씨 어때?" (지식베이스 무관)
  → bypass
```

### 5.3 thinking 태그 처리

Qwen3 모델 사용 시 `<think>...</think>` 태그가 출력될 수 있습니다.
WebUI는 이를 자동으로 파싱하여 추론 과정과 최종 답변을 분리해 표시합니다.

---

## 6. 지식 그래프 탐색

### 6.1 GraphViewer 탭

1. 상단 탭 **Graph** 클릭
2. 문서 처리가 완료된 엔티티가 노드로 표시
3. 엔티티 간 관계가 엣지로 연결

### 6.2 그래프 조작

| 기능 | 방법 |
|------|------|
| **확대/축소** | 마우스 휠 |
| **이동** | 드래그 |
| **노드 선택** | 클릭 |
| **노드 상세 보기** | 노드 클릭 → 우측 패널 |
| **관계 탐색** | 엣지 클릭 |
| **검색** | 상단 검색창에 엔티티명 입력 |
| **레이아웃 변경** | 우측 상단 Layout 버튼 |
| **전체화면** | 우측 상단 Fullscreen |

### 6.3 엔티티 편집

노드 클릭 → 우측 패널:
- 엔티티 이름·설명 수정
- 엔티티 삭제
- 연결된 관계 확인

### 6.4 그래프 레이블 필터

좌측 패널에서 특정 레이블(엔티티 유형)만 선택하여 표시할 수 있습니다.

---

## 7. HypermakinaRAG UI 연동

### 7.1 개요

http://localhost (HypermakinaRAG UI)를 통해 LightRAG를 사용할 수 있습니다.
HypermakinaRAG의 채팅 인터페이스(대화 히스토리, 세션 관리)를 활용하면서
LightRAG의 지식 그래프 기반 검색을 백엔드로 사용합니다.

```
HypermakinaRAG UI (http://localhost)
    │
    │  Ollama 호환 API 호출
    ▼
LightRAG (http://host.docker.internal:9621/api/chat)
    │
    │  지식 그래프 기반 RAG 수행
    ▼
Qwen3-30B-A3B (LiteLLM Proxy)
```

### 7.2 로그인

```
URL: http://localhost
이메일: hypermakina@gmail.com
비밀번호: Admin123!
```

### 7.3 LightRAG 에이전트 사용

1. 좌측 메뉴 **채팅** 클릭
2. 채팅 상단 **에이전트 선택** → **"LightRAG 지식 그래프 검색"** 선택
3. 질문 입력

> 에이전트를 선택하지 않으면 HypermakinaRAG 자체 RAG가 사용됩니다.

### 7.4 슬래시 모드 (HypermakinaRAG에서도 동일 적용)

HypermakinaRAG 채팅에서도 LightRAG 슬래시 접두사가 동일하게 동작합니다:

```
/local 특정 인물의 직책은?
/global 전체 문서의 핵심 주제는?
/bypass 일반적인 질문 (RAG 없이)
```

### 7.5 주의사항

- LightRAG와 HypermakinaRAG는 **별도의 지식 베이스**를 사용합니다
- LightRAG에 업로드한 문서가 HypermakinaRAG 지식 베이스에 자동 동기화되지 않습니다
- 문서는 **LightRAG(포트 9621)**에 업로드해야 LightRAG 에이전트가 활용합니다

---

## 8. API 직접 사용

인증 없이 API를 직접 호출할 수 있습니다 (로컬 개발 모드).

### 8.1 문서 업로드

```bash
# 텍스트 삽입
curl -X POST http://localhost:9621/documents/text \
  -H "Content-Type: application/json" \
  -d '{"text": "삽입할 텍스트 내용", "description": "문서 설명"}'

# 파일 업로드
curl -X POST http://localhost:9621/documents/upload \
  -F "file=@문서파일.pdf"

# 폴더 스캔
curl -X POST http://localhost:9621/documents/scan
```

### 8.2 문서 목록 조회

```bash
# 전체 목록
curl http://localhost:9621/documents

# 페이지 조회
curl -X POST http://localhost:9621/documents/paginated \
  -H "Content-Type: application/json" \
  -d '{"page": 1, "page_size": 20}'

# 처리 상태 통계
curl http://localhost:9621/documents/status_counts
```

### 8.3 질의 (표준)

```bash
# 기본 질의 (mix 모드)
curl -X POST http://localhost:9621/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "질문 내용을 여기에 입력",
    "mode": "mix"
  }'

# 상세 옵션 지정
curl -X POST http://localhost:9621/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "질문 내용",
    "mode": "hybrid",
    "top_k": 60,
    "chunk_top_k": 30,
    "max_entity_tokens": 8000,
    "max_relation_tokens": 10000,
    "max_total_tokens": 32000,
    "stream": false
  }'
```

### 8.4 질의 (스트리밍)

```bash
# 스트리밍 응답 (NDJSON 형식)
curl -X POST http://localhost:9621/query/stream \
  -H "Content-Type: application/json" \
  -d '{
    "query": "질문 내용",
    "mode": "mix",
    "stream": true
  }'
```

### 8.5 검색 컨텍스트만 조회 (LLM 호출 없음)

```bash
curl -X POST http://localhost:9621/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "질문 내용",
    "mode": "mix",
    "only_need_context": true
  }'
```

### 8.6 지식 그래프 조회

```bash
# 인기 레이블 목록
curl "http://localhost:9621/graph/label/popular?limit=20"

# 레이블 목록 전체
curl http://localhost:9621/graph/label/list

# 지식 그래프 데이터 (특정 레이블)
curl "http://localhost:9621/graphs?label=person&max_depth=2&max_nodes=50"
```

### 8.7 엔티티 관리

```bash
# 엔티티 존재 여부 확인
curl "http://localhost:9621/graph/entity/exists?entity_name=홍길동"

# 엔티티 생성
curl -X POST http://localhost:9621/graph/entity/create \
  -H "Content-Type: application/json" \
  -d '{"entity_name": "홍길동", "description": "주인공", "entity_type": "Person"}'

# 엔티티 수정
curl -X POST http://localhost:9621/graph/entity/edit \
  -H "Content-Type: application/json" \
  -d '{"entity_name": "홍길동", "description": "수정된 설명"}'

# 엔티티 삭제
curl -X DELETE "http://localhost:9621/documents/delete_entity?entity_name=홍길동"
```

### 8.8 Ollama 호환 API

다른 시스템과 연동 시 Ollama 호환 엔드포인트를 사용합니다:

```bash
# 사용 가능한 모델 목록
curl http://localhost:9621/api/tags

# 채팅 (스트리밍)
curl http://localhost:9621/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "lightrag:latest",
    "messages": [{"role": "user", "content": "/mix 질문 내용"}],
    "stream": true
  }'

# 텍스트 생성
curl http://localhost:9621/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "lightrag:latest",
    "prompt": "/local 질문 내용",
    "stream": false
  }'
```

---

## 9. 운영 현황

### 9.1 서비스 상태 확인

```bash
# 헬스 체크
curl http://localhost:9621/health

# 파이프라인 상태 (인덱싱 중 여부)
curl http://localhost:9621/documents/pipeline_status
```

### 9.2 현재 설정값

| 항목 | 값 |
|------|-----|
| **LLM** | Qwen3-30B-A3B (LiteLLM Proxy → 192.168.0.210:4000) |
| **Embedding** | bge-m3:latest (Ollama → localhost:11434, 1024 dim) |
| **KV 스토리지** | JsonKVStorage (파일 기반) |
| **벡터 스토리지** | NanoVectorDBStorage (파일 기반) |
| **그래프 스토리지** | NetworkXStorage (파일 기반) |
| **데이터 경로** | `./data/rag_storage/` |
| **입력 경로** | `./data/inputs/` |
| **청크 크기** | 1,200 tokens |
| **청크 오버랩** | 100 tokens |
| **요약 언어** | Korean |
| **LLM 캐시** | 활성화 |
| **최대 병렬 삽입** | 2개 |
| **코사인 임계값** | 0.2 |

### 9.3 Docker 컨테이너 관리

```bash
# 서비스 상태 확인
docker ps | grep lightrag

# 로그 확인
docker logs uiscloud_lightrag-lightrag-1 -f

# 설정 변경 후 재시작
docker-compose restart

# 완전 재시작
docker-compose down && docker-compose up -d
```

> ⚠️ `.env` 파일 수정 후에는 반드시 컨테이너를 재시작해야 새 설정이 적용됩니다.

### 9.4 LLM 캐시 관리

반복 질의에 대해 LLM 호출 없이 캐시를 반환합니다.
캐시를 초기화하려면:

```bash
curl -X POST http://localhost:9621/documents/clear_cache
```

---

## 10. 자주 묻는 질문

### Q. 문서를 업로드했는데 질의 결과가 없어요

**A.** 문서 처리(`PROCESSED`) 상태를 먼저 확인하세요.
Documents 탭에서 상태 아이콘을 확인하거나:
```bash
curl http://localhost:9621/documents/status_counts
```
`PROCESSING` 상태라면 LLM이 엔티티 추출 중이므로 완료될 때까지 기다리세요.

---

### Q. 답변 품질이 낮아요

**A.** 다음을 시도해보세요:
1. **모드 변경**: `mix` → `hybrid` 또는 `local`
2. **Top K 증가**: Query Settings에서 Top K를 60~80으로 늘리기
3. **User Prompt 추가**: "근거를 인용하고 구체적으로 답변해줘"
4. **더 구체적인 질문**: 모호한 질문보다 구체적인 엔티티명 포함

---

### Q. 처리 속도가 느려요

**A.** LightRAG는 인덱싱 시 LLM을 사용하므로 문서당 수 분이 소요됩니다.
처리 속도를 높이려면 `.env`에서:
```
MAX_PARALLEL_INSERT=4   # 기본 2 (최대 권장: 10)
MAX_ASYNC=8             # 기본 4
```
단, 너무 높은 값은 LLM API 과부하를 유발할 수 있습니다.

---

### Q. 임베딩 모델을 변경하면 어떻게 되나요?

**A.** 임베딩 모델 변경 시 기존 벡터 데이터와 차원이 맞지 않아 오류가 발생합니다.
변경 전에 반드시 `./data/rag_storage/` 내 벡터 파일을 삭제하고
모든 문서를 재인덱싱해야 합니다.

---

### Q. LLM 캐시를 비우고 싶어요

**A.** LLM 응답 캐시만 삭제하려면:
```bash
curl -X POST http://localhost:9621/documents/clear_cache
```
또는 `./data/rag_storage/kv_store_llm_response_cache.json` 파일을 삭제하세요.

---

### Q. 특정 문서만 삭제하고 싶어요

**A.** Documents 탭에서 문서 옆 삭제 버튼을 누르거나:
```bash
# 문서 ID 확인
curl http://localhost:9621/documents

# 특정 문서 삭제 (해당 문서의 엔티티·관계·청크 모두 제거)
curl -X DELETE "http://localhost:9621/documents/delete_document?doc_id=문서ID"
```

---

### Q. HypermakinaRAG에서 LightRAG 에이전트가 보이지 않아요

**A.** 좌측 메뉴 채팅 → 새 채팅 시작 → 상단의 에이전트 선택 드롭다운에서
**"LightRAG 지식 그래프 검색"**을 찾아보세요.
보이지 않는다면 http://localhost:8085 (백엔드 API)가 정상 동작 중인지 확인하세요.

---

## 부록: 주요 URL 목록

| 서비스 | URL | 용도 |
|--------|-----|------|
| LightRAG WebUI | http://localhost:9621 | 문서 관리, 질의, 그래프 탐색 |
| LightRAG API 문서 | http://localhost:9621/docs | REST API 명세 |
| LightRAG Health | http://localhost:9621/health | 서비스 상태 확인 |
| HypermakinaRAG | http://localhost | 통합 채팅 UI |
| HypermakinaRAG API | http://localhost:8085 | HypermakinaRAG 백엔드 |

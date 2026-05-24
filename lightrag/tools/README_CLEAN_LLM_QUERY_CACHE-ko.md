# LLM 쿼리 캐시 정리 도구 - 사용자 가이드

## 개요

이 도구는 KV 스토리지 구현체에서 LightRAG의 LLM 쿼리 캐시를 정리합니다. RAG 쿼리 작업(모드: `mix`, `hybrid`, `local`, `global`) 중에 생성된 쿼리 캐시와 키워드 캐시를 대상으로 합니다.

## 지원 스토리지 타입

1. **JsonKVStorage** - 파일 기반 JSON 스토리지
2. **RedisKVStorage** - Redis 데이터베이스 스토리지
3. **PGKVStorage** - PostgreSQL 데이터베이스 스토리지
4. **MongoKVStorage** - MongoDB 데이터베이스 스토리지
5. **OpenSearchKVStorage** - OpenSearch 인덱스 스토리지

## 캐시 타입

이 도구는 다음 쿼리 캐시 타입을 정리합니다:

### 쿼리 캐시 모드 (4종)
- `mix:*` - 혼합 모드 쿼리 캐시
- `hybrid:*` - 하이브리드 모드 쿼리 캐시
- `local:*` - 로컬 모드 쿼리 캐시
- `global:*` - 글로벌 모드 쿼리 캐시

### 캐시 내용 타입 (2종)
- `*:query:*` - 쿼리 결과 캐시
- `*:keywords:*` - 키워드 추출 캐시

### 캐시 키 형식
```
<모드>:<캐시_타입>:<해시>
```

예시:
- `mix:query:5ce04d25e957c290216cee5bfe6344fa`
- `mix:keywords:fee77b98244a0b047ce95e21060de60e`
- `global:query:abc123def456...`
- `local:keywords:789xyz...`

**중요 참고사항**: 이 도구는 추출 캐시(`default:extract:*` 및 `default:summary:*`)를 정리하지 않습니다. 해당 캐시는 마이그레이션 도구 또는 수동 삭제를 사용하세요.

## 사전 요구사항

- 이 도구는 환경 변수에서 스토리지 설정을 읽습니다
- 대상 스토리지가 올바르게 설정되고 접근 가능한지 확인하세요
- 정리 작업을 실행하기 전에 중요한 데이터를 백업하세요

## 사용법

### 기본 사용법

LightRAG 프로젝트 루트 디렉터리에서 실행:

```bash
python -m lightrag.tools.clean_llm_query_cache
# 또는
python lightrag/tools/clean_llm_query_cache.py
```

### 대화형 워크플로우

도구는 다음 단계를 안내합니다:

#### 1. 스토리지 타입 선택
```
============================================================
LLM Query Cache Cleanup Tool - LightRAG
============================================================

=== Storage Setup ===

Supported KV Storage Types:
[1] JsonKVStorage
[2] RedisKVStorage
[3] PGKVStorage
[4] MongoKVStorage
[5] OpenSearchKVStorage

Select storage type (1-5) (Press Enter to exit): 1
```

**참고**: 언제든지 Enter 키를 누르거나 `0`을 입력하면 정상적으로 종료됩니다.

#### 2. 스토리지 검증

도구가 다음을 수행합니다:
- 필수 환경 변수 확인
- 워크스페이스 설정 자동 감지
- 스토리지 초기화 및 연결
- 연결 상태 확인

```
Checking configuration...
✓ All required environment variables are set

Initializing storage...
- Storage Type: JsonKVStorage
- Workspace: space1
- Connection Status: ✓ Success
```

#### 3. 캐시 통계 확인

도구는 모드 및 타입별 쿼리 캐시 세부 정보를 표시합니다:

```
Counting query cache records...

📊 Query Cache Statistics (Before Cleanup):
┌────────────┬────────────┬────────────┬────────────┐
│ Mode       │ Query      │ Keywords   │ Total      │
├────────────┼────────────┼────────────┼────────────┤
│ mix        │      1,234 │        567 │      1,801 │
│ hybrid     │        890 │        423 │      1,313 │
│ local      │      2,345 │      1,123 │      3,468 │
│ global     │        678 │        345 │      1,023 │
├────────────┼────────────┼────────────┼────────────┤
│ Total      │      5,147 │      2,458 │      7,605 │
└────────────┴────────────┴────────────┴────────────┘
```

#### 4. 정리 범위 선택

삭제할 캐시 타입을 선택합니다:

```
=== Cleanup Options ===
[1] Delete all query caches (both query and keywords)
[2] Delete query caches only (keep keywords)
[3] Delete keywords caches only (keep query)
[0] Cancel

Select cleanup option (0-3): 1
```

**정리 타입:**
- **옵션 1 (전체)**: 모든 모드에 걸쳐 쿼리 및 키워드 캐시 모두 삭제
- **옵션 2 (쿼리만)**: 쿼리 캐시만 삭제, 키워드 캐시 보존
- **옵션 3 (키워드만)**: 키워드 캐시만 삭제, 쿼리 캐시 보존

#### 5. 삭제 확인

정리 계획을 검토하고 확인합니다:

```
============================================================
Cleanup Confirmation
============================================================
Storage: JsonKVStorage (workspace: space1)
Cleanup Type: all
Records to Delete: 7,605 / 7,605

⚠️  WARNING: This will delete ALL query caches across all modes!

Continue with deletion? (y/n): y
```

#### 6. 정리 실행

도구는 실시간 진행 상황과 함께 배치 삭제를 수행합니다:

**JsonKVStorage 예시:**
```
=== Starting Cleanup ===
💡 Processing 1,000 records at a time from JsonKVStorage

Batch 1/8: ████░░░░░░░░░░░░░░░░ 1,000/7,605 (13.1%) ✓
Batch 2/8: ████████░░░░░░░░░░░░ 2,000/7,605 (26.3%) ✓
...
Batch 8/8: ████████████████████ 7,605/7,605 (100.0%) ✓

Persisting changes to storage...
✓ Changes persisted successfully
```

**RedisKVStorage 예시:**
```
=== Starting Cleanup ===
💡 Processing Redis keys in batches of 1,000

Batch 1: Deleted 1,000 keys (Total: 1,000) ✓
Batch 2: Deleted 1,000 keys (Total: 2,000) ✓
...
```

**PostgreSQL 예시:**
```
=== Starting Cleanup ===
💡 Executing PostgreSQL DELETE query

✓ Deleted 7,605 records in 0.45s
```

**MongoDB 예시:**
```
=== Starting Cleanup ===
💡 Executing MongoDB deleteMany operations

Pattern 1/8: Deleted 1,234 records ✓
Pattern 2/8: Deleted 567 records ✓
...
Total deleted: 7,605 records
```

**OpenSearchKVStorage 예시:**
```
=== Starting Cleanup ===
💡 Processing 1,000 records at a time from OpenSearchKVStorage

Batch 1/8: ████░░░░░░░░░░░░░░░░ 1,000/7,605 (13.1%) ✓
Batch 2/8: ████████░░░░░░░░░░░░ 2,000/7,605 (26.3%) ✓
...
```

#### 7. 정리 보고서 검토

도구는 포괄적인 최종 보고서를 제공합니다:

**성공적인 정리:**
```
============================================================
Cleanup Complete - Final Report
============================================================

📊 Statistics:
  Total records to delete:  7,605
  Total batches:            8
  Successful batches:       8
  Failed batches:           0
  Successfully deleted:     7,605
  Failed to delete:         0
  Success rate:             100.00%

📈 Before/After Comparison:
  Total caches before:      7,605
  Total caches after:       0
  Net reduction:            7,605

============================================================
✓ SUCCESS: All records cleaned up successfully!
============================================================

📊 Query Cache Statistics (After Cleanup):
┌────────────┬────────────┬────────────┬────────────┐
│ Mode       │ Query      │ Keywords   │ Total      │
├────────────┼────────────┼────────────┼────────────┤
│ mix        │          0 │          0 │          0 │
│ hybrid     │          0 │          0 │          0 │
│ local      │          0 │          0 │          0 │
│ global     │          0 │          0 │          0 │
├────────────┼────────────┼────────────┼────────────┤
│ Total      │          0 │          0 │          0 │
└────────────┴────────────┴────────────┴────────────┘
```

**오류가 있는 정리:**
```
============================================================
Cleanup Complete - Final Report
============================================================

📊 Statistics:
  Total records to delete:  7,605
  Total batches:            8
  Successful batches:       7
  Failed batches:           1
  Successfully deleted:     6,605
  Failed to delete:         1,000
  Success rate:             86.85%

📈 Before/After Comparison:
  Total caches before:      7,605
  Total caches after:       1,000
  Net reduction:            6,605

⚠️  Errors encountered: 1

Error Details:
------------------------------------------------------------

Error Summary:
  - ConnectionError: 1 occurrence(s)

First 5 errors:

  1. Batch 3
     Type: ConnectionError
     Message: Connection timeout after 30s
     Records lost: 1,000

============================================================
⚠️  WARNING: Cleanup completed with errors!
   Please review the error details above.
============================================================
```

## 기술 세부사항

### 워크스페이스 처리

도구는 다음 우선순위로 워크스페이스를 가져옵니다:

1. **스토리지별 워크스페이스 환경 변수**
   - PGKVStorage: `POSTGRES_WORKSPACE`
   - MongoKVStorage: `MONGODB_WORKSPACE`
   - RedisKVStorage: `REDIS_WORKSPACE`
   - OpenSearchKVStorage: `OPENSEARCH_WORKSPACE`

2. **일반 워크스페이스 환경 변수**
   - `WORKSPACE`

3. **기본값**
   - 빈 문자열 (스토리지의 기본 워크스페이스 사용)

### 배치 삭제

- 기본 배치 크기: 1,000 레코드/배치
- 메모리 오버플로 및 연결 타임아웃 방지
- 각 배치는 독립적으로 처리됨
- 실패한 배치는 기록되지만 정리를 중단하지 않음

### 스토리지별 삭제 전략

#### JsonKVStorage
- 모든 일치하는 키를 먼저 수집 (스냅샷 방식)
- 잠금 보호와 함께 배치로 삭제
- 빠른 인메모리 작업

#### RedisKVStorage
- 패턴 매칭과 함께 SCAN 사용
- 배치 작업을 위한 파이프라인 DELETE
- 대용량 데이터셋을 위한 커서 기반 반복

#### PostgreSQL
- OR 조건을 포함한 단일 DELETE 쿼리
- 효율적인 서버 사이드 대량 삭제
- 모드/타입 매칭을 위한 LIKE 패턴 사용

#### MongoDB
- 여러 deleteMany 작업 (패턴당 하나)
- 정규식 기반 문서 매칭
- 정확한 삭제 개수 반환

### 패턴 매칭 구현

**JsonKVStorage:**
```python
# 직접 키 접두사 매칭
if key.startswith("mix:query:") or key.startswith("mix:keywords:")
```

**RedisKVStorage:**
```python
# 네임스페이스 접두사 패턴으로 SCAN
pattern = f"{namespace}:mix:query:*"
cursor, keys = await redis.scan(cursor, match=pattern)
```

**PostgreSQL:**
```python
# SQL LIKE 조건
WHERE id LIKE 'mix:query:%' OR id LIKE 'mix:keywords:%'
```

**MongoDB:**
```python
# _id 필드에 대한 정규식 쿼리
{"_id": {"$regex": "^mix:query:"}}
```

**OpenSearchKVStorage:**
```python
# 원시 히트를 스캔한 후 Python에서 캐시 키 접두사 매칭
if hit["_id"].startswith("mix:query:"):
```

## 오류 처리 및 복원력

도구는 포괄적인 오류 추적을 구현합니다:

### 배치 수준 오류 추적
- 각 배치는 독립적으로 오류 검사됨
- 실패한 배치는 전체 세부사항과 함께 기록됨
- 성공한 배치는 나중 배치가 실패해도 커밋됨
- 실시간 진행 상황에 ✓ (성공) 또는 ✗ (실패) 표시

### 오류 보고
정리 완료 후 상세 보고서 포함:
- **통계**: 총 레코드 수, 성공/실패 횟수, 성공률
- **정리 전/후 비교**: 캐시 수의 순 감소량
- **오류 요약**: 오류 타입별 그룹화 및 발생 횟수
- **오류 세부사항**: 배치 번호, 오류 타입, 메시지, 손실된 레코드 수
- **권장사항**: 성공 또는 검토 필요 여부 명확히 표시

### 검증
- 정리 후 카운트 검증
- 정리 전/후 통계 비교
- 부분 정리 시나리오 식별

## 중요 참고사항

1. **되돌릴 수 없는 작업**
   - 삭제된 캐시는 복구할 수 없음
   - 정리 전에 항상 중요한 데이터 백업
   - 먼저 비프로덕션 데이터에서 테스트

2. **성능 영향**
   - 정리 후 일시적으로 쿼리 성능 저하될 수 있음
   - 이후 쿼리에서 캐시가 재구축됨
   - 사용량이 적은 시간에 정리 고려

3. **선택적 정리**
   - 정리 범위를 신중하게 선택
   - 키워드 캐시는 향후 쿼리에 유용할 수 있음
   - 쿼리 캐시는 키워드 캐시보다 빠르게 재구축됨

4. **워크스페이스 격리**
   - 정리는 선택한 워크스페이스에만 영향
   - 다른 워크스페이스는 영향받지 않음
   - 정리 확인 전에 워크스페이스 확인

5. **중단 및 재시작**
   - 언제든지 정리 중단 가능 (Ctrl+C)
   - 이미 삭제된 레코드는 복구 불가
   - 자동 재개 없음 - 도구를 다시 실행해야 함

## 스토리지 설정

도구는 다음 우선순위로 여러 설정 방법을 지원합니다:

1. **환경 변수** (최고 우선순위)
2. **기본값** (최저 우선순위)

### 환경 변수 설정

`.env` 파일에서 스토리지 설정 구성:

#### 워크스페이스 설정 (선택사항)

```bash
# 일반 워크스페이스 (모든 스토리지에서 공유)
WORKSPACE=space1

# 또는 특정 스토리지에 대한 독립 워크스페이스 설정
POSTGRES_WORKSPACE=pg_space
MONGODB_WORKSPACE=mongo_space
REDIS_WORKSPACE=redis_space
```

**워크스페이스 우선순위**: 스토리지별 > 일반 WORKSPACE > 빈 문자열

#### JsonKVStorage

```bash
WORKING_DIR=./rag_storage
```

#### RedisKVStorage

```bash
REDIS_URI=redis://localhost:6379
```

#### PGKVStorage

```bash
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=your_username
POSTGRES_PASSWORD=your_password
POSTGRES_DATABASE=your_database
```

#### MongoKVStorage

```bash
MONGO_URI=mongodb://root:root@localhost:27017/
MONGO_DATABASE=LightRAG
```

#### OpenSearchKVStorage

```bash
OPENSEARCH_HOSTS=localhost:9200
OPENSEARCH_WORKSPACE=search_space
```

환경 변수가 제공되지 않으면 도구는 사용 가능한 내장 기본값으로 폴백됩니다.

## 문제 해결

### 환경 변수 누락
```
⚠️  Warning: Missing environment variables: POSTGRES_USER, POSTGRES_PASSWORD
```
**해결책**: `.env` 파일에 누락된 변수 추가

### 연결 실패
```
✗ Initialization failed: Connection refused
```
**해결책**:
- 데이터베이스 서비스가 실행 중인지 확인
- 연결 매개변수 확인 (호스트, 포트, 자격증명)
- 방화벽 설정 확인
- 원격 데이터베이스의 네트워크 연결 확인

### 캐시를 찾을 수 없음
```
⚠️  No query caches found in storage
```
**가능한 원인**:
- 아직 쿼리가 실행되지 않음
- 캐시가 이미 정리됨
- 잘못된 워크스페이스 선택
- 쿼리에 다른 스토리지 타입 사용됨

### 부분 정리
```
⚠️  WARNING: Cleanup completed with errors!
```
**해결책**:
- 보고서의 오류 세부사항 확인
- 스토리지 연결 안정성 확인
- 도구를 다시 실행하여 남은 캐시 정리
- 스토리지 용량 및 권한 확인

## 사용 사례

### 사용 사례 1: 모든 쿼리 캐시 정리

**시나리오**: 모든 쿼리 캐시를 제거하여 스토리지 공간 확보

```bash
# 도구 실행
python -m lightrag.tools.clean_llm_query_cache

# 선택: 스토리지 타입 -> 옵션 1 (전체) -> 확인 (y)
```

**결과**: 모든 쿼리 및 키워드 캐시 삭제, 최대 스토리지 확보

### 사용 사례 2: 쿼리 캐시만 새로 고침

**시나리오**: 키워드는 유지하면서 쿼리 캐시 재구축 강제

```bash
# 도구 실행
python -m lightrag.tools.clean_llm_query_cache

# 선택: 스토리지 타입 -> 옵션 2 (쿼리만) -> 확인 (y)
```

**결과**: 쿼리 캐시 삭제, 키워드는 빠른 재구축을 위해 보존

### 사용 사례 3: 오래된 키워드 정리

**시나리오**: 최근 쿼리 결과는 유지하면서 오래된 키워드 제거

```bash
# 도구 실행
python -m lightrag.tools.clean_llm_query_cache

# 선택: 스토리지 타입 -> 옵션 3 (키워드만) -> 확인 (y)
```

**결과**: 키워드 삭제, 쿼리 캐시 보존

### 사용 사례 4: 워크스페이스별 정리

**시나리오**: 특정 워크스페이스의 캐시 정리

```bash
# 워크스페이스 설정
export WORKSPACE=development

# 도구 실행
python -m lightrag.tools.clean_llm_query_cache

# 선택: 스토리지 타입 -> 정리 옵션 -> 확인 (y)
```

**결과**: development 워크스페이스 캐시만 정리됨

## 모범 사례

1. **정리 전 백업**
   - 주요 정리 전에 항상 스토리지 백업
   - 먼저 비프로덕션 데이터에서 정리 테스트
   - 정리 결정 문서화

2. **성능 모니터링**
   - 정리 중 스토리지 지표 모니터링
   - 정리 후 쿼리 성능 모니터링
   - 캐시 재구축 시간 허용

3. **예약 정리**
   - 주기적으로 캐시 정리 (주간/월간)
   - 개발 환경에서 정리 자동화
   - 안전을 위해 프로덕션 정리는 수동으로 유지

4. **선택적 삭제**
   - 필요에 따라 정리 범위 고려
   - 키워드 캐시는 재구축이 더 어려움
   - 쿼리 캐시는 자동으로 재구축됨

5. **스토리지 용량**
   - 스토리지 사용 추세 모니터링
   - 용량 한계에 도달하기 전에 캐시 정리
   - 필요한 경우 오래된 데이터 아카이브

## 마이그레이션 도구와 비교

| 기능 | 정리 도구 | 마이그레이션 도구 |
|------|-----------|-----------------|
| **목적** | 쿼리 캐시 삭제 | 추출 캐시 마이그레이션 |
| **캐시 타입** | mix/hybrid/local/global | default:extract/summary |
| **모드** | query, keywords | extract, summary |
| **작업** | 삭제 | 스토리지 간 복사 |
| **되돌리기 가능** | 아니오 | 예 (소스 변경 없음) |
| **사용 사례** | 스토리지 확보, 캐시 새로 고침 | 스토리지 백엔드 변경 |

## 제한사항

1. **단일 스토리지 작업**
   - 한 번에 하나의 스토리지 타입만 정리 가능
   - 여러 스토리지를 정리하려면 도구를 여러 번 실행

2. **드라이 런 모드 없음**
   - 확인 후 즉시 삭제됨
   - 미리보기 전용 모드 없음
   - 먼저 비프로덕션에서 테스트

3. **선택적 모드 정리 없음**
   - 특정 모드만 정리 불가 (예: `mix`만)
   - 선택한 캐시 타입에 대해 모든 모드에 적용됨
   - 캐시 타입당 전체 또는 없음

4. **예약 정리 없음**
   - 수동 실행 필요
   - 내장 스케줄링 없음
   - 자동화가 필요한 경우 cron/스케줄러 사용

5. **검증 제한사항**
   - 오류 시나리오에서 정리 후 검증이 실패할 수 있음
   - 중요한 작업에는 수동 검증 권장

## 향후 개선 계획

향후 버전을 위한 잠재적 개선사항:

- 선택적 모드 정리 (예: `mix` 모드만 정리)
- 나이 기반 정리 (X일 이상된 캐시 삭제)
- 크기 기반 정리 (가장 큰 캐시부터 삭제)
- 안전한 미리보기를 위한 드라이 런 모드
- 자동화된 스케줄링 지원
- 캐시 통계 내보내기
- 일시 중지/재개 기능이 있는 증분 정리

## 지원

문제, 질문 또는 기능 요청의 경우:
- 정리 보고서의 오류 세부사항 확인
- 스토리지 설정 검토
- 워크스페이스 설정 확인
- 먼저 소규모 데이터셋으로 테스트
- 프로젝트 이슈 트래커를 통해 버그 보고

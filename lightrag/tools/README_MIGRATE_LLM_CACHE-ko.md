# LLM 캐시 마이그레이션 도구 - 사용자 가이드

## 개요

이 도구는 LightRAG의 LLM 응답 캐시를 서로 다른 KV 스토리지 구현체 간에 마이그레이션합니다. 파일 추출(모드 `default`) 중에 생성된 캐시, 즉 엔티티 추출 및 요약 캐시를 마이그레이션합니다.

## 지원 스토리지 타입

1. **JsonKVStorage** - 파일 기반 JSON 스토리지
2. **RedisKVStorage** - Redis 데이터베이스 스토리지
3. **PGKVStorage** - PostgreSQL 데이터베이스 스토리지
4. **MongoKVStorage** - MongoDB 데이터베이스 스토리지
5. **OpenSearchKVStorage** - OpenSearch 인덱스 스토리지

## 캐시 타입

이 도구는 다음 캐시 타입을 마이그레이션합니다:
- `default:extract:*` - 엔티티 및 관계 추출 캐시
- `default:summary:*` - 엔티티 및 관계 요약 캐시

**참고**: 쿼리 캐시 (`mix`, `local`, `global` 등의 모드)는 마이그레이션되지 않습니다.

## 사전 요구사항

LLM 캐시 마이그레이션 도구는 LightRAG 서버의 스토리지 설정을 읽고, LLM 마이그레이션 옵션을 통해 소스 및 대상 스토리지를 선택할 수 있도록 합니다. 캐시 마이그레이션 전에 소스와 대상 스토리지 모두 올바르게 설정되어 있고 LightRAG 서버를 통해 접근 가능한지 확인하세요.

## 사용법

### 기본 사용법

LightRAG 프로젝트 루트 디렉터리에서 실행:

```bash
python -m lightrag.tools.migrate_llm_cache
# 또는
python lightrag/tools/migrate_llm_cache.py
```

### 대화형 워크플로우

도구는 다음 단계를 안내합니다:

#### 1. 소스 스토리지 타입 선택
```
Supported KV Storage Types:
[1] JsonKVStorage
[2] RedisKVStorage
[3] PGKVStorage
[4] MongoKVStorage
[5] OpenSearchKVStorage

Select Source storage type (1-5) (Press Enter to exit): 1
```

**참고**: 스토리지 선택 프롬프트에서 Enter 키를 누르거나 `0`을 입력하면 정상적으로 종료됩니다.

#### 2. 소스 스토리지 검증

도구가 다음을 수행합니다:
- 필수 환경 변수 확인
- 워크스페이스 설정 자동 감지
- 스토리지 초기화 및 연결
- 마이그레이션 가능한 캐시 레코드 수 계산

```
Checking environment variables...
✓ All required environment variables are set

Initializing Source storage...
- Storage Type: JsonKVStorage
- Workspace: space1
- Connection Status: ✓ Success

Counting cache records...
- Total: 8,734 records
```

**스토리지 타입별 진행 상황 표시:**
- **JsonKVStorage**: 빠른 인메모리 계산, 증분 진행 없이 최종 개수 표시
  ```
  Counting cache records...
  - Total: 8,734 records
  ```
- **RedisKVStorage**: 증분 횟수로 실시간 스캔 진행 상황 표시
  ```
  Scanning Redis keys... found 8,734 records
  ```
- **PostgreSQL**: 빠른 COUNT(*) 쿼리, 1초 이상 소요 시에만 타이밍 표시
  ```
  Counting PostgreSQL records... (took 2.3s)
  ```
- **MongoDB**: 빠른 count_documents(), 1초 이상 소요 시에만 타이밍 표시
  ```
  Counting MongoDB documents... (took 1.8s)
  ```
- **OpenSearchKVStorage**: PIT 기반 스캔, 눈에 띄는 경우 타이밍 표시
  ```
  Scanning OpenSearch documents... (took 1.5s)
  ```

#### 3. 대상 스토리지 타입 선택

도구는 소스 스토리지 타입을 자동으로 대상 선택에서 제외하고 나머지 옵션을 순서대로 다시 번호를 매깁니다:

```
Available Storage Types for Target (source: JsonKVStorage excluded):
[1] RedisKVStorage
[2] PGKVStorage
[3] MongoKVStorage
[4] OpenSearchKVStorage

Select Target storage type (1-4) (Press Enter or 0 to exit): 1
```

**중요 참고사항:**
- 소스와 대상에 동일한 스토리지 타입을 선택할 수 **없음**
- 옵션은 자동으로 다시 번호가 매겨짐 (예: [2], [3], [4] 대신 [1], [2], [3])
- 이 시점에서도 Enter 키 또는 `0`을 입력하면 종료 가능

그런 다음 도구는 소스와 동일한 프로세스(환경 변수 확인, 연결 초기화, 레코드 수 계산)로 대상 스토리지를 검증합니다.

#### 4. 마이그레이션 확인

```
==================================================
Migration Confirmation
Source: JsonKVStorage (workspace: space1) - 8,734 records
Target: MongoKVStorage (workspace: space1) - 0 records
Batch Size: 1,000 records/batch
Memory Mode: Streaming (memory-optimized)

⚠️  Warning: Target storage already has 0 records
Migration will overwrite records with the same keys

Continue? (y/n): y
```

#### 5. 마이그레이션 실행

도구는 메모리 효율성을 위해 기본적으로 **스트리밍 마이그레이션**을 사용합니다. 마이그레이션 진행 상황 확인:

```
=== Starting Streaming Migration ===
💡 Memory-optimized mode: Processing 1,000 records at a time

Batch 1/9: ████████░░░░░░░░░░░░ 1000/8734 (11.4%) - default:extract ✓
Batch 2/9: ████████████░░░░░░░░ 2000/8734 (22.9%) - default:extract ✓
...
Batch 9/9: ████████████████████ 8734/8734 (100.0%) - default:summary ✓

Persisting data to disk...
✓ Data persisted successfully
```

**주요 기능:**
- **스트리밍 모드**: 전체 데이터셋을 메모리에 로드하지 않고 배치로 데이터 처리
- **실시간 진행 상황**: 정확한 백분율 및 캐시 타입이 포함된 진행 표시줄 표시
- **성공 지표**: 성공한 배치에 ✓, 실패한 배치에 ✗
- **일정한 메모리 사용**: 수백만 개의 레코드를 효율적으로 처리

#### 6. 마이그레이션 보고서 검토

도구는 통계 및 발생한 오류를 보여주는 포괄적인 최종 보고서를 제공합니다:

**성공적인 마이그레이션:**
```
Migration Complete - Final Report

📊 Statistics:
  Total source records:    8,734
  Total batches:           9
  Successful batches:      9
  Failed batches:          0
  Successfully migrated:   8,734
  Failed to migrate:       0
  Success rate:            100.00%

✓ SUCCESS: All records migrated successfully!
```

**오류가 있는 마이그레이션:**
```
Migration Complete - Final Report

📊 Statistics:
  Total source records:    8,734
  Total batches:           9
  Successful batches:      8
  Failed batches:          1
  Successfully migrated:   7,734
  Failed to migrate:       1,000
  Success rate:            88.55%

⚠️  Errors encountered: 1

Error Details:
------------------------------------------------------------

Error Summary:
  - ConnectionError: 1 occurrence(s)

First 5 errors:

  1. Batch 2
     Type: ConnectionError
     Message: Connection timeout after 30s
     Records lost: 1,000

⚠️  WARNING: Migration completed with errors!
   Please review the error details above.
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

### 배치 마이그레이션

- 기본 배치 크기: 1,000 레코드/배치
- 한 번에 너무 많은 데이터를 로드하여 발생하는 메모리 오버플로 방지
- 각 배치는 독립적으로 커밋됨, 재개 기능 지원

### 메모리 효율적인 페이지네이션

대규모 데이터셋의 경우, 도구는 스토리지별 페이지네이션 전략을 구현합니다:

- **JsonKVStorage**: 직접 인메모리 접근 (데이터가 이미 공유 스토리지에 로드됨)
- **RedisKVStorage**: 파이프라인 배칭을 사용한 커서 기반 SCAN (1,000 키/배치)
- **PGKVStorage**: SQL LIMIT/OFFSET 페이지네이션 (1,000 레코드/배치)
- **MongoKVStorage**: batch_size를 사용한 커서 스트리밍 (1,000 문서/배치)
- **OpenSearchKVStorage**: KV 인덱스의 PIT + `search_after` 스캔 (1,000 문서/배치)

이를 통해 도구가 메모리 문제 없이 수백만 개의 캐시 레코드를 처리할 수 있습니다.

### 접두사 필터링 구현

도구는 서로 다른 스토리지 타입에 대해 최적화된 필터링 방법을 사용합니다:

- **JsonKVStorage**: 잠금 보호와 함께 직접 딕셔너리 반복
- **RedisKVStorage**: 네임스페이스 접두사 패턴이 있는 SCAN 명령 + 대량 GET을 위한 파이프라인
- **PGKVStorage**: 적절한 필드 매핑 (id, return_value 등)을 사용한 SQL LIKE 쿼리
- **MongoKVStorage**: 커서 스트리밍과 함께 `_id` 필드에 대한 MongoDB 정규식 쿼리
- **OpenSearchKVStorage**: Python에서 `_id` 접두사 필터링 및 `_source` 패스스루를 사용한 전체 인덱스 스캔

## 오류 처리 및 복원력

도구는 투명하고 탄력적인 마이그레이션을 위한 포괄적인 오류 추적을 구현합니다:

### 배치 수준 오류 추적
- 각 배치는 독립적으로 오류 검사됨
- 실패한 배치는 기록되지만 마이그레이션을 중단하지 않음
- 성공한 배치는 나중 배치가 실패해도 커밋됨
- 실시간 진행 상황에 각 배치에 대해 ✓ (성공) 또는 ✗ (실패) 표시

### 오류 보고
마이그레이션 완료 후 상세 보고서 포함:
- **통계**: 총 레코드 수, 성공/실패 횟수, 성공률
- **오류 요약**: 오류 타입별 그룹화 및 발생 횟수
- **오류 세부사항**: 배치 번호, 오류 타입, 메시지, 손실된 레코드 수
- **권장사항**: 성공 또는 검토 필요 여부 명확히 표시

### 이중 데이터 로딩 없음
- 기존 검증 방식과 달리, 도구는 모든 대상 데이터를 다시 로드하지 않음
- 오류는 마이그레이션 후가 아닌 마이그레이션 중에 감지됨
- 이로써 메모리 오버헤드가 제거되고 기존 대상 데이터가 올바르게 처리됨

## 중요 참고사항

1. **데이터 덮어쓰기 경고**
   - 마이그레이션은 대상 스토리지에서 동일한 키를 가진 레코드를 덮어씁니다
   - 대상 스토리지에 이미 데이터가 있는 경우 도구가 경고를 표시합니다
   - 데이터 마이그레이션은 반복적으로 수행할 수 있습니다
   - 대상 스토리지의 기존 데이터는 올바르게 처리됩니다

3. **중단 및 재개**
   - 언제든지 마이그레이션 중단 가능 (Ctrl+C)
   - 이미 마이그레이션된 데이터는 대상 스토리지에 유지됨
   - 다시 실행하면 기존 레코드를 덮어씁니다
   - 실패한 배치는 수동으로 재시도할 수 있음

4. **성능 고려사항**
   - 대규모 데이터 마이그레이션은 상당한 시간이 걸릴 수 있음
   - 사용량이 적은 시간에 마이그레이션 권장
   - 안정적인 네트워크 연결 확보 (원격 데이터베이스의 경우)
   - 메모리 사용량은 데이터셋 크기에 관계없이 일정하게 유지됨

## 스토리지 설정

도구는 다음 우선순위로 여러 설정 방법을 지원합니다:

1. **환경 변수** (최고 우선순위)
2. **기본값** (최저 우선순위)

#### 옵션 A: 환경 변수 설정

`.env` 파일에서 스토리지 설정 구성:

#### 워크스페이스 설정 (선택사항)

```bash
# 일반 워크스페이스 (모든 스토리지에서 공유)
WORKSPACE=space1

# 또는 특정 스토리지에 대한 독립 워크스페이스 설정
POSTGRES_WORKSPACE=pg_space
MONGODB_WORKSPACE=mongo_space
REDIS_WORKSPACE=redis_space
OPENSEARCH_WORKSPACE=os_space
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
OPENSEARCH_WORKSPACE=os_space
```

환경 변수가 제공되지 않으면 도구는 사용 가능한 내장 기본값으로 폴백됩니다. JsonKVStorage는 `WORKING_DIR`을 사용하거나 기본값 `./rag_storage`로 설정됩니다.

## 문제 해결

### 환경 변수 누락
```
✗ Missing required environment variables: POSTGRES_USER, POSTGRES_PASSWORD
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

**해결책**:
- 마이그레이션 프로세스의 오류 로그 확인
- 마이그레이션 도구 다시 실행
- 대상 스토리지 용량 및 권한 확인

## 예시 시나리오

### 시나리오 1: JSON에서 MongoDB로 마이그레이션

사용 사례: 단일 머신 개발에서 프로덕션으로 마이그레이션

```bash
# 1. 환경 변수 설정
WORKSPACE=production
MONGO_URI=mongodb://user:pass@prod-server:27017/
MONGO_DATABASE=LightRAG

# 2. 도구 실행
python -m lightrag.tools.migrate_llm_cache

# 3. 선택: 1 (JsonKVStorage) -> 1 (MongoKVStorage - 다시 번호 매겨진 옵션 4)
```

**참고**: 소스로 JsonKVStorage를 선택한 후, MongoKVStorage는 소스를 제외한 후 옵션이 다시 번호가 매겨지므로 대상 선택에서 옵션 [1]로 표시됩니다.

### 시나리오 2: Redis에서 PostgreSQL로

사용 사례: 캐시 스토리지에서 관계형 데이터베이스로 마이그레이션

```bash
# 1. 두 데이터베이스 모두 접근 가능한지 확인
REDIS_URI=redis://old-redis:6379
POSTGRES_HOST=new-postgres-server
# ... 기타 PostgreSQL 설정

# 2. 도구 실행
python -m lightrag.tools.migrate_llm_cache

# 3. 선택: 2 (RedisKVStorage) -> 2 (PGKVStorage - 다시 번호 매겨진 옵션 3)
```

**참고**: 소스로 RedisKVStorage를 선택한 후, PGKVStorage는 대상 선택에서 옵션 [2]로 표시됩니다.

### 시나리오 3: 다른 워크스페이스 간 마이그레이션

사용 사례: 다른 워크스페이스 환경 간에 데이터 마이그레이션

```bash
# 소스와 대상에 대한 별도 워크스페이스 설정
POSTGRES_WORKSPACE=dev_workspace  # 개발 환경용
MONGODB_WORKSPACE=prod_workspace  # 프로덕션 환경용

# 도구 실행
python -m lightrag.tools.migrate_llm_cache

# 선택: 3 (dev_workspace의 PGKVStorage) -> 3 (prod_workspace의 MongoKVStorage)
```

**참고**: 이를 통해 스토리지 백엔드를 변경하면서 다른 논리적 데이터 파티션 간에 마이그레이션할 수 있습니다.

## 도구 제한사항

1. **동일한 스토리지 타입 불가**
   - 동일한 스토리지 타입 간에 마이그레이션 불가 (예: PostgreSQL에서 PostgreSQL로)
   - 도구가 자동으로 소스 스토리지 타입을 대상 선택에서 제외하여 적용됨
   - 동일한 스토리지 마이그레이션 (예: 데이터베이스 전환)의 경우 데이터베이스 네이티브 도구 사용

2. **기본 모드 캐시만 해당**
   - `default:extract:*` 및 `default:summary:*`만 마이그레이션
   - 쿼리 캐시는 포함되지 않음

4. **네트워크 의존성**
   - 도구는 원격 데이터베이스에 안정적인 네트워크 연결 필요
   - 연결이 중단되면 대규모 데이터셋 마이그레이션이 실패할 수 있음

## 모범 사례

1. **마이그레이션 전 백업**
   - 마이그레이션 전에 항상 데이터 백업
   - 먼저 비프로덕션 데이터에서 마이그레이션 테스트

2. **결과 확인**
   - 마이그레이션 후 검증 출력 확인
   - 필요한 경우 일부 캐시 항목 수동 확인

3. **성능 모니터링**
   - 마이그레이션 중 데이터베이스 리소스 사용량 모니터링
   - 필요한 경우 더 작은 배치로 마이그레이션 고려

4. **이전 데이터 정리**
   - 성공적인 마이그레이션 후 이전 캐시 데이터 정리 고려
   - 삭제 전 적절한 기간 동안 백업 유지

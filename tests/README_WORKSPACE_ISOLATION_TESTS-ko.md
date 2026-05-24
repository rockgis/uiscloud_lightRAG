# 워크스페이스 격리 테스트 스위트

## 개요
LightRAG의 워크스페이스 격리 기능에 대한 포괄적인 테스트 커버리지로, 서로 다른 워크스페이스(프로젝트)가 데이터 오염이나 리소스 충돌 없이 독립적으로 공존할 수 있는지 확인합니다.

## 테스트 아키텍처

### 설계 원칙
1. **동시성 기반 어설션**: 타이밍 기반 테스트(불안정한) 대신, 실제 동시 잠금 보유자 수를 측정
2. **타임라인 검증**: 유한 상태 머신으로 적절한 순차 실행을 검증
3. **성능 지표**: 각 테스트는 디버깅 및 최적화를 위한 실행 지표를 보고
4. **구성 가능한 스트레스 테스트**: 환경 변수로 테스트 강도 제어

## 테스트 카테고리

### 1. 데이터 격리 테스트
**테스트:** 1, 4, 8, 9, 10
**목적:** 한 워크스페이스의 데이터가 다른 워크스페이스로 누출되지 않는지 확인

- **테스트 1: 파이프라인 상태 격리** - 핵심 공유 데이터 구조가 분리된 상태 유지
- **테스트 4: 멀티 워크스페이스 동시성** - 동시 작업이 서로 간섭하지 않음
- **테스트 8: 업데이트 플래그 격리** - 플래그 관리가 워크스페이스 경계를 준수
- **테스트 9: 빈 워크스페이스 표준화** - 빈 워크스페이스 문자열의 엣지 케이스 처리
- **테스트 10: JsonKVStorage 통합** - 스토리지 레이어가 데이터를 적절히 격리

### 2. 잠금 메커니즘 테스트
**테스트:** 2, 5, 6
**목적:** 잠금 메커니즘이 동일 워크스페이스 내에서는 직렬화를 적용하면서 워크스페이스 간에는 병렬 처리를 허용하는지 검증

- **테스트 2: 잠금 메커니즘** - 서로 다른 워크스페이스는 병렬로 실행되고, 동일 워크스페이스는 직렬화됨
- **테스트 5: 재진입 보호** - 재진입 잠금 획득으로 인한 교착 상태 방지
- **테스트 6: 네임스페이스 잠금 격리** - 동일 워크스페이스 내 서로 다른 네임스페이스는 독립적

### 3. 하위 호환성 테스트
**테스트:** 3
**목적:** 워크스페이스 매개변수 없는 레거시 코드가 올바르게 작동하는지 확인

- 기본 워크스페이스 폴백 동작
- 빈 워크스페이스 처리
- None 대 빈 문자열 정규화

### 4. 오류 처리 테스트
**테스트:** 7
**목적:** 잘못된 설정에 대한 안전장치 검증

- 누락된 워크스페이스 검증
- 워크스페이스 정규화
- 엣지 케이스 처리

### 5. 엔드투엔드 통합 테스트
**테스트:** 11
**목적:** 완전한 LightRAG 워크플로우가 격리를 유지하는지 검증

- 전체 문서 삽입 파이프라인
- 파일 시스템 분리
- 데이터 내용 확인

## 테스트 실행

### 기본 사용법
```bash
# 모든 워크스페이스 격리 테스트 실행
pytest tests/test_workspace_isolation.py -v

# 특정 테스트 실행
pytest tests/test_workspace_isolation.py::test_lock_mechanism -v

# 상세 출력으로 실행
pytest tests/test_workspace_isolation.py -v -s
```

### 환경 설정

#### 스트레스 테스트
구성 가능한 워커 수로 스트레스 테스트 활성화:
```bash
# 기본 3개 워커로 스트레스 모드 활성화
LIGHTRAG_STRESS_TEST=true pytest tests/test_workspace_isolation.py -v

# 커스텀 워커 수 (예: 10개)
LIGHTRAG_STRESS_TEST=true LIGHTRAG_TEST_WORKERS=10 pytest tests/test_workspace_isolation.py -v
```

#### 테스트 아티팩트 유지
수동 검사를 위해 임시 디렉터리 보존:
```bash
# 테스트 아티팩트 유지 (디버깅에 유용)
LIGHTRAG_KEEP_ARTIFACTS=true pytest tests/test_workspace_isolation.py -v
```

#### 조합 예시
```bash
# 20개 워커로 스트레스 테스트 및 아티팩트 유지
LIGHTRAG_STRESS_TEST=true \
LIGHTRAG_TEST_WORKERS=20 \
LIGHTRAG_KEEP_ARTIFACTS=true \
pytest tests/test_workspace_isolation.py::test_lock_mechanism -v -s
```

### CI/CD 통합
```bash
# 권장 CI/CD 명령 (아티팩트 없음, 기본 워커)
pytest tests/test_workspace_isolation.py -v --tb=short
```

## 테스트 구현 세부사항

### 헬퍼 함수

#### `_measure_lock_parallelism`
벽시계 시간이 아닌 실제 동시성을 측정합니다.

**반환:**
- `max_parallel`: 동시 잠금 보유자의 최대 수
- `timeline`: (task_name, event) 튜플의 순서 있는 목록
- `metrics`: 성능 데이터가 포함된 딕셔너리 (duration, concurrency, workers)

**예시:**
```python
workload = [
    ("task1", "workspace1", "namespace"),
    ("task2", "workspace2", "namespace"),
]
max_parallel, timeline, metrics = await _measure_lock_parallelism(workload)

# 타이밍이 아닌 실제 동작에 대한 어설션
assert max_parallel >= 2  # 두 개의 서로 다른 워크스페이스는 동시에 실행되어야 함
```

#### `_assert_no_timeline_overlap`
유한 상태 머신을 사용하여 순차 실행을 검증합니다.

**검증:**
- 겹치는 잠금 획득 없음
- 적절한 잠금 해제 순서
- 모든 잠금이 적절히 해제됨

**예시:**
```python
timeline = [
    ("task1", "start"),
    ("task1", "end"),
    ("task2", "start"),
    ("task2", "end"),
]
_assert_no_timeline_overlap(timeline)  # 통과 - 겹침 없음

timeline_bad = [
    ("task1", "start"),
    ("task2", "start"),  # 오류: task1이 끝나기 전에 task2가 시작됨
    ("task1", "end"),
]
_assert_no_timeline_overlap(timeline_bad)  # AssertionError 발생
```

## 설정 변수

| 변수 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `LIGHTRAG_STRESS_TEST` | bool | `false` | 스트레스 테스트 모드 활성화 |
| `LIGHTRAG_TEST_WORKERS` | int | `3` | 스트레스 모드의 병렬 워커 수 |
| `LIGHTRAG_KEEP_ARTIFACTS` | bool | `false` | 임시 테스트 디렉터리 유지 |

## 성능 벤치마크

### 예상 성능 (참조 시스템)
- **테스트 1-9**: 각 1초 미만
- **테스트 10**: 2초 미만 (파일 I/O 포함)
- **테스트 11**: 5초 미만 (전체 RAG 파이프라인 포함)
- **전체 스위트**: 15초 미만

### 스트레스 테스트 성능
`LIGHTRAG_TEST_WORKERS=10` 설정 시:
- **테스트 2 (병렬)**: ~0.05초 (10개 워커, 모두 동시 실행)
- **테스트 2 (직렬)**: ~0.10초 (2개 워커, 직렬화됨)

## 문제 해결

### 일반적인 문제

#### 불안정한 테스트 실패
**증상:** 로컬에서는 통과하지만 CI/CD에서 실패
**원인:** 시스템 과부하, 타이밍 기반 어설션
**해결책:** 우리 테스트는 타이밍이 아닌 동시성 기반 어설션을 사용합니다. 실패가 계속되면 오류 메시지의 `timeline` 출력을 확인하세요.

#### 리소스 정리 오류
**증상:** "Directory not empty" 또는 "Cannot remove directory"
**원인:** 동시 테스트 실행 또는 OS 파일 잠금
**해결책:** 직렬로 테스트 실행 (`pytest -n 1`) 또는 상태 검사를 위해 `LIGHTRAG_KEEP_ARTIFACTS=true` 사용

#### 잠금 타임아웃 오류
**증상:** "Lock acquisition timeout"
**원인:** 교착 상태 또는 리소스 기아
**해결책:** 교착 상태 패턴을 위한 테스트 출력 확인, 잠금 획득 순서 검토

### 디버그 팁

1. **상세 출력 활성화:**
   ```bash
   pytest tests/test_workspace_isolation.py -v -s
   ```

2. **아티팩트로 단일 테스트 실행:**
   ```bash
   LIGHTRAG_KEEP_ARTIFACTS=true pytest tests/test_workspace_isolation.py::test_json_kv_storage_workspace_isolation -v -s
   ```

3. **성능 지표 확인:**
   테스트 출력에서 duration 및 concurrency를 보여주는 "Performance:" 줄을 찾으세요.

4. **실패 시 타임라인 검사:**
   타임라인 데이터는 어설션 오류 메시지에 포함되어 있습니다.

## 기여

### 새 테스트 추가

1. **명명 규칙 준수:** `test_<기능>_<측면>`
2. **목적/범위 주석 추가:** 무엇을 왜 테스트하는지 설명
3. **헬퍼 함수 사용:** `_measure_lock_parallelism`, `_assert_no_timeline_overlap`
4. **어설션 문서화:** 예상 동작을 어설션으로 설명
5. **이 README 업데이트:** 적절한 카테고리에 테스트 추가

### 테스트 템플릿
```python
@pytest.mark.asyncio
async def test_new_feature():
    """
    이 테스트가 검증하는 내용에 대한 간략한 설명.
    """
    # 목적: 이 테스트가 존재하는 이유
    # 범위: 어떤 함수/클래스를 테스트하는지
    print("\n" + "=" * 60)
    print("TEST N: Feature Name")
    print("=" * 60)

    # 테스트 구현
    # ...

    print("✅ PASSED: Feature Name")
    print(f"   Validation details")
```

## 관련 문서

- [워크스페이스 격리 설계 문서](../docs/LightRAG_concurrent_explain.md)
- [워크스페이스 격리 설계 문서 (한국어)](../docs/LightRAG_concurrent_explain-ko.md)

## 테스트 커버리지 매트릭스

| 컴포넌트 | 데이터 격리 | 잠금 메커니즘 | 하위 호환성 | 오류 처리 | E2E |
|----------|:-----------:|:-------------:|:-----------:|:---------:|:---:|
| shared_storage | ✅ T1, T4 | ✅ T2, T5, T6 | ✅ T3 | ✅ T7 | ✅ T11 |
| update_flags | ✅ T8 | - | - | - | - |
| JsonKVStorage | ✅ T10 | - | - | - | ✅ T11 |
| LightRAG Core | - | - | - | - | ✅ T11 |
| Namespace | ✅ T9 | - | ✅ T3 | ✅ T7 | - |

**범례:** T# = 테스트 번호

## 버전 이력

- **v2.0** (2025-01-18): 성능 지표, 스트레스 테스트, 구성 가능한 정리 추가
- **v1.0** (초기 버전): 타이밍 기반 어설션을 사용한 기본 워크스페이스 격리 테스트

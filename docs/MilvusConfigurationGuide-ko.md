# vector_db_storage_cls_kwargs를 통한 Milvus 설정

## 개요

Milvus 인덱스 파라미터는 `vector_db_storage_cls_kwargs`를 통해 설정할 수 있으며, 이는 프레임워크 통합 시나리오(예: RAGAnything 또는 LightRAG 기반 다른 프레임워크 사용 시)에 **권장되는 방법**입니다.

## vector_db_storage_cls_kwargs를 사용하는 이유?

✅ **프레임워크 통합**: 환경 변수 변경 없이 프레임워크 레이어를 통해 설정을 전달할 수 있습니다
✅ **프로그래밍 방식 설정**: 환경 변수 대신 코드에서 파라미터를 설정합니다
✅ **동적 설정**: 서로 다른 RAG 인스턴스에 대해 다른 설정 사용 가능
✅ **깔끔한 API**: 모든 파라미터를 초기화 시 한 곳에서 전달

## 지원 파라미터

모든 11개의 MilvusIndexConfig 파라미터를 `vector_db_storage_cls_kwargs`로 설정할 수 있습니다:

### 기본 설정
- `index_type`: 인덱스 유형 (AUTOINDEX, HNSW, HNSW_SQ, IVF_FLAT 등)
- `metric_type`: 거리 메트릭 (COSINE, L2, IP)

### HNSW 파라미터
- `hnsw_m`: 레이어당 연결 수 (2-2048, 기본값: 16)
- `hnsw_ef_construction`: 구성 시 동적 후보 목록 크기 (기본값: 360)
- `hnsw_ef`: 검색 시 동적 후보 목록 크기 (기본값: 200)

### HNSW_SQ 파라미터 (Milvus 2.6.8+ 필요)
- `sq_type`: 양자화 유형 (SQ4U, SQ6, SQ8, BF16, FP16, 기본값: SQ8)
- `sq_refine`: 정제 활성화 (기본값: False)
- `sq_refine_type`: 정제 유형 (SQ6, SQ8, BF16, FP16, FP32, 기본값: FP32)
- `sq_refine_k`: 정제할 후보 수 (기본값: 10)

### IVF 파라미터
- `ivf_nlist`: 클러스터 단위 수 (1-65536, 기본값: 1024)
- `ivf_nprobe`: 쿼리할 단위 수 (기본값: 16)

## 설정 우선순위

설정은 다음 순서로 적용됩니다:
1. **vector_db_storage_cls_kwargs로 전달된 파라미터** (최고 우선순위)
2. 환경 변수 (MILVUS_INDEX_TYPE 등)
3. 기본값

## 사용 예제

### 기본 설정

```python
from lightrag import LightRAG

rag = LightRAG(
    working_dir="./demo",
    vector_storage="MilvusVectorDBStorage",
    vector_db_storage_cls_kwargs={
        "cosine_better_than_threshold": 0.2,
        "index_type": "HNSW",
        "metric_type": "COSINE",
        "hnsw_m": 32,
        "hnsw_ef_construction": 256,
        "hnsw_ef": 150,
    }
)
```

### RAGAnything 프레임워크 통합

```python
# RAGAnything 프레임워크 코드 내:
def create_lightrag_instance(user_config):
    """사용자 제공 Milvus 설정으로 LightRAG 인스턴스 생성"""

    # RAGAnything의 사용자 설정
    milvus_config = {
        "cosine_better_than_threshold": user_config.get("threshold", 0.2),
        "index_type": user_config.get("index_type", "HNSW"),
        "hnsw_m": user_config.get("hnsw_m", 32),
        # ... 기타 파라미터
    }

    # LightRAG에 설정 전달
    rag = LightRAG(
        working_dir=user_config["working_dir"],
        vector_storage="MilvusVectorDBStorage",
        vector_db_storage_cls_kwargs=milvus_config,
    )

    return rag
```

### HNSW_SQ를 사용한 고급 설정

```python
rag = LightRAG(
    working_dir="./demo",
    vector_storage="MilvusVectorDBStorage",
    vector_db_storage_cls_kwargs={
        "cosine_better_than_threshold": 0.2,
        "index_type": "HNSW_SQ",  # Milvus 2.6.8+ 필요
        "metric_type": "COSINE",
        "hnsw_m": 48,
        "hnsw_ef_construction": 400,
        "hnsw_ef": 200,
        "sq_type": "SQ8",
        "sq_refine": True,
        "sq_refine_type": "FP32",
        "sq_refine_k": 20,
    }
)
```

### IVF 설정

```python
rag = LightRAG(
    working_dir="./demo",
    vector_storage="MilvusVectorDBStorage",
    vector_db_storage_cls_kwargs={
        "cosine_better_than_threshold": 0.2,
        "index_type": "IVF_FLAT",
        "metric_type": "L2",
        "ivf_nlist": 2048,
        "ivf_nprobe": 32,
    }
)
```

## 구현 세부 사항

### 동작 방식

1. `MilvusVectorDBStorage.__post_init__()`이 호출될 때:
   ```python
   kwargs = self.global_config.get("vector_db_storage_cls_kwargs", {})
   index_config_keys = MilvusIndexConfig.get_config_field_names()
   index_config_params = {
       k: v for k, v in kwargs.items() if k in index_config_keys
   }
   self.index_config = MilvusIndexConfig(**index_config_params)
   ```

2. `MilvusIndexConfig.get_config_field_names()`이 데이터클래스에서 모든 유효한 파라미터 이름을 동적으로 추출합니다
3. kwargs에서 유효한 Milvus 인덱스 파라미터만 추출합니다
4. 파라미터는 기본값을 적용하고 검증하는 `MilvusIndexConfig`로 전달됩니다
5. 환경 변수는 kwargs에서 제공되지 않은 파라미터의 폴백으로 사용됩니다

### 자동 동기화

구현은 `MilvusIndexConfig.get_config_field_names()`를 사용하여 유효한 파라미터를 동적으로 추출합니다. 이는:
- ✅ `MilvusIndexConfig`에 추가된 새 파라미터가 **자동으로 인식**됩니다
- ✅ 중복 파라미터 목록 유지 불필요
- ✅ 설정 파라미터의 단일 출처

## 테스트

`vector_db_storage_cls_kwargs`를 통한 설정은 충분히 테스트되어 있습니다:

```bash
# 모든 kwargs 브릿지 테스트 실행
python -m pytest tests/test_milvus_kwargs_bridge.py -v

# RAGAnything 통합 시나리오 특별 테스트
python -m pytest tests/test_milvus_kwargs_bridge.py::TestMilvusKwargsParameterBridge::test_raganything_framework_integration_scenario -v

# 모든 파라미터 지원 테스트
python -m pytest tests/test_milvus_kwargs_bridge.py::TestMilvusKwargsParameterBridge::test_all_milvus_parameters_supported_via_kwargs -v
```

## 예제

완전한 작동 예제는 `examples/milvus_kwargs_configuration_demo.py`를 참조하세요.

## 하위 호환성

✅ 기존 코드와 **100% 하위 호환**됩니다
✅ 환경 변수 설정이 계속 작동합니다
✅ 모든 기존 테스트가 통과합니다

## FAQ

### Q: kwargs와 환경 변수를 혼합하여 사용할 수 있나요?
**A:** 네! `vector_db_storage_cls_kwargs`의 파라미터가 환경 변수보다 우선합니다.

### Q: kwargs의 Milvus가 아닌 파라미터는 어떻게 되나요?
**A:** 무시됩니다. 유효한 MilvusIndexConfig 파라미터만 추출됩니다. 이를 통해 프레임워크가 Milvus 설정과 함께 자체 파라미터를 전달할 수 있습니다.

### Q: 환경 변수를 설정해야 하나요?
**A:** 아니요! `vector_db_storage_cls_kwargs`를 사용할 때 환경 변수는 선택사항입니다. 폴백 값으로 사용됩니다.

### Q: RAGAnything에 이 방법이 권장되나요?
**A:** 네! 이는 LightRAG를 기반으로 구축된 모든 프레임워크에 **권장되는 방법**으로, 프레임워크 레이어를 통해 설정을 깔끔하게 전달할 수 있습니다.

## 참조

- 테스트 스위트: `tests/test_milvus_kwargs_bridge.py`
- 구현: `lightrag/kg/milvus_impl.py` (1237-1272번 줄)
- 예제: `examples/milvus_kwargs_configuration_demo.py`
- MilvusIndexConfig: `lightrag/kg/milvus_impl.py` (75-303번 줄)

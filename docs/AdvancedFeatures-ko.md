# 고급 기능

## 멀티모달 문서 처리 (RAG-Anything 통합)

LightRAG는 [RAG-Anything](https://github.com/HKUDS/RAG-Anything)과 통합됩니다. RAG-Anything은 PDF, 이미지, Office 문서, 표, 수식 등 다양한 문서 형식에서 고급 파싱 및 RAG 기능을 제공하는 **올인원 멀티모달 문서 처리 RAG 시스템**입니다.

**주요 기능:**
- 엔드투엔드 멀티모달 파이프라인: 문서 수집부터 멀티모달 쿼리 응답까지 완전한 워크플로우
- 범용 문서 지원: PDF, Office 문서(DOC/DOCX/PPT/PPTX/XLS/XLSX), 이미지, 다양한 파일 형식
- 전문 콘텐츠 분석: 이미지, 표, 수식 전용 프로세서
- 멀티모달 지식 그래프: 자동 엔티티 추출 및 크로스모달 관계 발견
- 하이브리드 지능형 검색: 텍스트 및 멀티모달 콘텐츠를 아우르는 고급 검색

### 빠른 시작

* RAG-Anything 설치

```bash
pip install raganything
```

* RAGAnything 사용 예제

```python
import asyncio
from raganything import RAGAnything
from lightrag import LightRAG
from lightrag.llm.openai import openai_complete_if_cache, openai_embed
from lightrag.utils import EmbeddingFunc
import os

async def load_existing_lightrag():
    lightrag_working_dir = "./existing_lightrag_storage"

    from functools import partial

    lightrag_instance = LightRAG(
        working_dir=lightrag_working_dir,
        llm_model_func=lambda prompt, system_prompt=None, history_messages=[], **kwargs: openai_complete_if_cache(
            "gpt-4o-mini",
            prompt,
            system_prompt=system_prompt,
            history_messages=history_messages,
            api_key="your-api-key",
            **kwargs,
        ),
        embedding_func=EmbeddingFunc(
            embedding_dim=3072,
            max_token_size=8192,
            model="text-embedding-3-large",
            func=partial(
                openai_embed.func,
                model="text-embedding-3-large",
                api_key=api_key,
                base_url=base_url,
            ),
        )
    )

    await lightrag_instance.initialize_storages()

    rag = RAGAnything(
        lightrag=lightrag_instance,
        vision_model_func=lambda prompt, system_prompt=None, history_messages=[], image_data=None, **kwargs: openai_complete_if_cache(
            "gpt-4o",
            "",
            system_prompt=None,
            history_messages=[],
            messages=[
                {"role": "system", "content": system_prompt} if system_prompt else None,
                {"role": "user", "content": [
                    {"type": "text", "text": prompt},
                    {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image_data}"}}
                ]} if image_data else {"role": "user", "content": prompt}
            ],
            api_key="your-api-key",
            **kwargs,
        ) if image_data else openai_complete_if_cache(
            "gpt-4o-mini",
            prompt,
            system_prompt=system_prompt,
            history_messages=history_messages,
            api_key="your-api-key",
            **kwargs,
        )
    )

    result = await rag.query_with_multimodal(
        "이 LightRAG 인스턴스에서 처리된 데이터는 무엇입니까?",
        mode="hybrid"
    )
    print("쿼리 결과:", result)

    await rag.process_document_complete(
        file_path="path/to/new/multimodal_document.pdf",
        output_dir="./output"
    )

if __name__ == "__main__":
    asyncio.run(load_existing_lightrag())
```

* 자세한 문서 및 고급 사용법은 [RAG-Anything 저장소](https://github.com/HKUDS/RAG-Anything)를 참조하세요.

---

## 토큰 사용량 추적

**개요 및 사용법**

LightRAG는 대형 언어 모델의 토큰 소비를 모니터링하고 관리하기 위한 `TokenTracker` 도구를 제공합니다. 이 기능은 API 비용 관리 및 성능 최적화에 유용합니다.

```python
from lightrag.utils import TokenTracker

token_tracker = TokenTracker()

# 방법 1: 컨텍스트 매니저 사용 (권장)
with token_tracker:
    result1 = await llm_model_func("질문 1")
    result2 = await llm_model_func("질문 2")

# 방법 2: 수동으로 토큰 사용량 기록 추가
token_tracker.reset()

rag.insert()

rag.query("질문 1", param=QueryParam(mode="naive"))
rag.query("질문 2", param=QueryParam(mode="mix"))

print("토큰 사용량:", token_tracker.get_usage())
```

**사용 팁:**
- 장기 세션이나 배치 작업에는 컨텍스트 매니저를 사용하여 모든 토큰 소비를 자동으로 추적하세요
- 분할 통계를 원할 경우 수동 모드를 사용하고 적절한 시점에 `reset()`을 호출하세요
- 정기적인 토큰 사용량 확인으로 비정상적인 소비를 조기에 감지하세요

**예제 파일:**
- `examples/lightrag_gemini_track_token_demo.py`: Google Gemini를 사용한 토큰 추적
- `examples/lightrag_siliconcloud_track_token_demo.py`: SiliconCloud를 사용한 토큰 추적

---

## 데이터 내보내기 기능

LightRAG는 분석, 공유, 백업을 위해 지식 그래프 데이터를 다양한 형식으로 내보낼 수 있습니다.

**기본 사용법**

```python
# 기본 CSV 내보내기 (기본 형식)
rag.export_data("knowledge_graph.csv")

# 형식 지정
rag.export_data("output.xlsx", file_format="excel")
```

**지원 파일 형식**

```python
rag.export_data("graph_data.csv", file_format="csv")
rag.export_data("graph_data.xlsx", file_format="excel")
rag.export_data("graph_data.md", file_format="md")
rag.export_data("graph_data.txt", file_format="txt")
```

**추가 옵션**

벡터 임베딩을 내보내기에 포함 (선택사항):

```python
rag.export_data("complete_data.csv", include_vector_data=True)
```

모든 내보내기에는 엔티티 정보(이름, ID, 메타데이터), 관계 데이터(엔티티 간 연결), 벡터 데이터베이스의 관계 정보가 포함됩니다.

---

## 캐시 관리

**캐시 지우기**

`aclear_cache()`는 `llm_response_cache`의 모든 캐시 항목을 지웁니다. 모드나 캐시 유형별 선택적 삭제는 지원하지 않습니다.

```python
# 비동기
await rag.aclear_cache()

# 동기
rag.clear_cache()
```

쿼리 관련 캐시의 선택적 삭제는 `lightrag.tools.clean_llm_query_cache` 도구를 사용하세요. 이 도구는 `mix`, `hybrid`, `local`, `global` 모드의 쿼리 캐시와 키워드 캐시를 관리합니다. `default:extract:*`, `default:summary:*` 같은 추출 캐시는 삭제하지 **않습니다**.

---

## Langfuse 옵저버빌리티 통합

Langfuse는 OpenAI 클라이언트의 드롭인 대체품으로, 모든 LLM 상호작용을 자동으로 추적하여 RAG 시스템을 모니터링, 디버깅, 최적화할 수 있게 해줍니다.

### 설치

```bash
pip install lightrag-hku[observability]
# 또는 소스에서:
pip install -e ".[observability]"
```

### 설정

`.env` 파일에 추가:

```
## Langfuse 옵저버빌리티 (선택사항)
LANGFUSE_SECRET_KEY=""
LANGFUSE_PUBLIC_KEY=""
LANGFUSE_HOST="https://cloud.langfuse.com"  # 또는 자체 호스팅 인스턴스
LANGFUSE_ENABLE_TRACE=true
```

### 기능

설치 및 설정 후 Langfuse는 모든 OpenAI LLM 호출을 자동으로 추적합니다. 대시보드 기능:
- **추적**: 전체 LLM 호출 체인 조회
- **분석**: 토큰 사용량, 지연시간, 비용 메트릭
- **디버깅**: 프롬프트 및 응답 검사
- **평가**: 모델 출력 비교
- **모니터링**: 실시간 알림

> **참고**: LightRAG는 현재 OpenAI 호환 API 호출만 Langfuse와 통합됩니다. Ollama, Azure, AWS Bedrock 등의 API는 아직 Langfuse 옵저버빌리티를 지원하지 않습니다.

---

## RAGAS 기반 평가

**RAGAS** (Retrieval Augmented Generation Assessment)는 LLM을 사용하여 RAG 시스템을 참조 없이 평가하는 프레임워크입니다. LightRAG는 RAGAS 기반 평가 스크립트를 제공합니다. 자세한 내용은 [RAGAS 기반 평가 프레임워크](../lightrag/evaluation/README_EVALUASTION_RAGAS.md)를 참조하세요.

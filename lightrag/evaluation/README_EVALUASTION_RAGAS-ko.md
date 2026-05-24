# 📊 RAGAS 기반 평가 프레임워크

## RAGAS란?

**RAGAS** (Retrieval Augmented Generation Assessment)는 LLM을 사용하여 RAG 시스템을 참조 없이 평가하는 프레임워크입니다. RAGAS는 최첨단 평가 메트릭을 사용합니다:

### 핵심 메트릭

| 메트릭 | 측정 항목 | 좋은 점수 |
| ------ | -------- | --------- |
| **충실도(Faithfulness)** | 검색된 컨텍스트를 기반으로 답변이 사실적으로 정확한가? | > 0.80 |
| **답변 관련성(Answer Relevance)** | 답변이 사용자 질문과 관련이 있는가? | > 0.80 |
| **컨텍스트 재현율(Context Recall)** | 문서에서 모든 관련 정보가 검색되었는가? | > 0.80 |
| **컨텍스트 정밀도(Context Precision)** | 검색된 컨텍스트가 관련 없는 노이즈 없이 깨끗한가? | > 0.80 |
| **RAGAS 점수** | 전체 품질 메트릭 (위 항목의 평균) | > 0.80 |

### 📁 LightRAG 평가 프레임워크 디렉토리 구조

```
lightrag/evaluation/
├── eval_rag_quality.py      # 메인 평가 스크립트
├── sample_dataset.json      # LightRAG에 대한 3개의 테스트 질문
├── sample_documents/        # 테스트용 매칭 마크다운 파일
│   ├── 01_lightrag_overview.md
│   ├── 02_rag_architecture.md
│   ├── 03_lightrag_improvements.md
│   ├── 04_supported_databases.md
│   ├── 05_evaluation_and_deployment.md
│   └── README.md
├── __init__.py              # 패키지 초기화
├── results/                 # 출력 디렉토리
│   ├── results_YYYYMMDD_HHMMSS.json    # JSON 형식 원시 메트릭
│   └── results_YYYYMMDD_HHMMSS.csv     # CSV 형식 메트릭
└── README.md                # 이 파일
```

**빠른 테스트:** `sample_documents/`의 파일을 LightRAG에 인덱싱한 후 평가자를 실행하면 결과를 재현할 수 있습니다 (질문당 약 89-100% RAGAS 점수).

---

## 🚀 빠른 시작

### 1. 의존성 설치

```bash
pip install ragas datasets langfuse
```

또는 프로젝트 의존성 사용 (pyproject.toml에 이미 포함됨):

```bash
pip install -e ".[evaluation]"
```

### 2. 평가 실행

**기본 사용법 (기본값 사용):**
```bash
cd /path/to/LightRAG
python lightrag/evaluation/eval_rag_quality.py
```

**커스텀 데이터셋 지정:**
```bash
python lightrag/evaluation/eval_rag_quality.py --dataset my_test.json
```

**커스텀 RAG 엔드포인트 지정:**
```bash
python lightrag/evaluation/eval_rag_quality.py --ragendpoint http://my-server.com:9621
```

**두 가지 모두 지정 (단축 형태):**
```bash
python lightrag/evaluation/eval_rag_quality.py -d my_test.json -r http://localhost:9621
```

**도움말 표시:**
```bash
python lightrag/evaluation/eval_rag_quality.py --help
```

### 3. 결과 확인

결과는 `lightrag/evaluation/results/`에 자동으로 저장됩니다:

```
results/
├── results_20241023_143022.json     ← JSON 형식 원시 메트릭
└── results_20241023_143022.csv      ← CSV 형식 메트릭 (스프레드시트용)
```

**결과에 포함된 항목:**
- ✅ 전체 RAGAS 점수
- 📊 메트릭별 평균 (충실도, 답변 관련성, 컨텍스트 재현율, 컨텍스트 정밀도)
- 📋 개별 테스트 케이스 결과
- 📈 질문별 성능 분석

---

## 📋 커맨드라인 인수

평가 스크립트는 쉬운 설정을 위해 커맨드라인 인수를 지원합니다:

| 인수 | 단축 | 기본값 | 설명 |
| ---- | ---- | ------ | ---- |
| `--dataset` | `-d` | `sample_dataset.json` | 테스트 데이터셋 JSON 파일 경로 |
| `--ragendpoint` | `-r` | `http://localhost:9621` 또는 `$LIGHTRAG_API_URL` | LightRAG API 엔드포인트 URL |

### 사용 예제

**기본 데이터셋 및 엔드포인트 사용:**
```bash
python lightrag/evaluation/eval_rag_quality.py
```

**기본 엔드포인트로 커스텀 데이터셋:**
```bash
python lightrag/evaluation/eval_rag_quality.py --dataset path/to/my_dataset.json
```

**기본 데이터셋으로 커스텀 엔드포인트:**
```bash
python lightrag/evaluation/eval_rag_quality.py --ragendpoint http://my-server.com:9621
```

**커스텀 데이터셋 및 엔드포인트:**
```bash
python lightrag/evaluation/eval_rag_quality.py -d my_dataset.json -r http://localhost:9621
```

---

## ⚙️ 설정

### 환경 변수

**⚠️ 중요: LLM 및 임베딩 엔드포인트 모두 OpenAI 호환 방식이어야 합니다**
- RAGAS 프레임워크는 OpenAI 호환 API 인터페이스를 필요로 합니다
- 커스텀 엔드포인트는 OpenAI API 형식을 구현해야 합니다 (예: vLLM, SGLang, LocalAI)
- 호환되지 않는 엔드포인트는 평가 실패를 유발합니다

| 변수 | 기본값 | 설명 |
| ---- | ------ | ---- |
| **LLM 설정** | | |
| `EVAL_LLM_MODEL` | `gpt-4o-mini` | RAGAS 평가에 사용되는 LLM 모델 |
| `EVAL_LLM_BINDING_API_KEY` | `OPENAI_API_KEY` 폴백 | LLM 평가용 API 키 |
| `EVAL_LLM_BINDING_HOST` | (선택사항) | LLM용 커스텀 OpenAI 호환 엔드포인트 URL |
| **임베딩 설정** | | |
| `EVAL_EMBEDDING_MODEL` | `text-embedding-3-large` | 평가용 임베딩 모델 |
| `EVAL_EMBEDDING_BINDING_API_KEY` | `EVAL_LLM_BINDING_API_KEY` → `OPENAI_API_KEY` 폴백 | 임베딩용 API 키 |
| `EVAL_EMBEDDING_BINDING_HOST` | `EVAL_LLM_BINDING_HOST` 폴백 | 임베딩용 커스텀 OpenAI 호환 엔드포인트 URL |
| **성능 튜닝** | | |
| `EVAL_MAX_CONCURRENT` | 2 | 동시 테스트 케이스 평가 수 (1=직렬) |
| `EVAL_QUERY_TOP_K` | 10 | 쿼리당 검색할 문서 수 |
| `EVAL_LLM_MAX_RETRIES` | 5 | 최대 LLM 요청 재시도 횟수 |
| `EVAL_LLM_TIMEOUT` | 180 | LLM 요청 타임아웃 (초) |

### 사용 예제

**예제 1: 기본 설정 (OpenAI 공식 API)**
```bash
export OPENAI_API_KEY=sk-xxx
python lightrag/evaluation/eval_rag_quality.py
```
LLM과 임베딩 모두 기본 모델로 OpenAI 공식 API를 사용합니다.

**예제 2: OpenAI에서 커스텀 모델**
```bash
export OPENAI_API_KEY=sk-xxx
export EVAL_LLM_MODEL=gpt-4o-mini
export EVAL_EMBEDDING_MODEL=text-embedding-3-large
python lightrag/evaluation/eval_rag_quality.py
```

**예제 3: 동일한 커스텀 OpenAI 호환 엔드포인트 사용**
```bash
export EVAL_LLM_BINDING_API_KEY=your-custom-key
export EVAL_LLM_BINDING_HOST=http://localhost:8000/v1
export EVAL_LLM_MODEL=qwen-plus
export EVAL_EMBEDDING_MODEL=BAAI/bge-m3
python lightrag/evaluation/eval_rag_quality.py
```
임베딩이 자동으로 LLM 엔드포인트 설정을 상속합니다.

**예제 4: 별도 엔드포인트 (비용 최적화)**
```bash
# OpenAI를 LLM에 사용 (고품질)
export EVAL_LLM_BINDING_API_KEY=sk-openai-key
export EVAL_LLM_MODEL=gpt-4o-mini

# 로컬 vLLM을 임베딩에 사용 (비용 효율적)
export EVAL_EMBEDDING_BINDING_API_KEY=local-key
export EVAL_EMBEDDING_BINDING_HOST=http://localhost:8001/v1
export EVAL_EMBEDDING_MODEL=BAAI/bge-m3

python lightrag/evaluation/eval_rag_quality.py
```

**예제 5: LLM과 임베딩에 서로 다른 커스텀 엔드포인트**
```bash
export EVAL_LLM_BINDING_API_KEY=key1
export EVAL_LLM_BINDING_HOST=http://llm-server:8000/v1
export EVAL_LLM_MODEL=custom-llm

export EVAL_EMBEDDING_BINDING_API_KEY=key2
export EVAL_EMBEDDING_BINDING_HOST=http://embedding-server:8001/v1
export EVAL_EMBEDDING_MODEL=custom-embedding

python lightrag/evaluation/eval_rag_quality.py
```

**예제 6: .env 파일의 환경 변수 사용**
```bash
cat > .env << EOF
EVAL_LLM_BINDING_API_KEY=your-key
EVAL_LLM_BINDING_HOST=http://localhost:8000/v1
EVAL_LLM_MODEL=qwen-plus
EVAL_EMBEDDING_MODEL=BAAI/bge-m3
EOF

python lightrag/evaluation/eval_rag_quality.py
```

### 동시성 제어 및 요청 제한

**동시성 제어가 중요한 이유:**
- RAGAS는 각 테스트 케이스에 대해 내부적으로 많은 동시 LLM 호출을 합니다
- 컨텍스트 정밀도 메트릭은 검색된 문서당 LLM을 한 번 호출합니다
- 제어 없이는 API 요청 제한을 쉽게 초과할 수 있습니다

**기본 설정 (보수적):**
```bash
EVAL_MAX_CONCURRENT=2    # 직렬 평가 (한 번에 하나씩)
EVAL_QUERY_TOP_K=10      # LightRAG의 TOP_K 쿼리 파라미터
EVAL_LLM_MAX_RETRIES=5   # 실패한 요청 5회 재시도
EVAL_LLM_TIMEOUT=180     # 요청당 3분 타임아웃
```

**일반적인 문제 및 해결책:**

| 문제 | 해결책 |
| ---- | ------ |
| **경고: "LM returned 1 generations instead of 3"** | `EVAL_MAX_CONCURRENT`를 1로 줄이거나 `EVAL_QUERY_TOP_K`를 낮추세요 |
| **Context Precision이 NaN 반환** | `EVAL_QUERY_TOP_K`를 낮춰 테스트 케이스당 LLM 호출 수를 줄이세요 |
| **요청 제한 오류 (429)** | `EVAL_LLM_MAX_RETRIES`를 늘리고 `EVAL_MAX_CONCURRENT`를 낮추세요 |
| **요청 타임아웃** | `EVAL_LLM_TIMEOUT`을 180 이상으로 늘리세요 |

---

## 📝 테스트 데이터셋

`sample_dataset.json`에는 LightRAG에 대한 3가지 일반 질문이 포함됩니다. 인덱싱된 문서에 맞는 질문으로 교체하세요.

**커스텀 테스트 케이스:**

```json
{
  "test_cases": [
    {
      "question": "여기에 질문을 입력하세요",
      "ground_truth": "데이터에서 예상되는 답변",
      "project": "evaluation_project_name"
    }
  ]
}
```

---

## 📊 결과 해석

### 점수 범위

- **0.80-1.00**: ✅ 우수 (프로덕션 준비 완료)
- **0.60-0.80**: ⚠️ 양호 (개선 여지 있음)
- **0.40-0.60**: ❌ 미흡 (최적화 필요)
- **0.00-0.40**: 🔴 심각 (주요 문제 있음)

### 낮은 점수의 의미

| 메트릭 | 낮은 점수 의미 |
| ------ | ------------- |
| **충실도** | 응답에 환각(hallucination) 또는 부정확한 정보 포함 |
| **답변 관련성** | 답변이 사용자 질문과 일치하지 않음 |
| **컨텍스트 재현율** | 검색에서 중요 정보 누락 |
| **컨텍스트 정밀도** | 검색된 문서에 관련 없는 노이즈 포함 |

### 최적화 팁

1. **낮은 충실도:**
   - 엔티티 추출 품질 개선
   - 더 나은 문서 청킹
   - 검색 온도 조정

2. **낮은 답변 관련성:**
   - 프롬프트 엔지니어링 개선
   - 더 나은 쿼리 이해
   - 의미적 유사도 임계값 확인

3. **낮은 컨텍스트 재현율:**
   - 검색 `top_k` 결과 수 증가
   - 임베딩 모델 개선
   - 더 나은 문서 전처리

4. **낮은 컨텍스트 정밀도:**
   - 더 작고 집중된 청크
   - 더 나은 필터링
   - 청킹 전략 개선

---

## 📚 참고 자료

- [RAGAS 문서](https://docs.ragas.io/)
- [RAGAS GitHub](https://github.com/explodinggradients/ragas)

---

## 🐛 문제 해결

### "ModuleNotFoundError: No module named 'ragas'"

```bash
pip install ragas datasets
```

### "Warning: LM returned 1 generations instead of requested 3" 또는 Context Precision NaN

**원인**: 이 경고는 API 요청 제한 또는 동시 요청 과부하를 나타냅니다:
- RAGAS는 테스트 케이스당 여러 LLM 호출을 합니다
- 컨텍스트 정밀도는 검색된 문서당 LLM을 한 번 호출합니다 (`EVAL_QUERY_TOP_K=10`이면 10번 호출)
- 동시 평가는 이 호출을 곱합니다: `EVAL_MAX_CONCURRENT × 테스트당 LLM 호출 수`

**해결책 (효과 순서):**

1. **직렬 평가** (기본값):
   ```bash
   export EVAL_MAX_CONCURRENT=1
   python lightrag/evaluation/eval_rag_quality.py
   ```

2. **검색 문서 수 줄이기:**
   ```bash
   export EVAL_QUERY_TOP_K=5  # 컨텍스트 정밀도 LLM 호출을 절반으로 줄임
   python lightrag/evaluation/eval_rag_quality.py
   ```

3. **재시도 및 타임아웃 증가:**
   ```bash
   export EVAL_LLM_MAX_RETRIES=10
   export EVAL_LLM_TIMEOUT=180
   python lightrag/evaluation/eval_rag_quality.py
   ```

### "AttributeError: 'InstructorLLM' object has no attribute 'agenerate_prompt'" 또는 NaN 결과

**해결책**: 다음 중 하나가 설정되어 있는지 확인하세요:
- `OPENAI_API_KEY` 환경 변수 (기본값)
- 커스텀 API 키용 `EVAL_LLM_BINDING_API_KEY`

### "No sample_dataset.json found"

프로젝트 루트에서 실행하고 있는지 확인하세요:
```bash
cd /path/to/LightRAG
python lightrag/evaluation/eval_rag_quality.py
```

### 평가에 실행 중인 LightRAG API 필요

평가자는 `http://localhost:9621`에서 실행 중인 LightRAG API 서버를 쿼리합니다. 확인 사항:
1. LightRAG API 서버가 실행 중인지 확인 (`python lightrag/api/lightrag_server.py`)
2. LightRAG 인스턴스에 문서가 인덱싱되어 있는지 확인
3. API가 설정된 URL에서 접근 가능한지 확인

---

## 📝 다음 단계

1. LightRAG API 서버 시작
2. WebUI를 통해 샘플 문서를 LightRAG에 업로드
3. `python lightrag/evaluation/eval_rag_quality.py` 실행
4. `results/` 폴더의 결과(JSON/CSV) 검토

### 평가 결과 샘플

```
INFO: ======================================================================
INFO: 🔍 RAGAS 평가 - 실제 LightRAG API 사용
INFO: ======================================================================
INFO: 평가 모델:
INFO:   • LLM 모델:            gpt-4.1
INFO:   • 임베딩 모델:         text-embedding-3-large
INFO:   • 엔드포인트:          OpenAI 공식 API
INFO: ======================================================================
INFO: 📊 평가 결과 요약
INFO: ======================================================================
INFO: #    | 질문                                               |  충실도  | 답변관련 | 컨텍재현 | 컨텍정밀 |  RAGAS | 상태
INFO: 1    | LightRAG는 어떻게 환각 문제를 해결하나요?...       | 1.0000 |  1.0000 | 1.0000 |  1.0000 | 1.0000 |      ✓
INFO: ...
INFO: ======================================================================
INFO: 📈 벤치마크 결과 (평균)
INFO: ======================================================================
INFO: 평균 충실도:          0.9053
INFO: 평균 답변 관련성:     0.8646
INFO: 평균 컨텍스트 재현율: 1.0000
INFO: 평균 컨텍스트 정밀도: 1.0000
INFO: 평균 RAGAS 점수:      0.9425
```

---

**즐거운 평가 되세요! 🚀**

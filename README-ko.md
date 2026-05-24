<div align="center">

<div style="margin: 20px 0;">
  <img src="./assets/logo.png" width="120" height="120" alt="LightRAG Logo" style="border-radius: 20px; box-shadow: 0 8px 32px rgba(0, 217, 255, 0.3);">
</div>

# 🚀 LightRAG: 간단하고 빠른 검색 증강 생성

<div align="center">
    <a href="https://trendshift.io/repositories/13043" target="_blank"><img src="https://trendshift.io/api/badge/repositories/13043" alt="HKUDS%2FLightRAG | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
</div>

<div align="center">
  <p>
    <a href='https://github.com/HKUDS/LightRAG'><img src='https://img.shields.io/badge/🔥Project-Page-00d9ff?style=for-the-badge&logo=github&logoColor=white&labelColor=1a1a2e'></a>
    <a href='https://arxiv.org/abs/2410.05779'><img src='https://img.shields.io/badge/📄arXiv-2410.05779-ff6b6b?style=for-the-badge&logo=arxiv&logoColor=white&labelColor=1a1a2e'></a>
  </p>
  <p>
    <a href="https://discord.gg/yF2MmDJyGJ"><img src="https://img.shields.io/badge/💬Discord-커뮤니티-7289da?style=for-the-badge&logo=discord&logoColor=white&labelColor=1a1a2e"></a>
  </p>
  <p>
    <a href="README-zh.md"><img src="https://img.shields.io/badge/🇨🇳中文版-1a1a2e?style=for-the-badge"></a>
    <a href="README.md"><img src="https://img.shields.io/badge/🇺🇸English-1a1a2e?style=for-the-badge"></a>
    <a href="README-ko.md"><img src="https://img.shields.io/badge/🇰🇷한국어-1a1a2e?style=for-the-badge"></a>
  </p>
</div>

</div>

---

## 🎉 최신 소식

- [2026.03] 🎯 [새 기능]: **OpenSearch**를 통합 스토리지 백엔드로 추가 — LightRAG의 4가지 스토리지 유형 모두 지원.
- [2026.03] 🎯 [새 기능]: 설정 마법사 도입. Docker를 통한 임베딩, 리랭킹, 스토리지 백엔드 로컬 배포 지원.
- [2025.11] 🎯 [새 기능]: **평가용 RAGAS** 및 **추적용 Langfuse** 통합. 컨텍스트 정밀도 메트릭 지원을 위해 쿼리 결과와 함께 검색 컨텍스트를 반환하도록 API 업데이트.
- [2025.10] 🎯 [확장성 강화]: 처리 병목 현상을 제거하여 **대규모 데이터셋을 효율적으로** 지원.
- [2025.09] 🎯 [새 기능]: Qwen3-30B-A3B 등 **오픈소스 LLM**의 지식 그래프 추출 정확도 향상.
- [2025.08] 🎯 [새 기능]: **리랭커(Reranker)** 지원 추가 — 혼합 쿼리 성능 대폭 향상 (기본 쿼리 모드로 설정).
- [2025.08] 🎯 [새 기능]: 자동 KG 재생성을 포함한 **문서 삭제** 기능 추가.
- [2025.06] 🎯 [새 릴리스]: [RAG-Anything](https://github.com/HKUDS/RAG-Anything) 출시 — 텍스트, 이미지, 표, 수식을 원활하게 처리하는 **올인원 멀티모달 RAG** 시스템.
- [2025.03] 🎯 [새 기능]: 출처 표기 및 문서 추적성 향상을 위한 인용(Citation) 기능 추가.
- [2025.02] 🎯 [새 기능]: **MongoDB**를 통합 스토리지 솔루션으로 사용 가능.
- [2025.02] 🎯 [새 릴리스]: [VideoRAG](https://github.com/HKUDS/VideoRAG) 출시 — 초장기 컨텍스트 비디오 이해 RAG 시스템.
- [2025.01] 🎯 [새 릴리스]: [MiniRAG](https://github.com/HKUDS/MiniRAG) 출시 — 소형 모델로 RAG를 더 간단하게.
- [2025.01] 🎯 **PostgreSQL**을 통합 스토리지 솔루션으로 사용 가능.
- [2024.11] 🎯 [새 기능]: LightRAG WebUI 출시 — 직관적인 웹 대시보드로 문서 삽입, 쿼리, 지식 그래프 시각화.
- [2024.11] 🎯 [새 기능]: [Neo4J 스토리지 사용](https://github.com/HKUDS/LightRAG?tab=readme-ov-file#using-neo4j-for-storage) 추가 — 그래프 데이터베이스 지원.
- [2024.10] 🎯 [새 채널]: [Discord 채널](https://discord.gg/yF2MmDJyGJ) 개설! 커뮤니티에 참여하세요.

<details>
  <summary style="font-size: 1.4em; font-weight: bold; cursor: pointer; display: list-item;">
    알고리즘 순서도
  </summary>

![LightRAG 인덱싱 순서도](https://learnopencv.com/wp-content/uploads/2024/11/LightRAG-VectorDB-Json-KV-Store-Indexing-Flowchart-scaled.jpg)
*그림 1: LightRAG 인덱싱 순서도 - [출처](https://learnopencv.com/lightrag/)*

![LightRAG 검색 및 쿼리 순서도](https://learnopencv.com/wp-content/uploads/2024/11/LightRAG-Querying-Flowchart-Dual-Level-Retrieval-Generation-Knowledge-Graphs-scaled.jpg)
*그림 2: LightRAG 검색 및 쿼리 순서도 - [출처](https://learnopencv.com/lightrag/)*

</details>

---

## 설치

**💡 uv 패키지 관리자 사용**: 이 프로젝트는 빠르고 안정적인 Python 패키지 관리를 위해 [uv](https://docs.astral.sh/uv/)를 사용합니다. 먼저 uv를 설치하세요:
- Unix/macOS: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Windows: `powershell -c "irm https://astral.sh/uv/install.ps1 | iex"`

> **참고**: pip도 사용 가능하지만, 더 나은 성능과 안정적인 의존성 관리를 위해 uv를 권장합니다.
>
> **📦 오프라인 배포**: 인터넷 연결이 제한된 환경의 경우 [오프라인 배포 가이드](./docs/OfflineDeployment-ko.md)를 참조하세요.

### LightRAG 서버 설치

LightRAG 서버는 Web UI와 API 지원을 제공합니다. Web UI는 문서 인덱싱, 지식 그래프 탐색, 간단한 RAG 쿼리 인터페이스를 제공합니다. 또한 Ollama 호환 인터페이스를 제공하여 Open WebUI 같은 AI 챗봇이 LightRAG에 쉽게 접근할 수 있습니다.

* PyPI에서 설치

```bash
### uv를 사용하여 LightRAG 서버 설치 (권장)
uv tool install "lightrag-hku[api]"

### 또는 pip 사용
# python -m venv .venv
# source .venv/bin/activate  # Windows: .venv\Scripts\activate
# pip install "lightrag-hku[api]"

### 프론트엔드 빌드
cd lightrag_webui
bun install --frozen-lockfile
bun run build
cd ..

# 환경 파일 설정
cp env.example .env  # LLM 및 임베딩 설정으로 업데이트
# 서버 시작
lightrag-server
```

* 소스에서 설치

```bash
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG

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

# 환경 파일 설정
make env-base  # 또는: cp env.example .env 후 수동 수정
# 서버 시작
lightrag-server
```

* Docker Compose로 시작

```bash
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG
cp env.example .env  # LLM 및 임베딩 설정 수정
docker compose up
```

### 설정 마법사로 .env 파일 생성

`env.example`을 직접 편집하는 대신, 대화형 설정 마법사를 사용하여 `.env`를 생성할 수 있습니다:

```bash
make env-base           # 필수 첫 단계: LLM, 임베딩, 리랭커
make env-storage        # 선택: 스토리지 백엔드 및 데이터베이스 서비스
make env-server         # 선택: 서버 포트, 인증, SSL
make env-security-check # 선택: 현재 .env 보안 감사
```

자세한 내용은 [docs/InteractiveSetup-ko.md](./docs/InteractiveSetup-ko.md)를 참조하세요.

### LightRAG 코어 설치

* 소스에서 설치 (권장)

```bash
cd LightRAG
uv sync
source .venv/bin/activate  # Linux/macOS
# 또는: pip install -e .
```

* PyPI에서 설치

```bash
uv pip install lightrag-hku
# 또는: pip install lightrag-hku
```

---

## 빠른 시작

### LightRAG의 LLM 및 기술 스택 요구사항

LightRAG는 문서에서 엔티티-관계 추출 작업을 수행해야 하므로, 일반 RAG보다 LLM 성능에 대한 요구 사항이 훨씬 높습니다.

- **LLM 선택**:
  - 최소 320억(32B) 파라미터 이상의 LLM 사용 권장
  - 컨텍스트 길이: 최소 32KB, 64KB 권장
  - 문서 인덱싱 단계에서는 추론 모델 사용 비권장
  - 쿼리 단계에서는 인덱싱 단계보다 더 강력한 모델 사용 권장
- **임베딩 모델**:
  - 고성능 임베딩 모델이 RAG에 필수적
  - 권장 모델: `BAAI/bge-m3`, `text-embedding-3-large`
  - **중요**: 임베딩 모델은 문서 인덱싱 전에 결정해야 하며, 쿼리 단계에서도 동일한 모델을 사용해야 합니다. 임베딩 모델 변경 시 벡터 관련 테이블/저장소를 삭제하고 새 차원으로 재생성해야 합니다.
- **리랭커 모델 설정**:
  - 리랭커 설정 시 LightRAG 검색 성능이 크게 향상됩니다
  - 리랭커 활성화 시 기본 쿼리 모드를 "mix"로 설정 권장
  - 권장 모델: `BAAI/bge-reranker-v2-m3` 또는 Jina 리랭커

### LightRAG 서버 빠른 시작

LightRAG 서버는 Web UI와 API 지원을 위해 설계되었습니다. 다양한 중력 레이아웃, 노드 쿼리, 서브그래프 필터링 등을 지원하는 지식 그래프 시각화 기능을 제공합니다. 자세한 내용은 [LightRAG 서버 문서](./docs/LightRAG-API-Server-ko.md)를 참조하세요.

### LightRAG 코어 빠른 시작

`examples` 폴더의 샘플 코드를 참조하세요. OpenAI API 키가 있다면 즉시 데모를 실행할 수 있습니다:

```bash
cd LightRAG
export OPENAI_API_KEY="sk-...your_openai_key..."
# "크리스마스 캐럴" 데모 문서 다운로드
curl https://raw.githubusercontent.com/gusye1234/nano-graphrag/main/tests/mock_data.txt > ./book.txt
# 데모 실행
python examples/lightrag_openai_demo.py
```

**참고 1**: 서로 다른 임베딩 모델을 사용하는 테스트 스크립트 간에 전환 시, `./dickens` 데이터 디렉토리를 반드시 삭제해야 합니다. LLM 캐시를 유지하려면 `kv_store_llm_response_cache.json` 파일만 보존하고 나머지를 삭제하세요.

**참고 2**: `lightrag_openai_demo.py`와 `lightrag_openai_compatible_demo.py`만 공식 지원 샘플입니다.

---

## LightRAG 코어로 프로그래밍

초기화 파라미터, `QueryParam`, LLM/임베딩 제공자 예제(OpenAI, Ollama, Azure, Gemini, HuggingFace, LlamaIndex), 리랭커 주입, 삽입 연산, 엔티티/관계 관리, 삭제/병합 등 전체 Core API 레퍼런스는 **[docs/ProgramingWithCore-ko.md](./docs/ProgramingWithCore-ko.md)**를 참조하세요.

> ⚠️ **프로젝트에 LightRAG를 통합하려면 LightRAG 서버가 제공하는 REST API를 활용하는 것을 권장합니다.** LightRAG Core는 임베디드 애플리케이션이나 연구/평가 목적으로 주로 사용됩니다.

### 고급 기능

토큰 사용량 추적, 지식 그래프 데이터 내보내기, LLM 캐시 관리, Langfuse 옵저버빌리티 통합, RAGAS 기반 평가 등의 추가 기능은 **[docs/AdvancedFeatures-ko.md](./docs/AdvancedFeatures-ko.md)**를 참조하세요.

### 멀티모달 문서 처리 (RAG-Anything 통합)

LightRAG는 [RAG-Anything](https://github.com/HKUDS/RAG-Anything)과 통합하여 PDF, Office 문서, 이미지, 표, 수식 등 다양한 형식의 엔드투엔드 멀티모달 RAG를 지원합니다. 설정 및 사용 예제는 **[docs/AdvancedFeatures-ko.md](./docs/AdvancedFeatures-ko.md)**를 참조하세요.

---

## 논문 결과 재현

LightRAG는 농업, 컴퓨터과학, 법률, 혼합 도메인에서 NaiveRAG, RQ-RAG, HyDE, GraphRAG를 일관되게 능가합니다. 전체 평가 방법론, 프롬프트, 재현 단계는 **[docs/Reproduce-ko.md](./docs/Reproduce-ko.md)**를 참조하세요.

**전체 성능 표**

||**농업**||**CS**||**법률**||**혼합**||
|----------------------|---------------|------------|------|------------|---------|------------|-------|------------|
||NaiveRAG|**LightRAG**|NaiveRAG|**LightRAG**|NaiveRAG|**LightRAG**|NaiveRAG|**LightRAG**|
|**포괄성**|32.4%|**67.6%**|38.4%|**61.6%**|16.4%|**83.6%**|38.8%|**61.2%**|
|**다양성**|23.6%|**76.4%**|38.0%|**62.0%**|13.6%|**86.4%**|32.4%|**67.6%**|
|**역량강화**|32.4%|**67.6%**|38.8%|**61.2%**|16.4%|**83.6%**|42.8%|**57.2%**|
|**전체**|32.4%|**67.6%**|38.8%|**61.2%**|15.2%|**84.8%**|40.0%|**60.0%**|

---

## 🔗 관련 프로젝트

| 프로젝트 | 설명 |
|---------|------|
| [RAG-Anything](https://github.com/HKUDS/RAG-Anything) | 멀티모달 RAG |
| [VideoRAG](https://github.com/HKUDS/VideoRAG) | 초장기 컨텍스트 비디오 RAG |
| [MiniRAG](https://github.com/HKUDS/MiniRAG) | 초간단 RAG |

---

## 🤝 기여

버그 수정, 새 기능, 문서 개선 등 모든 형태의 기여를 환영합니다.
PR 제출 전에 [기여 가이드](.github/CONTRIBUTING-ko.md)를 먼저 읽어주세요.

---

## 📖 인용

```python
@article{guo2024lightrag,
title={LightRAG: Simple and Fast Retrieval-Augmented Generation},
author={Zirui Guo and Lianghao Xia and Yanhua Yu and Tu Ao and Chao Huang},
year={2024},
eprint={2410.05779},
archivePrefix={arXiv},
primaryClass={cs.IR}
}
```

---

<div align="center">
  <span>LightRAG를 방문해 주셔서 감사합니다!</span>
</div>

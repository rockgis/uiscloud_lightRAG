# 저장소 가이드라인

LightRAG는 그래프 기반 지식 표현을 통해 정보 검색 및 생성을 향상시키도록 설계된 고급 검색 증강 생성(RAG) 프레임워크입니다.

## 프로젝트 구조 및 모듈 구성

- `lightrag/`: 오케스트레이터(`lightrag/lightrag.py`), `kg/`의 스토리지 어댑터, `llm/`의 LLM 바인딩, `operate.py` 및 `utils_*.py` 같은 헬퍼를 포함한 핵심 Python 패키지.
- `lightrag-api/`: `routers/`의 라우터와 Gunicorn 런처 `run_with_gunicorn.py`를 포함한 FastAPI 서비스(`lightrag_server.py`).
- `lightrag_webui/`: Bun + Vite로 구동되는 React 19 + TypeScript 클라이언트; UI 컴포넌트는 `src/`에 있습니다.
- `scripts/setup/`: 대화형 환경 설정 마법사. `setup.sh`는 단계적 `--base` / `--storage` / `--server` / 검증 흐름을 조율하고, `lib/`에는 프롬프트/검증/파일 헬퍼가, `templates/*.yml`에는 번들 서비스용 컴포즈 조각이 포함됩니다.
- 테스트는 `tests/` 및 루트 수준 `test_*.py`에 있습니다. 작업 데이터셋은 `inputs/`, `rag_storage/`, `temp/`에, 배포 관련 파일은 `docs/`, `k8s-deploy/`, `docker-compose.yml`에 있습니다.
- `Makefile`: 설정 마법사 및 로컬 개발자 단축키의 표준 진입점; 임시 셸 스니펫 대신 문서화된 타겟을 사용하세요.

## 빌드, 테스트 및 개발 명령

- `python -m venv .venv && source .venv/bin/activate`: Python 런타임 설정.
- `pip install -e .` / `pip install -e .[api]`: 패키지 및 API extras를 편집 가능한 모드로 설치.
- `make env-base`: LLM, 임베딩 및 리랭커 설정을 위한 첫 실행 대화형 설정; `.env`와 선택적으로 `docker-compose.final.yml`을 작성합니다.
- `make env-storage`, `make env-server`: 스토리지 백엔드 및 서버/보안/SSL 설정을 위한 선택적 후속 마법사 단계.
- `make env-validate`, `make env-security-check`, `make env-backup`: 설정 마법사를 통해 현재 `.env`를 검증, 감사 또는 백업.
- `lightrag-server` 또는 `uvicorn lightrag.api.lightrag_server:app --reload`: API를 로컬에서 시작; `.env`가 있는지 확인.
- `python -m pytest tests` (오프라인 마커 기본 적용) 또는 통합 테스트 포함, 개별 스크립트 타겟.
- `ruff check .`: 커밋 전 Python 소스 린트.
- 프론트엔드 워크플로우는 `lightrag_webui/`에서 Bun 사용.

## 코딩 스타일 및 명명 규칙

- 백엔드 코드는 4칸 들여쓰기의 PEP 8을 따르고, 함수에 타입 어노테이션을 달며, 상태 모델링에는 데이터클래스를 사용합니다.
- `print` 대신 `lightrag.utils.logger`를 사용하세요.
- 저장소 또는 파이프라인 추상화는 `lightrag.base`를 통해 확장하고 재사용 가능한 헬퍼는 기존 `utils_*.py`에 유지합니다.
- Python 모듈은 밑줄이 있는 소문자로, React 컴포넌트는 `PascalCase.tsx`와 훅 우선 패턴을 사용합니다.
- 프론트엔드 코드는 2칸 들여쓰기의 TypeScript로, 훅을 사용하는 함수형 React 컴포넌트, Tailwind 유틸리티 스타일을 따릅니다.

## 테스트 가이드라인

- pytest 추가는 건드린 코드와 가까운 위치에 (`tests/`는 기능 폴더를 미러링하고 루트 수준 `test_*.py` 헬퍼가 있습니다); 함수 이름은 `test_`로 시작해야 합니다.
- `tests/pytest.ini`를 따르세요: 마커는 `offline`, `integration`, `requires_db`, `requires_api`이며, 스위트는 기본적으로 `-m "not integration"`으로 실행됩니다. 외부 서비스를 사용할 수 있으면 `--run-integration`을 전달하거나 `LIGHTRAG_RUN_INTEGRATION=true`를 설정하세요.
- `tests/conftest.py`의 커스텀 CLI 토글 사용: `--keep-artifacts`, `--stress-test`, `--test-workers N`.
- UI 업데이트의 경우 `bun:test`를 통한 Bun 테스트 커버리지와 변경 사항을 페어링하세요.

## 커밋 및 풀 리퀘스트 가이드라인

- 간결한 명령형 커밋 제목을 사용하고 (예: `Fix lock key normalization`) 필요한 경우에만 본문에 컨텍스트를 추가하세요.
- PR에는 요약, 운영 영향, 연결된 이슈, 사용자 대면 작업의 스크린샷 또는 API 샘플이 포함되어야 합니다.
- 리뷰 요청 전에 `ruff check .`, `python -m pytest`, 영향받은 프론트엔드 명령이 성공하는지 확인하세요.
- 이 저장소는 `HKUDS/LightRAG`의 포크입니다. PR 생성 시 항상 **`HKUDS/LightRAG:main`** (업스트림)을 타겟으로 하세요.

## 보안 및 설정 팁

- `.env.example`을 복사하고 절대 시크릿이나 실제 연결 문자열을 커밋하지 마세요.
- `LIGHTRAG_*` 변수를 통해 스토리지 백엔드를 설정하고 필요한 경우 `docker-compose` 서비스로 검증하세요.
- `lightrag.log*`를 로컬 아티팩트로 취급하세요; 로그나 출력을 공유하기 전에 민감한 정보를 제거하세요.

## 자동화 및 에이전트 워크플로우

- 모든 셸 명령에 저장소 상대 `workdir` 인수를 사용하고, CLI 하네스에서 더 빠른 `rg`/`rg --files`를 검색에 사용하세요.
- 기본 편집은 ASCII로, 단일 파일 변경에는 `apply_patch`를 사용하고, 복잡한 로직 이해에 도움이 되는 간결한 주석만 추가하세요.
- 기존 로컬 수정 사항을 존중하세요; 명시적으로 요청받지 않는 한 사용자 변경 사항을 되돌리거나 버리지 마세요.
- 사소한 수정에는 계획 도구를 건너뛰고, 복잡한 작업에는 다단계 계획을 제공하며 진행에 따라 계획을 업데이트하세요.
- 관련 `ruff`/`pytest`/`bun test` 명령을 실행하여 변경 사항을 검증하세요.
- 설정 워크플로우 변경 시 `scripts/setup/setup.sh`를 직접 호출하는 대신 `make env-*` 타겟을 사용하세요.

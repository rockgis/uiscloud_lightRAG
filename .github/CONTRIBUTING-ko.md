# LightRAG 기여 가이드

기여에 관심을 가져주셔서 감사합니다! 이 가이드는 시작하는 데 필요한 모든 것을 안내합니다.

## 목차

- [기여 방법](#기여-방법)
- [개발 환경 설정](#개발-환경-설정)
- [코드 스타일](#코드-스타일)
- [테스트 실행](#테스트-실행)
- [풀 리퀘스트 제출](#풀-리퀘스트-제출)
- [버그 신고](#버그-신고)
- [기능 요청](#기능-요청)

---

## 기여 방법

- **버그 신고** — 버그 신고 템플릿으로 [이슈](https://github.com/HKUDS/LightRAG/issues) 작성
- **기능 요청** — 기능 요청 템플릿으로 [이슈](https://github.com/HKUDS/LightRAG/issues) 작성
- **문서화** — 오타 수정, 설명 명확화, 예제 추가
- **코드** — 버그 수정, 기능 구현, 스토리지/LLM 백엔드 추가
- **테스트** — 미테스트 코드 경로에 대한 테스트 커버리지 추가

---

## 개발 환경 설정

```bash
# 저장소 클론
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG

# 개발 모드로 설치 (uv 필요)
uv sync
source .venv/bin/activate        # Linux/macOS
# .venv\Scripts\activate         # Windows

# 필요에 따라 선택적 extras 설치
uv sync --extra api              # FastAPI 서버
uv sync --extra test             # 테스트 의존성
uv sync --extra offline-storage  # 스토리지 백엔드
uv sync --extra offline-llm      # 추가 LLM 제공자

# pre-commit 훅 설정 (한 번만)
pip install pre-commit
pre-commit install
```

---

## 코드 스타일

이 프로젝트는 [Ruff](https://docs.astral.sh/ruff/)를 형식화 및 린팅에 사용하며, [pre-commit](https://pre-commit.com/)으로 적용됩니다.

### 자동 수정

`pre-commit run --all-files`를 실행하면 대부분의 스타일 문제가 자동으로 수정됩니다:

```bash
# 모든 파일 수정
pre-commit run --all-files

# 스테이징된 파일만 수정 (개발 중 빠른 방법)
pre-commit run
```

### 검사 항목

| 훅 | 역할 |
|------|-------------|
| `trailing-whitespace` | 후행 공백 제거 |
| `end-of-file-fixer` | 파일이 줄 바꿈으로 끝나도록 보장 |
| `requirements-txt-fixer` | `requirements.txt` 항목을 정렬 상태로 유지 |
| `ruff-format` | Python 코드 형식화 (Black 호환) |
| `ruff` | Python 린트 오류 수정 |

### CI 검사

동일한 검사가 모든 풀 리퀘스트에서 자동으로 실행됩니다. CI 검사가 실패하면 로컬에서 `pre-commit run --all-files`를 실행하고, 수정 사항을 커밋하여 다시 push하세요.

### 언어 규칙

- **Python 코드 및 주석**: 영어
- **프론트엔드 (WebUI)**: 문자열을 하드코딩하는 대신 i18next 번역 키를 추가하세요

---

## 테스트 실행

```bash
# 오프라인 테스트 실행 (외부 서비스 불필요)
python -m pytest tests

# 통합 테스트 실행 (설정된 외부 서비스 필요)
python -m pytest tests --run-integration

# 특정 테스트 파일 실행
python -m pytest tests/test_lightrag.py

# 디버깅을 위해 테스트 아티팩트 유지
python -m pytest tests --keep-artifacts
```

`--run-integration`의 대안으로 환경 변수 `LIGHTRAG_RUN_INTEGRATION=true`를 설정하세요.

---

## 풀 리퀘스트 제출

1. 저장소를 **포크**하고 `main`에서 브랜치를 생성하세요:
   ```bash
   git checkout -b fix/your-descriptive-branch-name
   ```

2. **변경 사항을 적용**하고 다음을 확인하세요:
   - Pre-commit 검사 통과: `pre-commit run --all-files`
   - 관련 테스트 통과: `python -m pytest tests`
   - 해당되는 경우 새 동작이 테스트로 커버됨

3. 변경이 *왜* 이루어졌는지 설명하는 명확한 메시지로 **커밋**하세요:
   ```bash
   git commit -m "fix: 암호 없이 권한 전용 암호화된 PDF 처리"
   ```

4. **Push**하고 `main`을 대상으로 풀 리퀘스트를 열어 PR 템플릿을 완전히 작성하세요.

5. **리뷰 피드백에 응답**하세요 — 유지 관리자가 PR을 검토하고 변경을 요청할 수 있습니다.

### 풀 리퀘스트 체크리스트

- [ ] 로컬에서 변경 사항 테스트됨
- [ ] Pre-commit 검사 통과 (`pre-commit run --all-files`)
- [ ] 해당되는 경우 단위/통합 테스트 추가 또는 업데이트됨
- [ ] 동작 변경 시 문서 업데이트됨
- [ ] PR 설명이 *무엇*이 아닌 *왜*를 설명함

---

## 버그 신고

[버그 신고 이슈 템플릿](https://github.com/HKUDS/LightRAG/issues/new?template=bug_report.yml)을 사용하세요. 다음을 포함하세요:

- LightRAG 버전 및 Python 버전
- 사용 중인 스토리지 백엔드 및 LLM 제공자
- 최소 재현 가능한 예제
- 전체 오류 추적

---

## 기능 요청

[기능 요청 이슈 템플릿](https://github.com/HKUDS/LightRAG/issues/new?template=feature_request.yml)을 사용하세요. 다음을 설명하세요:

- 해결하려는 문제
- 제안하는 해결책
- 고려한 대안

---

## 질문

사용 관련 질문은 [Discussions](https://github.com/HKUDS/LightRAG/discussions) 탭을 확인하거나 [Question 이슈](https://github.com/HKUDS/LightRAG/issues/new?template=question.yml)를 작성하세요.

# 프론트엔드 빌드 가이드

## 개요

LightRAG 프로젝트에는 React 기반의 WebUI 프론트엔드가 포함됩니다. 이 가이드는 다양한 시나리오에서 프론트엔드 빌드가 어떻게 작동하는지 설명합니다.

## 핵심 원칙

- **Git 저장소**: 프론트엔드 빌드 결과물은 **포함되지 않습니다** (저장소를 깔끔하게 유지)
- **PyPI 패키지**: 프론트엔드 빌드 결과물이 **포함됩니다** (바로 사용 가능)
- **빌드 도구**: **Bun**이 권장되지만, **Node.js/npm**도 대체로 완전 지원됩니다

## 설치 시나리오

### 1. 최종 사용자 (PyPI에서) ✨

**명령:**
```bash
pip install lightrag-hku[api]
```

**무슨 일이 일어나나요:**
- 프론트엔드가 이미 빌드되어 패키지에 포함됩니다
- 추가 단계가 필요하지 않습니다
- 웹 인터페이스가 즉시 작동합니다

---

### 2. 개발 모드 (기여자에게 권장) 🔧

**명령:**
```bash
# 저장소 클론
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG

# 편집 가능한 모드로 설치 (아직 프론트엔드 빌드 불필요)
pip install -e ".[api]"

# 필요할 때 프론트엔드 빌드 (언제든지 가능)
cd lightrag_webui
bun install --frozen-lockfile
bun run build
cd ..
```

**장점:**
- 먼저 설치 후 나중에 빌드 (유연한 워크플로우)
- 변경사항이 즉시 적용됩니다 (심볼릭 링크 모드)
- 재설치 없이 언제든지 프론트엔드 재빌드 가능

**작동 방식:**
- 소스 디렉토리에 심볼릭 링크를 생성합니다
- 프론트엔드 빌드 출력이 `lightrag/api/webui/`로 이동합니다
- 설치된 패키지에서 변경사항이 즉시 표시됩니다

---

### 3. 일반 설치 (패키지 빌드 테스트) 📦

**명령:**
```bash
# 저장소 클론
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG

# ⚠️ 먼저 프론트엔드를 빌드해야 합니다
cd lightrag_webui
bun install --frozen-lockfile
bun run build
cd ..

# 이제 설치
pip install ".[api]"
```

**무슨 일이 일어나나요:**
- 프론트엔드 파일이 site-packages로 **복사**됩니다
- 빌드 후 수정사항이 설치된 패키지에 영향을 미치지 않습니다
- 업데이트하려면 재빌드 + 재설치가 필요합니다

**언제 사용하나요:**
- 전체 설치 프로세스 테스트
- 패키지 설정 검증
- PyPI 사용자 경험 시뮬레이션

---

### 4. 배포 패키지 생성 🚀

**명령:**
```bash
# 먼저 프론트엔드 빌드
cd lightrag_webui
bun install --frozen-lockfile --production
bun run build
cd ..

# 배포 패키지 생성
python -m build

# 출력: dist/lightrag_hku-*.whl 및 dist/lightrag_hku-*.tar.gz
```

**무슨 일이 일어나나요:**
- `setup.py`가 프론트엔드 빌드 여부를 확인합니다
- 없으면 도움이 되는 오류 메시지와 함께 설치가 실패합니다
- 생성된 패키지에 모든 프론트엔드 파일이 포함됩니다

---

## GitHub Actions (자동 릴리스)

GitHub에서 릴리스를 생성할 때:

1. **자동으로 Bun을 사용하여 프론트엔드를 빌드**합니다
2. **빌드 완료 여부를 검증**합니다
3. 프론트엔드가 포함된 **Python 패키지를 생성**합니다
4. 기존 신뢰할 수 있는 게시자 설정을 사용하여 **PyPI에 게시**합니다

**수동 개입이 필요하지 않습니다!**

---

## 빠른 참조

| 시나리오 | 명령 | 프론트엔드 필요 | 나중에 빌드 가능 |
|----------|---------|-------------------|-----------------|
| PyPI에서 | `pip install lightrag-hku[api]` | 포함됨 | 아니요 (이미 설치됨) |
| 개발 | `pip install -e ".[api]"` | 아니요 | ✅ 예 (언제든지) |
| 일반 설치 | `pip install ".[api]"` | ✅ 예 (사전에) | 아니요 (재설치 필요) |
| 패키지 생성 | `python -m build` | ✅ 예 (사전에) | 해당 없음 |

---

## Bun 설치

Bun이 설치되지 않은 경우:

```bash
# macOS/Linux
curl -fsSL https://bun.sh/install | bash

# Windows
powershell -c "irm bun.sh/install.ps1 | iex"
```

공식 문서: https://bun.sh

---

## 파일 구조

```
LightRAG/
├── lightrag_webui/          # 프론트엔드 소스 코드
│   ├── src/                 # React 컴포넌트
│   ├── package.json         # 의존성
│   └── vite.config.ts       # 빌드 설정
│       └── outDir: ../lightrag/api/webui  # 빌드 출력
│
├── lightrag/
│   └── api/
│       └── webui/           # 프론트엔드 빌드 출력 (gitignore에 포함)
│           ├── index.html   # 빌드된 파일 (bun run build 실행 후)
│           └── assets/      # 빌드된 에셋
│
├── setup.py                 # 빌드 확인
├── pyproject.toml           # 패키지 설정
└── .gitignore               # lightrag/api/webui/*를 제외 (.gitkeep 제외)
```

---

## 문제 해결

### Q: 개발 모드로 설치했는데 웹 인터페이스가 작동하지 않습니다

**A:** 프론트엔드를 빌드하세요:
```bash
cd lightrag_webui && bun run build
```

### Q: 프론트엔드를 빌드했는데 설치된 패키지에 없습니다

**A:** 빌드 후 `pip install .`을 사용했을 가능성이 높습니다. 다음 중 하나를 선택하세요:
- 개발용: `pip install -e ".[api]"` 사용
- 재설치: `pip uninstall lightrag-hku && pip install ".[api]"`

### Q: 빌드된 프론트엔드 파일은 어디에 있나요?

**A:** `bun run build` 실행 후 `lightrag/api/webui/`에 있습니다

### Q: Bun 대신 npm이나 yarn을 사용할 수 있나요?

**A:** 네. 빌드 스크립트(`dev`, `build`, `preview`, `lint`)는 런타임에 독립적이며 Bun과 Node.js/npm 모두에서 작동합니다:
```bash
npm install
npm run build
```
Bun이 속도 면에서 권장되지만 npm도 완전히 지원됩니다. 테스트(`bun test`)는 여전히 Bun이 필요합니다.

### Q: `Cannot find package '@/lib'` 오류로 빌드가 실패합니다

**A:** 이는 `vite.config.ts`가 Bun만 설정 로드 시점에 해결할 수 있는 TypeScript 경로 별칭(`@/`)을 사용하여 발생했습니다. 상대적 임포트로 수정된 최신 버전으로 업데이트하세요.

---

## 요약

✅ **PyPI 사용자**: 별도 작업 불필요, 프론트엔드 포함됨
✅ **개발자**: `pip install -e ".[api]"` 사용, 필요할 때 프론트엔드 빌드
✅ **CI/CD**: GitHub Actions에서 자동 빌드
✅ **Git**: 프론트엔드 빌드 출력이 절대 커밋되지 않음

질문이나 문제가 있으면 GitHub 이슈를 열어주세요.

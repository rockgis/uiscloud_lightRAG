# 대화형 설정 가이드

`.env`를 직접 편집하는 대신 LightRAG 설정 마법사를 사용하면 단계별 안내를 받아 설정할 수 있습니다.

마법사는 `make` 타겟으로 실행됩니다:

- `make env-base`
- `make env-storage`
- `make env-server`
- `make env-validate`
- `make env-security-check`
- `make env-backup`
- `make env-base-rewrite`
- `make env-storage-rewrite`

기반 셸 스크립트를 직접 호출할 필요가 없습니다.

## 마법사의 용도

설정 마법사는 세 부분으로 LightRAG를 설정합니다:

- `env-base`: LLM, 임베딩 모델, 선택적 리랭커를 설정합니다.
- `env-storage`: PostgreSQL, Neo4j, Redis, Milvus, Qdrant, MongoDB, Memgraph 등의 스토리지 백엔드를 추가하거나 변경합니다.
- `env-server`: 서버 호스트 및 포트, WebUI 레이블, 인증, API 키, SSL을 설정합니다.

각 단계를 나중에 다시 실행할 수 있습니다. 마법사는 기존 `.env`를 로드하고 현재 값을 기본값으로 표시하므로 변경이 필요한 항목만 수정하면 됩니다.

## 시작 전 확인 사항

- 저장소 루트에서 명령을 실행하세요.
- `make env-*` 타겟은 자동으로 호환 가능한 Bash 4+ 인터프리터를 선택합니다.
- 설정 스크립트를 직접 호출하는 대신 문서화된 `make env-*` 타겟을 사용하세요.
- `make env-base`가 초기 `.env`를 생성하는 일반적인 시작점입니다.
- `make env-storage`와 `make env-server`는 기존 `.env`가 필요합니다.
- 마법사 관리 Docker 서비스를 선택하면 마법사가 Docker 시작 경로도 준비합니다.

## 설정 경로 선택

빠른 가이드:

- 원격 모델 제공자로 최소한의 첫 실행을 원하는 경우: `make env-base`
- Docker에서 임베딩 또는 리랭킹을 로컬에서 실행하려는 경우: `make env-base`
- 모델 설정 완료 후 데이터베이스를 추가하려는 경우: `make env-storage`
- 모델 설정 완료 후 인증, API 키, SSL을 추가하려는 경우: `make env-server`
- 현재 설정이 유효한지 확인하려는 경우: `make env-validate`
- 노출 전 현재 설정을 감사하려는 경우: `make env-security-check`
- 설정 변경 없이 독립적인 백업을 만들려는 경우: `make env-backup`
- 번들 템플릿에서 생성된 컴포즈 서비스를 복구해야 하는 경우: `make env-base-rewrite` 또는 `make env-storage-rewrite`

## 시나리오 1: 최초 로컬 설정

원격 모델 엔드포인트 또는 API 키가 있고 최소한의 설정으로 LightRAG를 실행하려는 경우.

**명령**

```bash
make env-base
```

**마법사 질문 내용**

- LLM 제공자, 모델, 엔드포인트, API 키
- 임베딩 모델을 Docker를 통해 로컬에서 실행할지 여부
- 임베딩이 원격인 경우: 임베딩 제공자, 모델, 차원, 엔드포인트, API 키
- 리랭킹 활성화 여부
- 리랭킹 활성화 시: 리랭크 서비스를 Docker를 통해 로컬에서 실행할지 여부
- 리랭킹이 원격인 경우: 리랭크 제공자, 모델, 엔드포인트, API 키

**작성되는 내용**

- `.env`
- `docker-compose.final.yml` (마법사 관리 Docker 서비스를 활성화한 경우에만)

**다음 단계**

- 마법사 관리 Docker 서비스를 활성화하지 않은 경우:

```bash
lightrag-server
```

- 마법사 관리 Docker 서비스를 활성화한 경우:

```bash
docker compose -f docker-compose.final.yml up -d
```

## 시나리오 2: Docker 호스팅 임베딩 또는 리랭크를 포함한 로컬 설정

Docker를 통해 임베딩 및/또는 리랭킹을 위한 로컬 추론 서비스를 실행하려는 경우.

**명령**

```bash
make env-base
```

**권장 답변**

- 로컬 임베딩을 원하면 `임베딩 모델을 Docker(vLLM)로 로컬에서 실행할까요?`에 `yes`
- 로컬 리랭킹을 원하면 `리랭킹을 활성화할까요?` 및 `리랭크 서비스를 Docker로 로컬에서 실행할까요?`에 모두 `yes`

**다음 단계**

```bash
docker compose -f docker-compose.final.yml up -d
```

## 시나리오 3: 기본 설정 후 스토리지 추가

`make env-base`로 생성된 `.env`가 있고, 기본 로컬 파일 스토리지에서 데이터베이스 기반 스토리지로 전환하려는 경우.

**명령**

```bash
make env-storage
```

**사전 조건**

- `.env`가 이미 존재해야 합니다

**마법사 질문 내용**

- KV 스토리지 백엔드
- 벡터 스토리지 백엔드
- 그래프 스토리지 백엔드
- 문서 상태 스토리지 백엔드
- 각 필요한 데이터베이스에 대해 Docker를 통해 로컬에서 실행할지 여부
- 각 데이터베이스에 필요한 연결 세부 정보 (호스트, URI, 포트, 사용자, 비밀번호, 데이터베이스 이름 등)

**중요 규칙**

- 벡터 스토리지로 `MongoVectorDBStorage`를 선택하면 마법사는 번들 로컬 Docker MongoDB 서비스를 제공하지 않습니다. Atlas Search / Vector Search를 지원하는 MongoDB 배포를 직접 제공해야 합니다.

**다음 단계**

- Docker 관리 스토리지 서비스를 선택한 경우:

```bash
docker compose -f docker-compose.final.yml up -d
```

- 외부 데이터베이스를 지정한 경우 LightRAG를 시작하기 전에 해당 서비스가 접근 가능한지 확인하세요.

## 시나리오 4: 인증 및 SSL로 배포 강화

`.env`가 있고 공유 또는 외부 사용을 위해 서버를 준비해야 하는 경우.

**명령**

```bash
make env-server
make env-security-check
```

**사전 조건**

- `.env`가 이미 존재해야 합니다

**`env-server` 질문 내용**

- 서버 호스트 및 포트
- WebUI 제목 및 설명
- 요약 언어
- 인증 및 API 키 설정 구성 여부
- 인증 계정, JWT 시크릿, 토큰 유효기간, API 키, 화이트리스트 경로
- SSL/TLS 활성화 여부
- SSL 인증서 파일 경로 및 SSL 키 파일 경로

**다음 단계**

- `make env-security-check` 실행
- 스택이 Docker를 사용하는 경우 컴포즈 파일로 LightRAG 서비스 재생성
- 스택이 호스트에서 실행되는 경우 `lightrag-server` 재시작

## 검증, 감사 및 백업

이 명령들은 전체 설정 흐름을 거치지 않지만 일반 운영의 일부입니다.

### 현재 설정 검증

```bash
make env-validate
```

현재 `.env`가 내부적으로 일관성이 있는지 확인합니다. 누락된 필수 값, 잘못된 인증 설정, 잘못된 URI, 잘못된 포트, 누락된 SSL 파일 등의 문제를 보고합니다.

### 노출 전 보안 감사

```bash
make env-security-check
```

localhost 외부에 LightRAG를 노출하기 전에 실행하세요. 인증 누락, 약하거나 누락된 JWT 시크릿, 안전하지 않은 화이트리스트 설정, 해결되지 않은 민감한 플레이스홀더 등의 위험한 설정을 보고합니다.

### 독립적인 백업 생성

```bash
make env-backup
```

설정 흐름 실행 없이 수동 백업을 원할 때 사용합니다.

## 출력 및 의미

### `.env`

마법사는 저장소 루트에 `.env`를 작성합니다. 이 파일은 최근 마법사 실행에서 생성된 현재 런타임 설정이 됩니다.

- 마법사를 다시 실행하면 `.env`가 업데이트됩니다
- 기존 값은 이후 실행에서 기본값으로 재사용됩니다
- `.env`를 가장 최근에 설정한 워크플로우의 활성 설정으로 취급하세요
- `env-base`, `env-storage`, `env-server`가 `.env`를 작성하기 전에 마법사는 기존 파일이 있으면 타임스탬프가 있는 백업을 자동으로 생성합니다

### `docker-compose.final.yml`

마법사는 마법사 관리 Docker 서비스를 선택하거나 기존 마법사 생성 컴포즈 설정이 새 서버 설정과 일치하도록 유지해야 할 때만 `docker-compose.final.yml`을 생성하거나 업데이트합니다.

생성된 Docker 스택 시작 시:

```bash
docker compose -f docker-compose.final.yml up -d
```

기본 `docker-compose.yml`은 일반 프로젝트 컴포즈 파일로 유지됩니다.

## 문제 해결 및 고급 참고 사항

- `make env-storage` 또는 `make env-server`가 `.env`가 없다고 하면 먼저 `make env-base`를 실행하세요.
- `env-base`, `env-storage`, `env-server`를 다시 실행하기 전에 `make env-backup`을 실행할 필요가 없습니다. 이 흐름들은 이미 기존 `.env`를 백업하고 변경 전에 생성된 컴포즈 파일도 백업합니다.
- 마법사 관리 컴포즈 서비스를 현재 번들 템플릿에서 완전히 재빌드해야 하면 `make env-base-rewrite` 또는 `make env-storage-rewrite`를 사용하세요.
- 호스트 지향과 Docker 지향 워크플로우 간에 전환하는 경우 수동으로 이전 설정을 병합하려 하지 말고 관련 설정 단계를 다시 실행하세요.
- 생성된 스택에 로컬 Milvus가 포함된 경우 `docker compose -f docker-compose.final.yml up -d`를 실행하기 전에 `MINIO_ACCESS_KEY_ID`와 `MINIO_SECRET_ACCESS_KEY`를 사용 가능하게 해야 합니다.

## 일반적인 명령 순서

### 원격 모델, 로컬 서버

```bash
make env-base
lightrag-server
```

### 원격 LLM, Docker에서 로컬 임베딩 및 리랭크

```bash
make env-base
docker compose -f docker-compose.final.yml up -d
```

### 기본 설정 후 스토리지 추가

```bash
make env-base
make env-storage
docker compose -f docker-compose.final.yml up -d
```

### 노출 전 보안 및 SSL 추가

```bash
make env-base
make env-storage
make env-server
make env-security-check
docker compose -f docker-compose.final.yml up -d
```

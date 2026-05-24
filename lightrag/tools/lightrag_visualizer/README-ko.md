# LightRAG 3D 그래프 뷰어

LightRAG 패키지에 포함된 대화형 3D 그래프 시각화 도구로, RAG(검색 증강 생성) 그래프 및 기타 그래프 구조를 시각화하고 분석합니다.

![image](https://github.com/user-attachments/assets/b0d86184-99fc-468c-96ed-c611f14292bf)

## 설치

### 빠른 설치
```bash
pip install lightrag-hku[tools]  # 시각화 도구만 설치
# 또는
pip install lightrag-hku[api,tools]  # API와 시각화 도구 함께 설치
```

## 뷰어 실행
```bash
lightrag-viewer
```

## 기능

- **3D 대화형 시각화**: ModernGL을 사용한 고성능 3D 그래픽 렌더링
- **다양한 레이아웃 알고리즘**: 여러 그래프 레이아웃 지원
  - 스프링 레이아웃
  - 원형 레이아웃
  - 쉘 레이아웃
  - 랜덤 레이아웃
- **커뮤니티 감지**: 그래프 커뮤니티 구조의 자동 감지 및 시각화
- **대화형 컨트롤**:
  - WASD + QE 키로 카메라 이동
  - 마우스 우클릭 드래그로 시야각 제어
  - 노드 선택 및 하이라이트
  - 조절 가능한 노드 크기 및 엣지 너비
  - 구성 가능한 레이블 표시
  - 노드 연결 간 빠른 탐색

## 기술 스택

- **imgui_bundle**: 사용자 인터페이스
- **ModernGL**: OpenGL 그래픽 렌더링
- **NetworkX**: 그래프 데이터 구조 및 알고리즘
- **NumPy**: 수치 계산
- **community**: 커뮤니티 감지

## 대화형 컨트롤

### 카메라 이동
- W: 앞으로 이동
- S: 뒤로 이동
- A: 왼쪽으로 이동
- D: 오른쪽으로 이동
- Q: 위로 이동
- E: 아래로 이동

### 시야 제어
- 마우스 오른쪽 버튼을 누른 채 드래그하여 시야 회전

### 노드 상호작용
- 마우스를 올리면 노드 하이라이트
- 클릭하여 노드 선택

## 시각화 설정

UI 제어판을 통해 조절 가능:
- 레이아웃 타입
- 노드 크기
- 엣지 너비
- 레이블 표시 여부
- 레이블 크기
- 배경 색상

## 커스터마이징 옵션

- **노드 스케일링**: `node_scale` 매개변수를 통해 노드 크기 조절
- **엣지 너비**: `edge_width` 매개변수를 사용하여 엣지 너비 수정
- **레이블 표시**: `show_labels`로 레이블 표시 여부 전환
- **레이블 크기**: `label_size`를 사용하여 레이블 크기 조절
- **레이블 색상**: `label_color`를 통해 레이블 색상 설정
- **시야 거리**: `label_culling_distance`로 최대 레이블 표시 거리 제어

## 시스템 요구사항

- Python 3.9+
- OpenGL 3.3+ 지원 그래픽 카드
- 지원 운영 체제: Windows/Linux/macOS

## 문제 해결

### 일반적인 문제

1. **명령을 찾을 수 없음**
   ```bash
   # 'tools' 옵션으로 설치했는지 확인
   pip install lightrag-hku[tools]

   # 설치 확인
   pip list | grep lightrag-hku
   ```

2. **ModernGL 초기화 실패**
   ```bash
   # OpenGL 버전 확인
   glxinfo | grep "OpenGL version"

   # 필요한 경우 그래픽 드라이버 업데이트
   ```

3. **폰트 로딩 문제**
   - 필요한 폰트는 패키지에 포함되어 있음
   - 문제가 지속되면 그래픽 드라이버 확인

## LightRAG와 함께 사용

이 뷰어는 다음과 같은 경우에 특히 유용합니다:
- RAG 지식 그래프 시각화
- 문서 관계 분석
- 의미적 연결 탐색
- 검색 패턴 디버깅

## 성능 최적화

- ModernGL을 사용한 효율적인 그래픽 렌더링
- 레이블 표시 최적화를 위한 시야 거리 컬링
- 대규모 그래프 시각화 최적화를 위한 커뮤니티 감지 알고리즘

## 지원

- GitHub Issues: [LightRAG 저장소](https://github.com/HKUDS/LightRAG)
- 문서: [LightRAG 문서](https://URL-to-docs)

## 라이선스

이 도구는 LightRAG의 일부로 MIT 라이선스 하에 배포됩니다. 자세한 내용은 `LICENSE`를 참조하세요.

참고: 이 시각화 도구는 LightRAG 패키지의 선택적 구성 요소입니다. 뷰어 기능에 액세스하려면 [tools] 옵션으로 설치하세요.

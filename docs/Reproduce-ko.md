# 평가 결과 재현

## 데이터셋

LightRAG에서 사용된 데이터셋은 [TommyChien/UltraDomain](https://huggingface.co/datasets/TommyChien/UltraDomain)에서 다운로드할 수 있습니다.

## 쿼리 생성

LightRAG는 다음 프롬프트를 사용하여 고수준 쿼리를 생성합니다. 해당 코드는 `examples/generate_query.py`에 있습니다.

**프롬프트**

```
다음 데이터셋 설명이 주어졌을 때:

{description}

이 데이터셋을 사용할 5명의 잠재적 사용자를 식별하세요. 각 사용자에 대해 데이터셋으로 수행할 5가지 작업을 나열하세요. 그런 다음 각 (사용자, 작업) 조합에 대해 전체 데이터셋에 대한 높은 수준의 이해를 필요로 하는 5가지 질문을 생성하세요.

결과를 다음 구조로 출력하세요:
- 사용자 1: [사용자 설명]
    - 작업 1: [작업 설명]
        - 질문 1:
        - 질문 2:
        - 질문 3:
        - 질문 4:
        - 질문 5:
    - 작업 2: [작업 설명]
        ...
    - 작업 5: [작업 설명]
- 사용자 2: [사용자 설명]
    ...
- 사용자 5: [사용자 설명]
    ...
```

## 배치 평가

고수준 쿼리에서 두 RAG 시스템의 성능을 평가하기 위해 LightRAG는 다음 프롬프트를 사용합니다. 구체적인 코드는 `reproduce/batch_eval.py`에서 확인할 수 있습니다.

**프롬프트**

```
---역할---
당신은 세 가지 기준, 즉 **포괄성(Comprehensiveness)**, **다양성(Diversity)**, **역량강화(Empowerment)**를 기반으로 동일한 질문에 대한 두 답변을 평가하는 전문가입니다.

---목표---
세 가지 기준을 기반으로 동일한 질문에 대한 두 답변을 평가합니다:

- **포괄성**: 답변이 질문의 모든 측면과 세부 사항을 다루는 데 얼마나 상세한가요?
- **다양성**: 답변이 질문에 대해 다양한 관점과 통찰을 얼마나 다양하고 풍부하게 제공하나요?
- **역량강화**: 답변이 독자가 주제를 이해하고 정보에 입각한 판단을 내리는 데 얼마나 도움이 되나요?

각 기준에 대해 더 나은 답변(답변 1 또는 답변 2)을 선택하고 이유를 설명하세요. 그런 다음 이 세 가지 카테고리를 기반으로 전체 승자를 선택하세요.

질문: {query}

두 답변:

**답변 1:**
{answer1}

**답변 2:**
{answer2}

위에 나열된 세 가지 기준을 사용하여 두 답변을 평가하고 각 기준에 대한 상세한 설명을 제공하세요.

다음 JSON 형식으로 평가를 출력하세요:

{{
    "Comprehensiveness": {{
        "Winner": "[답변 1 또는 답변 2]",
        "Explanation": "[여기에 설명 제공]"
    }},
    "Empowerment": {{
        "Winner": "[답변 1 또는 답변 2]",
        "Explanation": "[여기에 설명 제공]"
    }},
    "Overall Winner": {{
        "Winner": "[답변 1 또는 답변 2]",
        "Explanation": "[세 가지 기준을 기반으로 이 답변이 전체 승자인 이유를 요약하세요]"
    }}
}}
```

## 전체 성능 표

||**농업**||**CS**||**법률**||**혼합**||
|----------------------|---------------|------------|------|------------|---------|------------|-------|------------|
||NaiveRAG|**LightRAG**|NaiveRAG|**LightRAG**|NaiveRAG|**LightRAG**|NaiveRAG|**LightRAG**|
|**포괄성**|32.4%|**67.6%**|38.4%|**61.6%**|16.4%|**83.6%**|38.8%|**61.2%**|
|**다양성**|23.6%|**76.4%**|38.0%|**62.0%**|13.6%|**86.4%**|32.4%|**67.6%**|
|**역량강화**|32.4%|**67.6%**|38.8%|**61.2%**|16.4%|**83.6%**|42.8%|**57.2%**|
|**전체**|32.4%|**67.6%**|38.8%|**61.2%**|15.2%|**84.8%**|40.0%|**60.0%**|
||RQ-RAG|**LightRAG**|RQ-RAG|**LightRAG**|RQ-RAG|**LightRAG**|RQ-RAG|**LightRAG**|
|**포괄성**|31.6%|**68.4%**|38.8%|**61.2%**|15.2%|**84.8%**|39.2%|**60.8%**|
|**다양성**|29.2%|**70.8%**|39.2%|**60.8%**|11.6%|**88.4%**|30.8%|**69.2%**|
|**역량강화**|31.6%|**68.4%**|36.4%|**63.6%**|15.2%|**84.8%**|42.4%|**57.6%**|
|**전체**|32.4%|**67.6%**|38.0%|**62.0%**|14.4%|**85.6%**|40.0%|**60.0%**|
||HyDE|**LightRAG**|HyDE|**LightRAG**|HyDE|**LightRAG**|HyDE|**LightRAG**|
|**포괄성**|26.0%|**74.0%**|41.6%|**58.4%**|26.8%|**73.2%**|40.4%|**59.6%**|
|**다양성**|24.0%|**76.0%**|38.8%|**61.2%**|20.0%|**80.0%**|32.4%|**67.6%**|
|**역량강화**|25.2%|**74.8%**|40.8%|**59.2%**|26.0%|**74.0%**|46.0%|**54.0%**|
|**전체**|24.8%|**75.2%**|41.6%|**58.4%**|26.4%|**73.6%**|42.4%|**57.6%**|
||GraphRAG|**LightRAG**|GraphRAG|**LightRAG**|GraphRAG|**LightRAG**|GraphRAG|**LightRAG**|
|**포괄성**|45.6%|**54.4%**|48.4%|**51.6%**|48.4%|**51.6%**|**50.4%**|49.6%|
|**다양성**|22.8%|**77.2%**|40.8%|**59.2%**|26.4%|**73.6%**|36.0%|**64.0%**|
|**역량강화**|41.2%|**58.8%**|45.2%|**54.8%**|43.6%|**56.4%**|**50.8%**|49.2%|
|**전체**|45.2%|**54.8%**|48.0%|**52.0%**|47.2%|**52.8%**|**50.4%**|49.6%|

## 재현

모든 코드는 `./reproduce` 디렉토리에 있습니다.

### 0단계: 고유 컨텍스트 추출

먼저 데이터셋에서 고유 컨텍스트를 추출합니다.

**코드**

```python
def extract_unique_contexts(input_directory, output_directory):

    os.makedirs(output_directory, exist_ok=True)

    jsonl_files = glob.glob(os.path.join(input_directory, '*.jsonl'))
    print(f"JSONL 파일 {len(jsonl_files)}개를 찾았습니다.")

    for file_path in jsonl_files:
        filename = os.path.basename(file_path)
        name, ext = os.path.splitext(filename)
        output_filename = f"{name}_unique_contexts.json"
        output_path = os.path.join(output_directory, output_filename)

        unique_contexts_dict = {}

        print(f"파일 처리 중: {filename}")

        try:
            with open(file_path, 'r', encoding='utf-8') as infile:
                for line_number, line in enumerate(infile, start=1):
                    line = line.strip()
                    if not line:
                        continue
                    try:
                        json_obj = json.loads(line)
                        context = json_obj.get('context')
                        if context and context not in unique_contexts_dict:
                            unique_contexts_dict[context] = None
                    except json.JSONDecodeError as e:
                        print(f"파일 {filename}의 {line_number}번째 줄에서 JSON 디코딩 오류: {e}")
        except FileNotFoundError:
            print(f"파일을 찾을 수 없음: {filename}")
            continue

        unique_contexts_list = list(unique_contexts_dict.keys())
        print(f"파일 {filename}에 고유 `context` 항목이 {len(unique_contexts_list)}개 있습니다.")

        with open(output_path, 'w', encoding='utf-8') as outfile:
            json.dump(unique_contexts_list, outfile, ensure_ascii=False, indent=4)
        print(f"고유 `context` 항목이 {output_filename}에 저장되었습니다.")

    print("모든 파일이 처리되었습니다.")
```

### 1단계: 컨텍스트 삽입

추출된 컨텍스트를 LightRAG 시스템에 삽입합니다.

**코드**

```python
def insert_text(rag, file_path):
    with open(file_path, mode='r') as f:
        unique_contexts = json.load(f)

    retries = 0
    max_retries = 3
    while retries < max_retries:
        try:
            rag.insert(unique_contexts)
            break
        except Exception as e:
            retries += 1
            print(f"삽입 실패, 재시도 중 ({retries}/{max_retries}), 오류: {e}")
            time.sleep(10)
    if retries == max_retries:
        print("최대 재시도 횟수 초과 후 삽입 실패")
```

### 2단계: 쿼리 생성

각 컨텍스트의 앞뒤 절반에서 토큰을 추출하여 데이터셋 설명으로 결합하고 쿼리를 생성합니다.

**코드**

```python
tokenizer = GPT2Tokenizer.from_pretrained('gpt2')

def get_summary(context, tot_tokens=2000):
    tokens = tokenizer.tokenize(context)
    half_tokens = tot_tokens // 2

    start_tokens = tokens[1000:1000 + half_tokens]
    end_tokens = tokens[-(1000 + half_tokens):1000]

    summary_tokens = start_tokens + end_tokens
    summary = tokenizer.convert_tokens_to_string(summary_tokens)

    return summary
```

### 3단계: 쿼리

2단계에서 생성된 쿼리로 LightRAG를 쿼리합니다.

**코드**

```python
def extract_queries(file_path):
    with open(file_path, 'r') as f:
        data = f.read()

    data = data.replace('**', '')

    queries = re.findall(r'- Question \d+: (.+)', data)

    return queries
```

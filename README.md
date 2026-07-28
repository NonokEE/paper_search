<div align="center">

<h1>🔍 paper_search</h1>

<p>
  자연어 질의 → 키워드 추출 → arxiv 검색 → 관련 논문 선별 → 요약 & 답변<br/>
  <b>LangGraph 기반 논문 검색 에이전트</b>
</p>

<p>
  <img src="https://img.shields.io/badge/Python-3.12-blue?logo=python" />
  <img src="https://img.shields.io/badge/LangGraph-Agent-blueviolet?logo=langchain" />
  <img src="https://img.shields.io/badge/LLM-Gemma4%3A31b-orange" />
  <img src="https://img.shields.io/badge/Source-arXiv-red?logo=arxiv" />
  <img src="https://img.shields.io/badge/Status-WIP🔧-yellow" />
</p>

</div>

---

## 📌 개요

사용자의 **자연어 질의**를 분석해서 검색 키워드를 추출하고,  
[arXiv](https://export.arxiv.org)에서 논문을 검색한 뒤, 가장 적합한 논문을 골라 **요약 및 답변**을 제공하는 LangGraph 기반 멀티노드 에이전트 프로젝트.

> *"LLM이 되면서 실질적으로 개발 효율이 좋아졌어?"* 같은 질문을 던지면, 알아서 논문 찾아서 답해줌

---

## 🏗️ 아키텍처

```
사용자 질의
    │
    ▼
[keyword_extract]  ← LLM으로 영문 키워드 추출
    │
    ▼
[paper_get]        ← export.arxiv.org 에서 논문 수집
    │
    ▼
[paper_select]     ← LLM으로 질의 관련도 높은 논문 선별
    │
    ▼
[paper_download]   ← 선별 논문 PDF 다운로드 (WIP)
    │
    ▼
[summarize]        ← 논문 내용 요약 후 사용자 질의에 답변 (TODO)
    │
    ▼
[feedback]         ← 사용자 승인 / 재요청 분기 처리 (TODO)
```

---

## ✅ 진행 현황

### DONE
- [x] `keyword_extract` 노드 — 자연어 → 영문 키워드 (CommaSeparatedListOutputParser)
- [x] `paper_get` 노드 — arXiv에서 title / authors / abstract / pdf_path 수집
- [x] `paper_select` 노드 — LLM으로 질의 관련 논문 선별 (PydanticOutputParser)
- [x] `arxivState` TypedDict 설계 — `input_query`, `keyword`, `papers`, `select_papers`

### WIP 🔧
- [ ] `paper_download` 노드 — 선별 논문 PDF 다운로드 후 `state`에 path 전달

### TODO 📋
- [ ] `summarize` 노드 — 다운받은 논문 요약 → 사용자 질의에 답변 생성
- [ ] `feedback` 노드 — 사용자 응답 해석 (승인 / 재요청 분기)
  - 재요청 원인 분석: 논문이 잘못됐는지 vs 키워드가 잘못됐는지 케이스 분리
  - `add_message` 로 컨텍스트 유지
- [ ] `conditional_edge` — 피드백 결과에 따른 분기 (RunableBranch or Literal)
- [ ] VectorDB 도입 검토 — PDF 통째로 LLM에 넣는 대신 유사도 검색 활용

---

## 💡 구현 메모

<details>
<summary><b>arXiv 크롤링 관련</b></summary>

- 일반 `arxiv.org` 대신 프로그램 접근 전용 **`export.arxiv.org`** 사용
- API의 Atom 응답 파싱 방식이 유지보수에도 유리함
- 논문을 재배포하는 게 아닌, 로컬 환경에 적재하는 flow이므로 라이센스 문제 없음

> <sub>Thank you to arXiv for use of its open access interoperability.</sub>

</details>

<details>
<summary><b>paper_select 노드 개선 아이디어</b></summary>

- 현재: LLM한테 논문 목록 던져서 관련도 높은 거 골라달라고 함
- 개선안: **VectorDB** 기반 유사도 검색으로 대체하면 더 정확하고 빠를 수 있음
- 일반 DB도 따로 두면 이미 적재한 논문 중복 방지 가능

</details>

---

## 🛠️ 기술 스택

| 항목 | 내용 |
|------|------|
| LLM | Gemma4:31b (via Ollama) |
| Agent Framework | LangGraph |
| 논문 소스 | export.arxiv.org |
| PDF 파싱 | PyPDFLoader (LangChain) |
| 출력 파싱 | PydanticOutputParser, CommaSeparatedListOutputParser |
| 개발 환경 | Python 3.12, Jupyter Notebook |

---

## 🚀 실행 방법

```bash
# 1. 환경 세팅
pip install langchain langgraph langchain-ollama python-dotenv beautifulsoup4 requests

# 2. .env 파일에 OLLAMA_API_KEY 설정
echo "OLLAMA_API_KEY=your_key_here" > .env

# 3. workbench/working.ipynb 실행
```

---

<div align="center">
  <sub><b> Written by NonokEE 🔨</b></sub>
</div>

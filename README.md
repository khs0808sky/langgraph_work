# langgraph_work

## 📅 목차

- [2025-08-29](#2025-08-29)
  
<br><br><br>

---

## 2025-08-29

---

### LangGraph

LangGraph는 \*\*LangChain 생태계에서 제공하는 “상태 기반(stateful) AI 워크플로우 프레임워크”\*\*입니다. LLM 애플리케이션을 설계할 때 단순한 직선형 체인이 아니라 **분기, 반복, 상태 유지**를 포함한 **그래프 기반 제어 흐름**을 지원합니다.

#### 📌 주요 특징

1. **그래프 구조 기반**

   * 기존 LangChain은 `Chain`(순차 실행) 개념이 중심이었는데, LangGraph는 **Node와 Edge**를 가진 **상태 기계(state machine)** 형태로 LLM 워크플로우를 표현합니다.
   * 예: "질문 → 요약 → 검증 → 최종 응답" 같은 흐름을 그래프로 표현.

2. **상태(state) 관리**

   * 대화나 실행 과정에서 생성되는 **컨텍스트·메모리·변수 값**을 지속적으로 관리합니다.
   * 단순히 입력→출력의 one-shot이 아니라, "현재 상태"에 따라 다음 실행 경로가 달라집니다.

3. **조건 분기 & 루프 지원**

   * LLM 응답이나 외부 입력에 따라 다른 노드로 이동할 수 있음.
   * 예: 모델이 "추가 정보 필요"라고 판단하면 → "검색 노드"로 이동.

4. **LangChain과 통합**

   * LangGraph는 LangChain의 **Runnable 인터페이스**를 확장한 형태이므로, 기존 LangChain 객체와 호환됨.
   * 따라서 기존 `LLMChain`, `Retriever`, `Tool` 등을 그대로 Node 안에서 사용 가능.

5. **에이전트(Agent) 구현에 유용**

   * Agent는 본질적으로 **상태를 가진 결정 엔진**인데, LangGraph는 이를 자연스럽게 모델링할 수 있음.
   * ReAct, Plan-and-Execute 같은 패턴을 그래프 구조로 쉽게 표현 가능.

#### 📌 활용 예시

* **대화형 챗봇**: 사용자의 질문에 따라 `검색 → 요약 → 답변` 또는 `직접 답변` 흐름 분기.
* **문서 분석 파이프라인**: "텍스트 추출 → 전처리 → 요약 → 질의응답".
* **멀티스텝 의사결정 Agent**: 목표 달성을 위해 "계획 수립 → 실행 → 검증 → 반복" 구조로 표현.

#### 📌 간단한 코드 예시 (개념용)

```python
from langgraph.graph import StateGraph, END

# 상태 정의
class MyState:
    question: str
    answer: str

# 그래프 생성
graph = StateGraph(MyState)

# 노드 정의
def generate_answer(state: MyState):
    return {"answer": f"답변: {state.question}"}

graph.add_node("answer_node", generate_answer)
graph.set_entry_point("answer_node")
graph.add_edge("answer_node", END)

# 실행
app = graph.compile()
result = app.invoke({"question": "LangGraph란 무엇인가요?"})
print(result)
# {'answer': '답변: LangGraph란 무엇인가요?'}
```

---

📅[목차로 돌아가기](#-목차)

---

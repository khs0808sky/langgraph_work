# langgraph_work

## 📅 목차

- [2025-08-29](#2025-08-29)
- [2025-09-01](#2025-09-01)
  
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

## 2025-09-01

### LangGraph: 상태(State), 노드(Node), 엣지(Edge)

LangGraph는 **LLM 애플리케이션을 그래프 형태로 구성**하도록 돕는 프레임워크예요.
핵심 개념은 **상태(State)**, **노드(Node)**, \*\*엣지(Edge)\*\*이며, 이 3가지를 이해하면 거의 전체 구조를 이해한 거나 마찬가지예요.

---

#### 🔹 1. 상태 (State)

* **현재 그래프 실행 맥락을 나타내는 데이터 저장소.**
* 워크플로우의 **입력과 출력**, 중간 단계 결과까지 전부 `State`에 저장됩니다.
* 상태는 **딕셔너리(dictionary)** 형태로 관리됨:

  ```python
  class MyState(TypedDict):
      question: str
      answer: str
  ```
* 그래프 실행 중 각 노드가 이 상태를 읽고, 필요한 값을 추가/수정.
* **장점**: 분기나 반복이 생겨도 상태가 유지되므로 복잡한 흐름도 안정적으로 관리.

---

#### 🔹 2. 노드 (Node)

* **실제 로직이 실행되는 단위.**
* 각 노드는 함수를 감싼 하나의 작업 단위이며, `State`를 입력받고 다시 `State`를 반환.
* 노드 안에서는 LLM 호출, 외부 API 호출, 데이터 전처리 등을 수행 가능.
* 코드 예:

  ```python
  def answer_node(state: MyState):
      return {"answer": f"질문 '{state['question']}'에 대한 답변입니다."}
  ```

---

#### 🔹 3. 엣지 (Edge)

* **노드 간 연결 경로.**
* 그래프에서 노드 실행 순서를 정의.
* 엣지는 **조건부** 또는 **단순 연결**로 구성 가능.

  * 단순 연결: `A → B`
  * 조건부 연결: `A → B 또는 C` (상태나 함수 결과에 따라 분기)
* 조건부 예시:

  ```python
  def route(state: MyState):
      if "날씨" in state["question"]:
          return "weather_node"
      return "default_node"

  graph.add_conditional_edges("router_node", route)
  ```

---

#### 🔹 전체 흐름 요약

1. **State**: 현재까지의 맥락과 데이터를 저장.
2. **Node**: 실제 처리(LLM 호출, 계산 등)를 수행하는 단계.
3. **Edge**: 어떤 노드를 다음으로 실행할지 결정하는 경로.

이 구조를 통해 LangGraph는 단순 체인보다 **복잡한 에이전트·파이프라인** 설계를 훨씬 쉽게 해줍니다.

---

📅[목차로 돌아가기](#-목차)

---


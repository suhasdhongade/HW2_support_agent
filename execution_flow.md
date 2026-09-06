# Support Agent Execution Flow

This diagram shows how one customer question moves through the current program.

## Whole Execution Flow

```mermaid
flowchart TD
    A[Customer query] --> B[SupportAgent.resolve]
    B --> C[Create SupportGraph]
    C --> D[Build initial state]
    D --> E[LangGraph: lookup]

    E --> E1[Read current query and history]
    E1 --> E2{Order number found?}
    E2 -- Yes --> E3[get_order for each order]
    E2 -- No --> F[LangGraph: retrieve]
    E3 --> F

    F --> F1[Search handbook]
    F1 --> F2[Store retrieved hits]
    F2 --> G[LangGraph: act]

    G --> H[Preflight policy checks]
    H --> H1{Missing order or customer info?}
    H1 -- Yes --> I[Return needs_info answer]
    H1 -- No --> H2{Prompt injection detected?}
    H2 -- Yes --> J[Log P2 human escalation]
    H2 -- No --> H3{Delayed order with order number?}
    H3 -- Yes --> K[Issue 500 INR wallet credit]
    H3 -- No --> H4{Refund needs approval?}
    H4 -- Yes --> J
    H4 -- No --> H5{Repeat failure?}
    H5 -- Yes --> L[Get ticket history]
    L --> M[Log P2 human escalation]
    H5 -- No --> H6{Safety or other danger?}
    H6 -- Yes --> N[Stop-use advice and human escalation]
    H6 -- No --> O[Ask model which tool to call]

    O --> P{Tool call requested?}
    P -- Yes --> Q[Run requested tool]
    Q --> R[Return tool result to model]
    R --> S{Tool-step limit reached?}
    S -- No --> O
    S -- Yes --> T[Return safe limit message]
    P -- No --> U[Model has draft answer]

    I --> V[LangGraph: respond]
    J --> V
    K --> V
    M --> V
    N --> V
    T --> V
    U --> V

    V --> W{Preflight result already exists?}
    W -- Yes --> X[Reuse answer and route]
    W -- No --> Y[Build prompt with context and order facts]
    Y --> Z[Ask model for structured answer]
    Z --> AA[Clean citations and route]
    X --> AB[LangGraph ends]
    AA --> AB

    AB --> AC[Build escalation packet]
    AC --> AD[Build trace record]
    AD --> AE[Return answer, route, citations, actions, and metadata]

    B -. Runtime exception .-> AF[Return escalated error trace]
```

## What Each Main Step Means

### 1. `SupportAgent.resolve`

This is the public entry point used by the evaluator, batch runner, and web app. It creates a graph and starts one customer request.

### 2. `lookup`

The lookup node searches the current question and previous conversation history for order IDs such as `MRD-700121`. If it finds one, it calls `get_order()` and stores the order facts in the graph messages.

### 3. `retrieve`

The retrieval node searches the handbook and stores the matching sections. These sections are later placed inside an untrusted-data block before being sent to the model.

### 4. `act`

The act node has two paths:

- **Deterministic preflight path:** Python handles missing information, injections, delayed-order credits, unsafe refunds, safety incidents, and repeat failures.
- **Model tool-loop path:** the model requests a tool, Python runs it, the result goes back to the model, and the loop repeats until the model stops requesting tools or reaches the step limit.

The important safety idea is that the model can suggest a tool, but Python policy code controls whether risky actions are allowed.

### 5. `respond`

If the act node already produced a deterministic answer, the response node reuses it. This prevents actions from being run twice.

Otherwise, the response node sends the retrieved context, order facts, conversation history, and customer question to the model. The model returns:

```json
{
  "answer": "...",
  "citations": ["returns"],
  "route": "resolved"
}
```

The program then removes invalid citation names and keeps only allowed routes.

### 6. Trace creation

The final trace records:

- the answer
- the route: `resolved`, `needs_info`, or `escalated`
- citations
- executed and blocked actions
- escalation information
- timing and model metadata

The evaluator scores this trace.

## Simple Version

```text
Question
  |
  v
Find order facts
  |
  v
Search handbook
  |
  v
Run Python safety checks
  |
  +--> Need information? Ask customer
  |
  +--> Unsafe or approval needed? Escalate
  |
  +--> Delayed order? Give allowed 500 INR credit
  |
  +--> Safe request? Let model use tools in a bounded loop
  |
  v
Generate or reuse final answer
  |
  v
Clean citations
  |
  v
Create trace and return result
```

## Important Current Limitation

The current graph still has fixed LangGraph edges:

```text
lookup -> retrieve -> act -> respond -> end
```

The branches shown inside `act` are Python decisions inside the node. Full LangGraph conditional edges, checkpoint memory, human interrupts, and `SupportAgent.resume()` are still future work for TODO 6.

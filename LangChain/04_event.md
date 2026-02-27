<div align="center">

# ✨ 事件監聽（Stream Event）

</div>

> [!WARNING]
> 為了讓 `astream_events` API 正常工作，請注意以下幾點：
> 
> - 盡可能在整個代碼中使用 `async`（例如，異步工具等）
> - 在定義自定義函數/ runnalbes 時傳播 callbacks
> - 當不使用 LCEL 而使用可運行對象時，確保調用 LLM 的 `.astream()` 而不是 `.ainvoke`，以強制 LLM 串流 token

## 00 Stream_Events 事件參考指南

| event                | name             | chunk                           | input                                         | output                                          |
|----------------------|------------------|---------------------------------|-----------------------------------------------|-------------------------------------------------|
| on_chat_model_start  | [model name]     |                                 | {"messages": [[SystemMessage, HumanMessage]]} |                                                 |
| on_chat_model_stream | [model name]     | AIMessageChunk(content="hello") |                                               |                                                 |
| on_chat_model_end    | [model name]     |                                 | {"messages": [[SystemMessage, HumanMessage]]} | AIMessageChunk(content="hello world")           |
| on_llm_start         | [model name]     |                                 | {'input': 'hello'}                            |                                                 |
| on_llm_stream        | [model name]     | 'Hello'                         |                                               |                                                 |
| on_llm_end           | [model name]     |                                 | 'Hello human!'                                |                                                 |
| on_chain_start       | format_docs      |                                 |                                               |                                                 |
| on_chain_stream      | format_docs      | "hello world!, goodbye world!"  |                                               |                                                 |
| on_chain_end         | format_docs      |                                 | [Document(...)]                               | "hello world!, goodbye world!"                  |
| on_tool_start        | some_tool        |                                 | {"x": 1, "y": "2"}                            |                                                 |
| on_tool_end          | some_tool        |                                 |                                               | {"x": 1, "y": "2"}                              |
| on_retriever_start   | [retriever name] |                                 | {"query": "hello"}                            |                                                 |
| on_retriever_end     | [retriever name] |                                 | {"query": "hello"}                            | [Document(...), ..]                             |
| on_prompt_start      | [template_name]  |                                 | {"question": "hello"}                         |                                                 |
| on_prompt_end        | [template_name]  |                                 | {"question": "hello"}                         | ChatPromptValue(messages: [SystemMessage, ...]) |

> [!WARNING]
> 當串流正確實現時，可運行對象的輸入直到輸入流完全消耗後才能知道。這意味著輸入通常只包含在結束事件中，而不是開始事件中。

## 01 函式庫

```
import langchain_core
import rich
```

### 版本確認

[版本提醒] 確保您使用的是最新版本的 LangChain：

```python
import langchain_core

langchain_core.__version__
```

## 02 收集所有事件

- astream_events() 是 非同步串流事件監聽，它會一個一個回傳「事件」
- 把每個 event 存進 events 陣列

```python
events = []

async for event in model.astream_events("早安", version="v2"):
    events.append(event)
```

> [!TIP]
> 這不是單純回傳文字
> 
> 而是回傳像這樣的結構：
>
> ```
> {
>     "event": "on_chat_model_stream",
> 
>     "data": {...}
> }
> ```

## 03 觀察事件

```
rich.print(events[:3]) # 印出前 3 個事件
rich.print(events[-2:]) # 印出最後 2 個事件
```

## 04 看有哪些事件類型（event types）

```python
event_types = {event["event"] for event in events}
print("抓出存在的 event types:", event_types)
```

> [!TIP]
> 可以注意到回傳 Message 當中不同 Chunk 所在的 event 是不同的，從一開始 on_chat_model_start, on_chat_model_stream, 到最後 on_chat_model_end

## 05 Streaming 事件監控與輸出

> 只監聽模型的部分 `model.astream_events()`

> 這次沒有存起來，而是即時處理

```python
async for event in model.astream_events("早安", version="v2"):
    event_type = event["event"]

    # 模型開始時觸發
    if event_type =="on_chat_model_start":
        print("Streamgin Start", flush=False)
    
    # 🔥 模型每吐出一小段 token 時觸發，會一段一段輸出
    if event_type == "on_chat_model_stream":
        content = event["data"]["chunk"].content
        if content:
          # 要考慮一下回傳沒有的情況
          print(content, end="|", flush=False)
    
    # 模型完整回應完成時觸發
    if event_type == "on_chat_model_end":
        print("\nStreamgin End", flush=False)
```

> [!TIP]
> **流程圖：**
> ```
> 呼叫模型
>     ↓
> on_chat_model_start
>     ↓
> on_chat_model_stream (很多次)
>     ↓
> on_chat_model_end
> ```

## 06 監控整個 LCEL Chain（Prompt → Model → Parser） 事件流

> `chain.astream_events`

### 6.1 第一段：建立 Chain

```
prompt = ChatPromptTemplate.from_template("講一個早安問候語關於 {topic}")
parser = StrOutputParser()
chain = prompt | model | parser
```

### 6.2 第二段：收集所有事件

```
parser_events = []
async for event in chain.astream_events({"topic": "父母親"}, version="v2"):
    parser_events.append(event)

# 印出前六筆事件
rich.print(parser_events[:6])
```

這裡跟前面一樣，使用 astream_events但這次監聽的是整條 chain

所以事件會包含：

```
on_prompt_start
on_prompt_end
on_chat_model_start
on_chat_model_stream
on_parser_stream
on_chat_model_end
on_parser_end
```

這種層次化的事件結構反映了整個處理流程的組織方式，讓我們能夠精確地追蹤每個組件的運作狀態。

### 6.3 即時監聽模型 & parser

```python
num_events = 0

async for event in chain.astream_events(
    {"topic": "父母親"},
    version="v2",
):
    kind = event["event"]
    if kind == "on_chat_model_stream":
        print(
            f"模型輸出 chunk: {repr(event['data']['chunk'].content)}",
            flush=True,
        )
    if kind == "on_parser_stream":
        print(f"解析器輸出 chunk: {event['data']['chunk']}", flush=True)
    num_events += 1
    if num_events > 30:
        # Truncate the output
        print("...")
        break
```

這是 LangChain / LangGraph 的核心概念：

- 你可以監聽任意節點
- 可以插入 log
- 可以做 UI
- 可以做 token 追蹤
- 可以做 tracing
- 可以做 observability

這其實就是 **LangSmith** 背後原理。

## 07 事件過濾

> 可以按組件 `name`、組件 `tags` 或組件 `type` 進行過濾。

### 7.1 按類型(type)過濾

```python
max_events = 0
async for event in chain.astream_events(
    {"topic": "父母親"},
    version="v2",
    include_types=["chat_model"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # Truncate output
        print("...")
        break
```

`include_types=["chat_model"]` 代表：只保留「屬於 chat_model 這個 runnable 類型」的事件

所以通常你會看到的事件大概是：

```
on_chat_model_start
on_chat_model_stream
on_chat_model_end
```

你不會看到：
- prompt 的事件（例如 on_prompt_start/end）
- parser 的事件（例如 on_parser_*）

### 7.2 一次擷取多個類型

```python
include_types=["chat_model", "prompt"]
```

### 7.3 綜合過濾：按類型(type)、名稱(name)和標籤(tags)

> [!TIP]
> run_name 是什麼？你可以把某個 runnable「取名字」，方便在 events 裡辨識它是誰。
> 注意：run_name 只是你給的標籤名字，不代表真的用 GPT-3.5（你只是改名字而已）。

```python
prompt = ChatPromptTemplate.from_template("講一個早安問候語關於 {topic}") # run_name 是什麼？你可以把某個 runnable「取名字」，方便在 events 裡辨識它是誰。
model = model.with_config({"run_name": "OAI_MODEL_GPT3.5"})
parser = StrOutputParser().with_config({"run_name": "my_str_parser"})

chain = prompt | model | parser

max_events = 0
async for event in chain.astream_events(
    {"topic": "父母親"},
    version="v2",
    include_names=["OAI_MODEL_GPT3.5"],
    include_types=["chat_model", "prompt"],
    include_tags=["my_str_parser"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # Truncate output
        print("...")
        break
```

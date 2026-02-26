<div align="center">

# ✨ 四、圖(Graph): LangGraph 的核心結構

</div>

## 01 函式庫

```
from langgraph.graph import StateGraph
```

## 02 設定「圖」

> LangGraph 的核心是將代理工作流程建模為圖，你可以制定你的 AI 代理行為，使用上面介紹的關鍵元素：

LangGraph 還有個特別的 `MessageGraph` 類別。這種圖的狀態就只是一串訊息列表。雖然主要用在聊天機器人上，但大多數應用可能需要更複雜的狀態結構。

```
graph_builder = StateGraph(AllState)
```

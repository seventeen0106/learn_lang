<div align="center">

# ✨ 四、邊(Edge)：連接節點的橋樑

</div>

> 邊就像是工作流程中的傳送帶或決策點。它們定義了信息如何在節點之間流動，以及在什麼條件下流動。

想像一下一個複雜的物流系統：邊決定了包裹（數據或狀態）應該送到哪個下一站。有時候這是一個簡單的直線運輸，有時候則需要根據包裹的特性做出決策。

## 01 函式庫

```python
from langgraph.graph import START
from langgraph.graph import END
```

## 02 三種邊的類型

通過 `add_node` 方法，每個節點都有一個唯一的名稱（如 "node_1"），這樣我們就可以在後續的工作流程中明確地引用它們。

### 2.1 普通邊(Simple Edge)

> 最基本的邊，直接連接兩個節點，使用 `add_edge` 方法來添加。

```python
# 這行程式碼告訴系統，'node_1' 完成處理後，訊息應該直接傳送到 'node_2'。
graph.add_edge('node_1', 'node_2')
```

### 2.2 入口點(Entry Point)和終點(End Point)

> 入口點是圖開始執行時運行的第一個節點。我們使用 `START` 節點來指定圖的入口。

```python
# 工作從這裡開始，到那裡結束
graph.add_edge(START, "node_a")
graph.add_edge("node_2", END)
```

### 2.3 條件邊(Conditional Edge)

> 允許我們根據特定條件選擇下一個執行的節點，使用 `add_conditional_edges` 添加

```python
def where_to_go(state):
  # 條件設定
  if state['Condition']:
    return "end"
  else:
    return "continue"

# 代理的節點，透過條件設定，連接到後續 2 個節點
graph.add_conditional_edges('agent',where_to_go,{
    "end": END,
    "continue": "weather_tool"
})
```

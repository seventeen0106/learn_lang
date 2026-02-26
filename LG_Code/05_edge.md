<div align="center">

# ✨ 五、邊(Edge)：連接節點的橋樑

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

## 03 加入節點的範例

```python
# 1. 註冊所有的節點 (Nodes)
# 把我們前面寫好的三個功能都加入到圖中
graph_builder.add_node("agent", call_model)      # 負責分析問題、萃取城市名稱
graph_builder.add_node("weather", weather_tool)  # 負責外部查詢 (假的天氣 API)
graph_builder.add_node("responder", responder)   # 負責將資料融合，寫出通順的最終回答

# 2. 設定起點 (Entry Point)
# 規定每次執行時，第一站一定要先經過 agent 節點
graph_builder.set_entry_point("agent")

# 3. 建立「條件邊」(Conditional Edge) 🌟 這是靈魂所在！
# 當資料離開 'agent' 節點後，會呼叫 `query_classify` 這個判斷函式
graph_builder.add_conditional_edges(
    'agent',          # 出發的節點
    query_classify,   # 負責做決定的函式 (它會去檢查目前的 state，然後回傳特定字串)
    {
        # 這裡是一個字典，負責將 query_classify 的回傳值對應到下一個節點：
        # 如果判斷函式回傳 "end" (代表這題不用查天氣，例如使用者只是說「你好」) -> 直接送到 responder
        "end": "responder", 
        
        # 如果判斷函式回傳 "continue" (代表這題有提到城市，需要查天氣) -> 送到 weather 節點
        "continue": "weather"
    }
)

# 4. 建立「一般邊」(Normal Edges)
# 規定天氣查完之後，下一步一定要交給 responder 來包裝回覆
graph_builder.add_edge("weather", "responder")

# 規定 responder 包裝完回覆後，走到 END，結束這次的對話流程
graph_builder.add_edge("responder", END)

# 5. 編譯 (Compile)
# 將這張設計好的藍圖，編譯成可以實際接收輸入並運作的應用程式
app = graph_builder.compile()
```

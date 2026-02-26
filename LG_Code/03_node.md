<div align="center">

# ✨ 三、節點(Node)：工作流程的基石

</div>

> 每個節點都是一個工作站，負責接收原材料（輸入），進行加工（處理），然後產出成品（輸出）。
>
> 這些「工作站」可以執行各種操作，從簡單的資料轉換到複雜的下達決策。

## 01 函式庫

```python
# 引入 LangChain 中用來串接 OpenAI 格式語言模型的套件
from langchain_openai import ChatOpenAI
# 引入用來建立提示詞範本（Prompt Template）的套件
from langchain_core.prompts import ChatPromptTemplate
```

## 02 設定函數（等等轉化成節點）

### 1. 設定 AI 模型的函數

```python
### 設定 AI 模型
## 1. 初始化語言模型 (LLM)
model = ChatOpenAI(
    model="gpt-4.1-nano",
    openai_api_base="https://api.apertis.ai/v1", 
)

## 2. 定義提示詞字串範本 (Prompt)
prompt_str = """
You are given one question and you have to extract city name from it
Don't respond anything except the city name and don't reply anything if you can't find city name
Only reply the city name if it exists or reply 'no_response' if there is no city name in question

  Here is the question:
  {user_query}
"""

# 將剛剛定義的字串轉換成 LangChain 的 PromptTemplate 物件
prompt = ChatPromptTemplate.from_template(prompt_str)

## 3. 建立執行鏈 (Chain)
# 使用 LangChain 表達式語言 (LCEL)，將 prompt 與 model 串接起來
# 意思是：資料會先進入 prompt 填寫，然後將填寫完的提示詞送給 model 產生結果
chain = prompt | model

## 4. 執行鏈並輸入資料
# 呼叫 invoke 方法，並將字典傳入，把 "請問高雄天氣如何?" 塞入剛剛的 {user_query} 變數中
result = chain.invoke({"user_query": "請問高雄天氣如何?"})

## 5. 印出結果
# result 是一個 AIMessage 物件，.content 可以取得純文字的回答內容
print(result.content)
```

### 2. 回傳天氣狀況（僅模擬外部 API）

```python
### 查詢台灣特定城市的天氣狀況 (模擬外部天氣 API)
def get_taiwan_weather(city: str) -> str:
    # 簡單的字典，用來模擬資料庫查詢
    weather_data = {
        "台北": "晴天，溫度28°C",
        "台中": "多雲，溫度26°C",
        "高雄": "陰天，溫度30°C"
    }
    # 找不到該城市時回傳 '暫無資料'
    return f"{city}的天氣：{weather_data.get(city, '暫無資料')}"
```

### 3. 設定條件篩選後得到的回應節點

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

model = ChatOpenAI(
    model="gpt-4.1-nano",
    openai_api_base="https://api.apertis.ai/v1", 
)

response_prompt_str = """
  You have given a weather information and you have to respond to user's query based on the information

  Here is the user query:
  ---
  {user_query}
  ---

  Here is the information:
  ---
  {information}
  ---
  """
response_prompt = ChatPromptTemplate.from_template(response_prompt_str)

response_chain = response_prompt | model
res = response_chain.invoke({
    "user_query": "請問杜拜天氣如何?",
    "information": "no_response的天氣：暫無資料"
    })
res.content
```

## 03 轉化函數為節點

```python
## 節點 A：呼叫 LLM 萃取城市名稱
def call_model(state: AllState):
    # 1. 從 state 拿出目前的訊息列表
    messages = state["messages"]
    
    # 2. 取出最後一條訊息（也就是使用者剛問的問題），放進變數 {user_query} 讓 chain 執行
    user_input = messages[-1]
    response = chain.invoke({"user_query": user_input})
    
    # 3. 回傳字典。因為我們用了 operator.add，這個 response 會被加到 messages 列表的最後面
    return {"messages": [response]}

## 節點 B：呼叫天氣工具
def weather_tool(state: AllState):
    # 1. 從 state 拿出目前的訊息列表
    context = state["messages"]
    
    # 2. 經過節點 A 處理後，最後一筆訊息 (-1) 就是 AI 萃取出來的「城市名稱」(例如：高雄)
    city_name = context[-1].content.strip() # .content 是取出文字內容，.strip() 是為了確保把前後不小心多出的空白去掉
    
    # 3. 把乾淨的城市名稱丟給天氣工具函式
    data = get_taiwan_weather(city_name)
    
    # 4. 回傳天氣結果
    return {"messages": [data]}

## 節點 C：負責產生最終人性化回覆 (Responder Node)
def responder(state: AllState):
    # 1. 從當前狀態 (state) 中取出整疊「訊息歷史紀錄」
    messages = state["messages"]
    
    # 2. 呼叫另一個設定好的執行鏈 (response_chain) 來生成最終回答
    # 這裡預期 response_chain 裡面有一個 Prompt，需要填入兩個變數：
    response = response_chain.invoke({
        # messages[0]: 取出列表的「第一筆」資料，也就是使用者最初提問的問題
        "user_query": messages[0], 
        
        # messages[1]: 取出列表的「第二筆」資料，前面工具節點查出來的結果
        "information": messages[1] 
    })
    
    # 3. 將 AI 潤飾並生成好的最終回覆包裝成字典回傳。這筆回覆會被加到 messages 列表的最後面。
    return {"messages": [response]}
```

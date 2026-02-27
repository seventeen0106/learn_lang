<div align="center">

# ✨ 一、環境建置

</div>

## 1️⃣ 安裝 langGraph 套件

### **colab**

```python
%%capture --no-stderr
%pip install --quiet langchain
%pip install --quiet langchain-openai
%pip install --quiet langgraph
```

### **jupyter(cmd)**

```cmd
:: Day3 LangChain、LangGraph : MVP 試跑
pip install langchain
pip install langchain-openai
pip install langgraph

:: 抓 .env 檔案
pip install python-dotenv

:: Day4 Langchain-community : state、node、edge、graph、display
pip install langchain-community

:: Day5 : LangChain - stream、LCEL、parser、event ； LangGraph - stream（無須安裝）
```

## 2️⃣ 抓API Key

### **colab**

```python
import os
import getpass
from google.colab import userdata

os.environ["OPENAI_API_KEY"] = userdata.get('OPENAI_API_KEY')
```

### **jupyter(cmd)**

> 先在 cmd 安裝

```cmd
pip install python-dotenv
```

```python
from dotenv import load_dotenv

# 在 .env 中抓API key
load_dotenv()
```

<div align="center">

# ✨ 環境建置

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
pip install langchain
pip install langchain-openai
pip install langgraph
```

## 2️⃣ 抓API Key

### **colab**

```python
import os
import getpass
from google.colab import userdata
```

### **jupyter(cmd)**

```cmd
pip install python-dotenv
```

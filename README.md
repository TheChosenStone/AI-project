# 运动鞋店铺知识库问答系统

这是一个基于 RAG (Retrieval-Augmented Generation，检索增强生成) 架构的运动鞋店铺知识库问答系统。系统能够根据用户输入的问题，从本地私有知识库中精准检索相关业务规则（如退换货、尺码、发货政策等），并结合大语言模型生成准确、连贯的专属客服回答。

## 目录结构

```text
├── conf/
│   └── config.template.json    # 配置文件模板（包含 MySQL 与 OpenAI 密钥配置）
├── database/
│   ├── init_schema.sql         # 数据库表结构建表脚本 (创建 ai_context 表)
│   └── seed_knowledge.sql      # 知识库初始数据 SQL 导入脚本
├── data/                       # 数据与索引存储目录（存放原始文本与生成的向量文件）
│   ├── store_knowledge.index   # Faiss 向量库索引文件 (运行代码后生成)
│   └── 运动鞋店铺知识库.txt       # 知识库原始文本文件
├── src/                        # 核心源代码目录
│   ├── rag_chat_bot.py         # 终端控制台 RAG 问答机器人交互主程序
│   ├── rag_chat_api.py         # RAG 问答机器人 Web API 服务端程序
│   ├── text_vectorizer.py      # 核心逻辑：文本向量化、MySQL入库与 Faiss 索引构建
│   └── vectorizer_test.py      # 向量检索功能独立测试脚本
```

## 配置说明

在使用前，需要根据 `conf/config.template.json` 创建 `config.json` 文件，填入相应的配置。

用户应当自行创建 `data/` 目录，并将知识库文本文件 `运动鞋店铺知识库.txt` 放入其中。

## 使用方法

1. 安装依赖：
```bash
pip install -r requirements.txt
```

2. 初始化数据库并构建知识库向量索引：
```bash
python src/text_vectorizer.py
```

3. 测试向量检索功能是否正常：
```bash
python src/vectorizer_test.py
```

4. 启动 RAG 问答机器人 (控制台交互模式)：
```bash
python src/rag_chat_bot.py
```

5. 启动 RAG 问答机器人 Web API 服务：
```bash
python src/rag_chat_api.py
```

## 测试样例

### 知识库向量化功能测试
输出示例：
```
搜索查询: 如何退换货？
----------
ID: 5
相似度距离: 0.2745
文本内容: 退换货政策: 自收到商品12天内，商品未使用且原包装完好可申请退换货，需提供订单号和付款截图。

ID: 8
相似度距离: 0.3190
文本内容: 注意事项: 特价和促销商品不支持退换货，退回商品需妥善包装避免运输损坏。

ID: 6
相似度距离: 0.3234
文本内容: 退款处理: 商品返回检查无误后，退款将在11个工作日内处理完成。
```

### RAG 问答机器人
输入示例：
```
你: 如何退换货？
```

输出示例：
```
ChatGPT: 用户可以在收到商品后的12天内申请退换货。需要确保商品未使用且原包装完好。在申请退换货时，用户需要提供订单号和付款截图作为凭证。请注意，特价和促销商品是不支持退换货的，退回商品时，请妥善包装以避免运输损坏。一旦商品返回并经过检查确认无误，退款将在11个工作日内处理完成。
```

### RAG 问答机器人Web API
请求示例：
```bash
curl -X POST http://localhost:8080/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "鞋子有哪些尺码可以选择呢",
    "historyMessages": ["历史聊天内容"]
  }'
```

响应示例：
```json
{
    "response": "根据产品信息提供的尺码选择范围，您可以在30至48的范围内选择适合您的尺码。具体而言，儿童款的尺码范围是30至36，而成人款的尺码范围是36至48。您可以根据自己的脚长来选择最适合您的尺码。",
    "status": "success"
}
```

## 开发说明

- `text_vectorizer.py`: 负责文本向量化，使用 OpenAI 的 embedding 模型
- `rag_chat_bot.py`: 实现 RAG 架构的问答逻辑
- `rag_chat_api.py`: 实现 RAG 问答机器人的Web API接口


## 注意事项

- 请确保你的网络环境能够正常访问 OpenAI API，或配置了相应的代理。
- 首次运行向量化脚本时需要建立向量库并请求外部 API，可能需要一定时间，请耐心等待。
- 建议使用 Python 3.8 或以上版本
- Web API 默认运行在 8080 端口，可通过配置文件修改

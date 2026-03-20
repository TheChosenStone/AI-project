运动鞋店铺知识库问答系统
这是一个基于 RAG (Retrieval-Augmented Generation，检索增强生成) 架构的运动鞋店铺知识库问答系统。系统能够根据用户输入的问题，从本地私有知识库中精准检索相关业务规则（如退换货、尺码、发货政策等），并结合大语言模型生成准确、连贯的专属客服回答。

目录结构
Plaintext

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
├── requirements.txt            # 项目依赖包清单
└── .gitignore                  # Git 忽略文件配置
配置说明
环境配置：在使用前，请复制 conf/config.template.json 并重命名为 conf/config.json，在其中填入你真实的 MySQL 数据库连接信息以及 OpenAI API Key。

数据准备：请确保项目根目录下存在 data/ 文件夹，并将你的业务规则文本命名为 运动鞋店铺知识库.txt 放入其中。

数据库准备：确保你的 MySQL 服务已启动，并在配置项中指定了正确的 database 名称。

使用方法
⚠️ 注意：请始终在项目根目录下执行以下命令，以确保相对路径能够正确解析。

1. 安装依赖：

Bash

pip install -r requirements.txt
2. 初始化数据库并构建知识库向量索引：
（此步骤会将 txt 文本写入 MySQL，并调用 Embedding 模型生成 Faiss 索引保存到本地）

Bash

python src/text_vectorizer.py
3. 测试向量检索功能是否正常：

Bash

python src/vectorizer_test.py
4. 启动 RAG 问答机器人 (控制台交互模式)：

Bash

python src/rag_chat_bot.py
5. 启动 RAG 问答机器人 Web API 服务：

Bash

python src/rag_chat_api.py
测试样例
知识库向量化功能测试 (vectorizer_test.py)
输出示例：

Plaintext

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
RAG 问答机器人控制台交互 (rag_chat_bot.py)
输入示例：

Plaintext

欢迎使用RAG增强的ChatGPT! 输入'quit'退出对话，输入'clear'清除对话历史。

你: 如何退换货？
输出示例：

Plaintext

ChatGPT: 用户可以在收到商品后的12天内申请退换货。需要确保商品未使用且原包装完好。在申请退换货时，用户需要提供订单号和付款截图作为凭证。请注意，特价和促销商品是不支持退换货的，退回商品时，请妥善包装以避免运输损坏。一旦商品返回并经过检查确认无误，退款将在11个工作日内处理完成。
RAG 问答机器人 Web API (rag_chat_api.py)
请求示例：

Bash

curl -X POST http://localhost:8080/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "鞋子有哪些尺码可以选择呢",
    "historyMessages": []
  }'
响应示例：

JSON

{
    "response": "根据产品信息提供的尺码选择范围，您可以在30至48的范围内选择适合您的尺码。具体而言，儿童款的尺码范围是30至36，而成人款的尺码范围是36至48。您可以根据自己的脚长来选择最适合您的尺码。",
    "status": "success"
}
开发说明
src/text_vectorizer.py: 负责连接 MySQL 进行数据清洗与存取，并使用 OpenAI 的 text-embedding-ada-002 模型将文本转化为向量，最终利用 Faiss 构建并持久化本地索引库。

src/rag_chat_bot.py: 实现了 RAG 架构的核心逻辑，包含对话历史管理 (ConversationManager)，并支持多轮对话、流式打字机输出及用户告别意图识别。

src/rag_chat_api.py: 基于 Flask 框架封装的 RESTful 服务端，对外提供标准 JSON 问答接口及流式 (Stream) 接口，支持跨域访问 (CORS)。

database/: 包含数据库初始化的结构化 SQL 脚本，便于项目迁移和部署。

注意事项
请确保你的网络环境能够正常访问 OpenAI API，或配置了相应的代理。

config.json 包含敏感密钥，已加入 .gitignore，请勿将其提交到公开的代码仓库中。

首次运行向量化脚本时需要建立向量库并请求外部 API，可能需要一定时间，请耐心等待。

Web API 默认运行在 8080 端口，如果被占用，可通过修改代码中的参数调整。

建议使用 Python 3.8 或以上版本运行本项目。

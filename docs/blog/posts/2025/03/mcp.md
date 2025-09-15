---
title: MCP
authors:
  - ysh
date: 2025-09-11
categories:
  - AI
slug: mcp
---

MCP（Model Context Protocol，模型上下文协议）是一种开放协议，旨在实现大型语言模型（LLM）应用与外部数据源、工具和服务之间的无缝集成。通过标准化模型与外部资源的交互方式，MCP 提升了 LLM 应用的功能性、灵活性和可扩展性。
<!-- more -->

!!!tip
    可将 MCP 协议想象成大模型世界的 USB 协议 或 应用商店。

## 为啥需要MCP

在没有 MCP 之前，AI 助手的能力存在一些明显的局限性：

1. **知识截止问题**：模型的知识仅限于其训练数据截止的日期，无法获取最新的信息（如今天的新闻、股价）。（该问题也可使用RAG解决）

2. **无法执行操作**：模型本身无法直接操作你的电脑，比如帮你读取一个本地文件、运行一个脚本或发送邮件。

3. **缺乏个性化上下文**：模型无法访问你的私人或工作数据，比如你的代码库、笔记、公司数据库，因此提供的建议可能不够精准。

4. **集成困难且不安全**：每个AI应用（如Claude、Cursor）都需要自己单独实现各种插件和集成，过程复杂，且安全难以保障。用户可能需要把自己的API密钥暴露给AI应用。

MCP 正是为了解决这些问题而诞生的。它通过一种标准化的方式，让模型的能力变得可扩展和动态化。


## MCP的核心架构

MCP 协议定义了几个关键组件：

1. **MCP客户端**：这是集成了 MCP 协议的 LLM 应用或平台（如Cursor、VScode、Claude桌面应用）。MCP 客户端负责与 MCP 服务器通信，发送请求并处理响应。

2. **MCP服务器**：这是实现了 MCP 协议的服务端组件，负责管理和提供各种插件和集成。MCP 服务器接收来自 MCP 客户端的请求，并将其路由到相应的插件或服务。

3. **传输层**：通信的传输层是 stdio（标准输入/输出）。客户端通过命令行启动服务器进程，并通过管道（pipe）与服务器的 stdin 和 stdout 进行数据交换。这种设计意味着服务器是在用户本地机器上运行的独立进程，而不是由 AI 公司控制的远程服务。你的数据和凭证永远不会离开你的机器，除非服务器明确设计为连接远程服务。

对于开发者来说，MCP 提供了一套标准的 API 和数据格式，使得创建新的插件和集成变得简单且一致。开发者只需实现 MCP 服务器接口，即可让他们的服务被任何支持 MCP 的客户端所使用。

## 实现一个简单的MCP服务器

目前流行使用Python或Node.js来实现MCP服务器。

下面是一个使用Node.js实现的简单MCP服务器示例：

1.首先创建项目目录并初始化：

```bash
npm init -y
npm install @modelcontextprotocol/sdk zod@3
```

<br>

2.在src目录下创建index.js：

```javascript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { z } from 'zod';

const version = '1.0.0';

// 创建MCP服务器实例
const server = new McpServer({
    name: 'Example MCP Server',
    version: version,
    capabilities: {
        tools: {}
    }
})


// 注册一个简单的工具
server.tool(
    'sayHello', // 工具名称
    'Says hello to the given name', // 工具描述
    {
        state: z.string().min(1).max(100) // 输入参数的Zod模式
    },

    // 工具的实现逻辑
    async ({ state }) => {

        // 返回结果
        return {
            content: [
                {
                    type: "text",
                    text: `Hello, ${state}! (from version ${version})`
                }
            ]
        };
    }
)

// 启动服务器并监听来自MCP客户端的请求
async function main() {
    const transport = new StdioServerTransport();
    await server.connect(transport);
    console.log('Server is running...');
}

// 启动主函数
main().catch((err) => {
    console.error('Error starting server:', err);
    process.exit(1);
});
```

<br>

3.在MCP客户端如Vscode中配置MCP服务器：

```json
{
    "modelcontextprotocol.mcpServers": [
        {
            "name": "Example MCP Server",
            "command": "node",
            "args": ["path/to/your/src/index.js"]
        }
    ]
}
```

<br>

4.启动MCP服务，并发送消息给LLM：
“请使用MCP工具sayHello，参数state为“World”。”

运行结果： 
```
Hello, World! (from version 1.0.0)
```

!!!quote "参考文档"
    [MCP官方文档](https://modelcontextprotocol.io/docs)
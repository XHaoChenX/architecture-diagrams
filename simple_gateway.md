
# Gateway Architecture: 截图例子

```mermaid
flowchart TD
    subgraph Lark["飞书 Lark"]
        WS[WebSocket<br>WSClient]
    end

    subgraph bot["bot"]
        Receive[接收消息]
        Parse[解析 content]
        Dedup[去重判断]
    end

    subgraph gateway["gateway（Agent）"]
        Agent[Gateway Agent<br>编排与决策]
        SysPrompt[系统提示词<br>大白助手]
        ToolCall{工具调用?}
        LLM[LLM 调用<br>Kimi API Provider]
    end

    subgraph tools["tools/"]
        Capture[captureScreen]
        ScreenshotExe[ScreenshotApp.exe]
        SaveImage[保存 cur.jpg]
    end

    subgraph LarkIM["飞书 IM API"]
        UploadImg[上传图片]
        SendText[发送文本]
        SendImg[发送图片消息]
    end

    WS --> Receive
    Receive --> Parse
    Parse --> Dedup
    Dedup -->|"text"| Agent
    Agent --> SysPrompt
    SysPrompt --> LLM
    LLM --> Agent
    Agent --> ToolCall

    ToolCall -->|"无工具调用"| SendText
    ToolCall -->|"captureScreen"| Capture
    Capture --> ScreenshotExe
    ScreenshotExe --> SaveImage
    SaveImage --> UploadImg

    UploadImg --> SendImg
    SendText --> LarkIM

    classDef external fill:#f9f,stroke:#333,stroke-width:2px
    classDef process fill:#bbf,stroke:#333,stroke-width:1px
    classDef ai fill:#dfd,stroke:#333,stroke-width:1px

    class WS,LarkIM external
    class Receive,Parse,Dedup,Capture,SaveImage,UploadImg,SendText,SendImg process
    class Agent,SysPrompt,ToolCall,LLM,ScreenshotExe ai
```

## 消息流程

1. **接收**: 飞书 WebSocket 接收用户消息
2. **解析**: bot.ts 解析 JSON content 提取 text
3. **Agent 处理**: gateway 作为 Agent 负责上下文、提示词、决策
4. **LLM 调用**: Agent 调用 Kimi 等 AI API 模型提供商获取回复
5. **判断**: Agent 判断是否需要工具，无工具→文本回复，有 captureScreen→截图
6. **响应**: 通过飞书 IM API 发送文本或图片

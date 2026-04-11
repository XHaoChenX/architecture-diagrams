
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
        Agent[Gateway Agent<br>消息接入与分发]
        Dabai[Dabai Agent 引擎<br>编排与决策]
        ToolCall{是否触发工具?}
        LLM[模型调用<br>Kimi API Provider]
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
    Agent --> Dabai
    Dabai --> LLM
    LLM --> Dabai
    Dabai --> ToolCall

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
    class Agent,Dabai,ToolCall,LLM,ScreenshotExe ai
```

## 消息流程

1. **接收**: 飞书 WebSocket 接收用户消息
2. **解析**: bot.ts 解析 JSON content 提取 text
3. **引擎调用**: gateway 将请求交给 `Dabai Agent` 引擎处理
4. **模型交互**: `Dabai Agent` 负责与 Kimi 等模型提供商交互并完成推理
5. **工具判断**: 由 `Dabai Agent` 判断是否需要工具，无工具则返回文本，有 `captureScreen` 则进入截图流程
6. **响应**: 通过飞书 IM API 发送文本或图片

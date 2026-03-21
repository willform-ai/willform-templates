---
category: ai-agent
tags: [langgraph, langchain, graph, ai, mcp, python]
---

# LangGraph 에이전트 프레임워크

MCP 툴이 Willform에 자동 연결된 상태 기반 LangGraph 에이전트를 배포하세요. 분기, 루프, 영속 상태가 있는 복잡한 에이전트 워크플로우를 구축하세요.

## 배포 결과

- 커스텀 그래프 정의로 실행되는 Python 워커
- MCP 엔드포인트 사전 구성 — Willform의 42가지 툴 바로 사용 가능
- 에이전트 간 메시지를 위한 A2A 엔드포인트
- 그래프 상태 및 체크포인팅을 위한 영구 볼륨
- OpenAI, Anthropic, LangChain 호환 모델 지원

## 사전 준비

아래 플레이스홀더를 실제 값으로 교체하세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys)에서 생성 | LangGraph 노드 LLM 추론 | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com/settings/keys)에서 생성. 사용하지 않으면 비워두세요. | Claude 모델 LLM 추론 | `sk-ant-api03-...` |
| `YOUR_WILLFORM_API_KEY` | [agent.willform.ai](https://agent.willform.ai) Dashboard > Settings > API Keys | Willform MCP 툴 호출 인증 | `wf_sk_...` |

## 배포 후 안내

LangGraph 워커가 실행 중입니다. `MCP_ENDPOINT` 환경 변수가 자동 주입됩니다:

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

mcp_client = MultiServerMCPClient({
    "willform": {
        "url": os.environ["MCP_ENDPOINT"],
        "transport": "streamable_http",
    }
})
tools = await mcp_client.get_tools()
```

**A2A 엔드포인트**: https://agent.willform.ai/a2a

## Deploy Config

```json
{
  "name": "langgraph",
  "chartType": "worker",
  "image": "python:3.12-slim",
  "replicas": 1,
  "volumeSizeGb": 4,
  "volumeMountPath": "/app/data",
  "env": {
    "OPENAI_API_KEY": "YOUR_OPENAI_API_KEY",
    "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
    "WILLFORM_API_KEY": "YOUR_WILLFORM_API_KEY"
  }
}
```

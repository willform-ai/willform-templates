---
category: ai-agent
tags: [crewai, multi-agent, ai, mcp, python]
---

# CrewAI 멀티 에이전트 프레임워크

MCP 툴이 Willform에 자동 연결된 CrewAI 멀티 에이전트 시스템을 배포하세요. Crew, 에이전트, 태스크를 정의하면 플랫폼이 오케스트레이션을 처리합니다.

## 배포 결과

- 커스텀 Crew 정의로 실행되는 Python 워커
- Willform의 42가지 MCP 툴이 사전 연결된 MCP 엔드포인트
- 에이전트 간 통신을 위한 A2A 엔드포인트
- Crew 상태 및 메모리를 위한 영구 볼륨
- OpenAI, Anthropic, LiteLLM 호환 모델 지원

## 사전 준비

아래 플레이스홀더를 실제 값으로 교체하세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys)에서 생성 | CrewAI 에이전트 LLM 추론 | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com/settings/keys)에서 생성. 사용하지 않으면 비워두세요. | Claude 모델 LLM 추론 | `sk-ant-api03-...` |
| `YOUR_WILLFORM_API_KEY` | [agent.willform.ai](https://agent.willform.ai) Dashboard > Settings > API Keys | Willform MCP 툴 호출 인증 | `wf_sk_...` |

## 배포 후 안내

CrewAI 워커가 실행 중입니다. `MCP_ENDPOINT` 환경 변수가 자동 주입되어 있습니다. Crew 정의에서 바로 사용하세요:

```python
from crewai_tools import MCPServerAdapter

mcp_tools = MCPServerAdapter({"url": os.environ["MCP_ENDPOINT"]})
```

**A2A 엔드포인트**: https://agent.willform.ai/a2a

## Deploy Config

```json
{
  "name": "crewai",
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

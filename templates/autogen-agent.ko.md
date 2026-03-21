---
category: ai-agent
tags: [autogen, microsoft, multi-agent, ai, mcp, python]
---

# AutoGen 멀티 에이전트 시스템

MCP 툴이 Willform에 자동 연결된 Microsoft AutoGen 멀티 에이전트 시스템을 배포하세요. 복잡한 태스크를 협력해 해결하는 대화형 에이전트 네트워크를 구축하세요.

## 배포 결과

- 커스텀 에이전트 정의로 실행되는 Python 워커
- MCP 엔드포인트 사전 구성 — Willform의 42가지 툴 자동 사용 가능
- 외부 에이전트 통신을 위한 A2A 엔드포인트
- 대화 기록 및 상태를 위한 영구 볼륨
- OpenAI, Anthropic, Azure OpenAI 호환

## 사전 준비

아래 플레이스홀더를 실제 값으로 교체하세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys)에서 생성 | AutoGen 에이전트 LLM 추론 | `sk-proj-...` |
| `YOUR_ANTHROPIC_API_KEY` | [Anthropic Console](https://console.anthropic.com/settings/keys)에서 생성. 사용하지 않으면 비워두세요. | Claude 모델 LLM 추론 | `sk-ant-api03-...` |
| `YOUR_WILLFORM_API_KEY` | [agent.willform.ai](https://agent.willform.ai) Dashboard > Settings > API Keys | Willform MCP 툴 호출 인증 | `wf_sk_...` |

## 배포 후 안내

AutoGen 워커가 실행 중입니다. `MCP_ENDPOINT` 환경 변수가 자동 주입됩니다:

```python
from autogen.mcp import create_toolkit
from autogen import AssistantAgent

toolkit = await create_toolkit(server_params={"url": os.environ["MCP_ENDPOINT"]})
agent = AssistantAgent("assistant", tools=toolkit.tools)
```

**A2A 엔드포인트**: https://agent.willform.ai/a2a

## Deploy Config

```json
{
  "name": "autogen",
  "chartType": "worker",
  "image": "willformhq/autogen-starter:latest",
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

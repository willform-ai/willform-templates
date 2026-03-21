---
category: ai-agent
tags: [openai, swarm, multi-agent, ai, mcp, python]
---

# OpenAI Swarm 에이전트 네트워크

MCP 툴이 Willform에 자동 연결된 OpenAI Swarm 멀티 에이전트 네트워크를 배포하세요. 각 에이전트가 하나의 태스크에 특화되어 다음 에이전트에게 핸드오프하는 경량 아키텍처입니다.

## 배포 결과

- 커스텀 에이전트 네트워크로 실행되는 Python 워커
- MCP 엔드포인트 사전 구성 — Willform의 42가지 툴 바로 사용 가능
- 외부 조율을 위한 A2A 엔드포인트
- 실행 기록을 위한 영구 볼륨
- 공식 OpenAI Swarm 라이브러리 기반

## 사전 준비

아래 플레이스홀더를 실제 값으로 교체하세요.

| 변수 | 발급 방법 | 용도 | 예시 |
|---|---|---|---|
| `YOUR_OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys)에서 생성 — Swarm 필수 | LLM 추론 | `sk-proj-...` |
| `YOUR_WILLFORM_API_KEY` | [agent.willform.ai](https://agent.willform.ai) Dashboard > Settings > API Keys | Willform MCP 툴 호출 인증 | `wf_sk_...` |

## 배포 후 안내

Swarm 워커가 실행 중입니다. `MCP_ENDPOINT` 환경 변수가 자동 주입됩니다:

```python
import os
from swarm import Swarm, Agent

# MCP_ENDPOINT로 Willform 툴 로드 후 Agent functions에 매핑
agent = Agent(name="Willform Agent", functions=[...])
client = Swarm()
response = client.run(agent=agent, messages=[{"role": "user", "content": "앱 배포해줘"}])
```

**A2A 엔드포인트**: https://agent.willform.ai/a2a

## Deploy Config

```json
{
  "name": "swarm",
  "chartType": "worker",
  "image": "python:3.12-slim",
  "replicas": 1,
  "volumeSizeGb": 2,
  "volumeMountPath": "/app/data",
  "env": {
    "OPENAI_API_KEY": "YOUR_OPENAI_API_KEY",
    "WILLFORM_API_KEY": "YOUR_WILLFORM_API_KEY"
  }
}
```

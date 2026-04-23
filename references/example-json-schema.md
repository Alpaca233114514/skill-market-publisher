# .skill-examples.json Schema

```json
{
  "model": "kimi k2.6",
  "examples": [
    {
      "prompt": "用户提问文本",
      "aiResponses": [
        {
          "type": "thinking",
          "content": "AI 思考过程"
        },
        {
          "type": "toolcall",
          "toolName": "ToolName",
          "toolInput": { "key": "value" }
        },
        {
          "type": "message",
          "content": "AI 最终回复"
        }
      ],
      "model": "kimi k2.6"
    }
  ]
}
```

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | Yes | Default model for all examples |
| `examples` | array | Yes | List of usage examples |
| `examples[].prompt` | string | Yes | User prompt text |
| `examples[].aiResponses` | array | Yes | Collected AI trajectory |
| `examples[].aiResponses[].type` | string | Yes | `thinking` / `toolcall` / `message` |
| `examples[].aiResponses[].content` | string | Yes* | Content for thinking/message |
| `examples[].aiResponses[].toolName` | string | Yes* | Tool name for toolcall |
| `examples[].aiResponses[].toolInput` | object | Yes* | Tool arguments for toolcall |
| `examples[].model` | string | No | Per-example model override |

*Required depending on `type`.

## Minimum Valid Example

At least one example with `prompt` and non-empty `aiResponses` array is required.

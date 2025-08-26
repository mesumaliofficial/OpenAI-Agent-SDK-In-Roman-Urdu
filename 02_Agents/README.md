## 🔹 Agent
### 🔸Agent
- Agent ek Large Language Model (LLM) hota hai jo instructions aur tools ke sath configure kiya jata hai, Yeh app ka core hissa hota hai jo user input ke mutabiq sochta, decision leta, aur kaam karta hai.

### 🔸Basic Configuration
Agents ki kuch important properties hoti hain jo apko configure karni hoti hain.
- **name:** Ye Agent ki idntification hoti hay. **Required**.

- **instructions:** Ye agent ka persona ya system prompt hota hai jo batata hai ke agent kis tarah behave karega **Optional**.

- **model:** Yeh batata hay konsa LLM use karna hay **Required**.
- **model_settings:** Is mein aap kuch tuning parameters set kar sakte ho jaise: temperature, top_p **Optional**.

- **tools:** ye wo external functions, APIs ya utilities hoti hain jo agent apne task complete ky leye use karta hay **Optional**.

```bash
from agents import Agent, ModelSettings, function_tool

@function_tool
def get_weather(city: str) -> str:
     """returns weather info for the specified city."""
    return f"The weather in {city} is sunny"

agent = Agent(
    name="Neura",
    instructions="Always respond in haiku form",
    model="gemini-2.0-flash",
    tools=[get_weather],
)
```

---

### 🔸Agents Arguments


| **Argument**          | **Required / Optional** | **Type**                           | **Short Description**                                 |
| --------------------- | ----------------------- | ---------------------------------- | ----------------------------------------------------- |
| `name`                | Required                | `str`                              | Agent ka unique identifier                            |
| `instructions`        | Optional                | `str` / `callable`                 | System prompt ya logic (static ya dynamic)            |
| `prompt`              | Optional                | `Prompt` / `callable`              | Advanced/dynamic prompt configuration                 |
| `handoff_description` | Optional                | `str`                              | Description for handoffs (explain role in delegation) |
| `handoffs`            | Optional                | `list[Agent]`                      | Sub-agents jinko tasks delegate ho sakte hain         |
| `model`               | Optional                | `str` / `Model`                    | Specify LLM (default: GPT-4.1)                        |
| `model_settings`      | Optional                | `ModelSettings`                    | Model tuning (temperature, top\_p, tool choices)      |
| `tools`               | Optional                | `list[Tool]`                       | External APIs/functions agent use kar sakta hai       |
| `input_guardrails`    | Optional                | `list[InputGuardrail]`             | Validation checks before agent runs                   |
| `output_guardrails`   | Optional                | `list[OutputGuardrail]`            | Checks after response generated                       |
| `output_type`         | Optional                | `type` / `AgentOutputSchemaBase`   | Expected format of final output (default: `str`)      |
| `hooks`               | Optional                | `AgentHooks`                       | Lifecycle event callbacks (logging/debugging)         |
| `tool_use_behavior`   | Optional                | `str` / `StopAtTools` / `callable` | Defines how tool outputs are handled                  |
| `reset_tool_choice`   | Optional                | `bool`                             | Reset tool selection after use to avoid loops         |

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

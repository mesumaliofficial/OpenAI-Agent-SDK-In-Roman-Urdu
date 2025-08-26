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

### 🔸Output Types → Api Refrences
```python
output_type: type[Any] | AgentOutputSchemaBase | None = None
```
Ye option decide karta hay keh agent ka output kis type ka object hoga.
- Agar aap kuch bhi na do, to **default string (str)** return hoge.
- Zyada cases mein aapko Python ka koi structured type use karna chahiye (jaise `dataclass`, `Pydantic model`, `TypedDict`, etc.) taa keh output organized aur predictable aaye.

1. **Non-strict Schema**   
Agar apko thora flexible output chahiye (matlab model chhoti-moti format ki galtiya ignore kar dy aur data accept kar ly), tw `AgentOutputSchema(MyClass, strict_json_schema=False)` use karo.

2. **Custom Schema**
Agar apko apna khud ka schema banana hai (SDK ka auto-schema use kiye bina), to ek class banao jo AgentOutputSchemaBase se inherit kare aur usko pass kar do.

#### Example:
Strict => Agar schema mein `title: str, price: float` likha hay, aur agent price string bana de "10" instead of 10.0, to error aayega.

Non-Strcit => Agar agent thora format different bana de (e.g. string instead of float), to SDK usko accept kar lega aur internally try karega correct karny ki. 

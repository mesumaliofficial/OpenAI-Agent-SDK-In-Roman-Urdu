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

### 🔸Context

- Agents apne context ky leye generic hoty hain, Context ek dependency-injection tool hay: yeh ek object hota hay jo hum create karty hain aur `Runner.run()` ko pass karty hain, har agent tool handoff wagera ko deya jata hay, aur yeh agentrun ky leye dependencies aur state ky leye ek grab bag ka kam karta hay. ap kisi bhi python object ko context ky tor per dy sakty hain.

- Jo context `Runner.run` mein pass keya jaye wo local context hota hay, aur ye kuch is tarha sy work karta hay keh jesy hum 1 greet agent bana rahy ho, jo greet kary aur uska naam use kary tw out put kuch is tarha ka hoga. "Hello Mesum! How can I help you today?"

```bash
@dataclass
class UserContext:
    name: str
    uid: str
    is_pro_user: bool

    async def fetch_purchases() -> list[Purchase]:
        return ...

agent = Agent[UserContext](
    ...,
)
```

### 🔸Handoffs
- Handoffs aesy sub agents hoty hain jinhy main agent apna kam delegate karta sakta hay.
- Ap ek agents ki list provide karty ho aur agar agent decide karta hay delagate kary agar revelant ho,
- Yeh aik powerful pattern hai jo tumhein allow karta hai ke tum orchestrating modular, specialized agents bana sako, jis mein har ek agent ka specific task hota hay jis m wo expert ho.

```bash
from agents import Agent

booking_agent = Agent(...)
refund_agent = Agent(...)

triage_agent = Agent(
    name="Triage agent",
    instructions=(
        "Help the user with their questions."
        "If they ask about booking, handoff to the booking agent."
        "If they ask about refunds, handoff to the refund agent."
    ),
    handoffs=[booking_agent, refund_agent],
)
```

### 🔸Dynamic Instructions
- Aksar cases mein ap insturctuction agent create karty waqt hi dy sakty hain,
- Lekin agar ap chahen tw dynamic Instruction bhi dy sakty hain ek function ky zariye,
- Yeh function agent aur context ko receive karega, aur prompt return karyga.
- is mein dono function use keye jaa sakty hain `async` aur normal.

```bash
def dynamic_instructions(
    context: RunContextWrapper[UserContext], agent: Agent[UserContext]
) -> str:
    return f"The user's name is {context.context.name}. Help them with their questions."


agent = Agent[UserContext](
    name="Triage agent",
    instructions=dynamic_instructions,
)
```

### 🔸Lifecycle events (hooks)
- Kabhi kabhi ap chahty hain ke ap agent ki lifecycle ko observe karen for example ap events ko log karna chahtay hain, ya jab kuch khaas events hon to data pehlay se hi fetch kar lena chahtay hain, tw ap `hooks` property ka use karky hook kar sakty hain `AgentHooks` class ko subclass karen aur un method ko override karden jin mein ap intrested hain.

### 🔸Guardials
- Guardials ka matlab hota hay aesy checks ya validation chalana jo user ky input per lagty hain aur yeh agent ky sath parallel chalty hain, For example: Aap yeh dekh sakte ho ke user ka input relevant hai ya nahi, ya koi banned word to nahi use kiya gaya.


### 🔸Cloning/copying agents
- colne() method ky zariye ap kisi bhi agent ka clone kar sakty hain aur jo chahen usko override kar sakty hain

```bash
pirate_agent = Agent(
    name="Pirate",
    instructions="Write like a pirate",
    model="o3-mini",
)

robot_agent = pirate_agent.clone(
    name="Robot",
    instructions="Write like a robot",
)
```
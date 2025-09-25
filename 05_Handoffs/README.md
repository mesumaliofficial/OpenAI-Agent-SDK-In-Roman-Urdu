## 🔹 Handoffs

### 🔸What is Handoffs?

Imagine karen, Ap ek school mein ho jahan har teacher ka ek alag subject ka expert hay.

- Agar apko Math sy related question hay tw ap Math teacher ke pass jaowgy.
- Agar Science sy related question hay tw ap Science teacher ke pass jaowgy. 


**Handoff** bilkul isi tarha kam karta hay, Jab ek **Triage Agent** ko lagta hay keh user ka sawal kisi ek specific topic sy related hay, jesy "Math" ya "Science", tw wo us topic ky **specialized agent** ko handoffs kardeta hay.

#### when the Handoffs happens:

- Puri conversation history naye agent ke pass chali jati hay.
- Naya agent usi point se conversation continue karta hay.
- Issy system modular aur efficient ho jata hai.

Handoffs LLM ke liye tools ke tor par represent hoty hain. Agar ek agent `Refund Agent` ko handoff hota hay, to tool ka naam hoga `transfer_to_refund_agent`.

```python 
from agents import Agent

booking_agent = Agent(...)
refund_agent = Agent(...)

triage_agent = Agent(
    name="Triage agent",
    instructions=(
        "Help the user with their questions. "
        "If they ask about booking, hand off to the booking agent. "
        "If they ask about refunds, hand off to the refund agent."
    ),
    handoffs=[booking_agent, refund_agent],
)
```

Is code mein, jab triage_agent ko "booking" ya "refund" ka sawal milega, tw wo automatically booking_agent ya refund_agent ko handoff kar dega.

---

### 🔸How many types to define handoffs?
OpenAI agent SDK mein handoffs do tareqo sy define keya jaa sakty hain:

1. **Simple Handoffs:** Sabse easy method hay ap directly agent ko **Triage Agent** ki list mein pass karden. Jesy upper wale exampe mein `booking_agent`, Ye sab sy easy method hay.

2. **Custom Handoff** Ye tab use karen jab apko `Handoff()` function ka use karka handoff ka zyada control karna chahiye ho. Is mein ap handoff ka naam, description, aur sath hony waly actions ko customize kar sakty ho.

### 🔸`handoff()` function ke parameters

- **agent:** Jis agent ko ap handoff karna chahty hain

- **tool_name_override:** By default, Handoff.default_tool_name() function use hota hay jo transfer_to_<agent_name> banta hay. Aap isy override kar saktay hain.

- **tool_description_override:** Ap tool ki description ko badal sakty hain.

- **on_handoff:** Ye ek callback function hai jo handoff invoke hone par chalaya jata hay. Ye tab kam ata hay jab apko handoff invoke hote hi data fetching start karni ho. Ye function agent context leta hay, aur optionally LLM generated input bhi le sakta hai. Input data ka control input_type param ke zariye hota hay.

- **input_type:** Ye define karta hay keh handoff ko kis type ka input chahiye (optional). 

- **input_filter:** Ye allow karta hay keh next agent ko jo input mile usy filter kiya jaye.

- **is_enabled:** Handoff ko on/off karne ke liye.

--- 

### 🔸What is Handoff inputs?

Jab ek agent kisi dusry agent ko conversation handoff karta hay, tw wo sirf conversation history hi nahi, balki kuch extra information bhi sath bhej sakta hay. Is extra information ko **handoff input** kehte hain.

Sochen, ap ek student hain aur apko teacher ne kaha, "jaow, mein class ke dusre teacher ke pass tumhy bhej raha hun, aur unko bolna keh 'mujhe ye sawal isliye samajh nahi aaya kyun ki usmein addition ka concept hay'." Yahan par 'addition' ek extra information hay jo ako naye teacher ko deni hai.

Bilkul isi tarha, jab ek LLM (yani apka agent) handoff karta hay, tw wo handoff tool ke sath kuch data bhi bhej sakta hay agar ap chahen tw.

```python 
from pydantic import BaseModel

from agents import Agent, handoff, RunContextWrapper

class EscalationData(BaseModel):
    reason: str

async def on_handoff(ctx: RunContextWrapper[None], input_data: EscalationData):
    print(f"Escalation agent called with reason: {input_data.reason}")

agent = Agent(name="Escalation agent")

handoff_obj = handoff(
    agent=agent,
    on_handoff=on_handoff,
    input_type=EscalationData,
)
```

---

### 🔸What is Handoff Input filters?

Jab handoff hota hat, tw naya agent puri pehly ki conversation ko handle karta hay aur history dekh sakta hay. kabhi-kabhi naye agent ko sari purani batein nahi dikhani chahiye. `HandoffInputFilter` bilkul wohi kam karta hay.

Ye ek special function hai jo `HandoffInputData` ko le kar usmein se kuch information hata deta hai ya modify kar deta hay. Iska sabse zyada use tab hota hay jab ap conversation se tool calls (jaise web_search ya file_search) ko remove karna chahty hain taki naye agent ko un sab cheezon se confusion na ho.

#### Real-world Example:

Sochen, Triage Agent ne ek customer ki problem samajhny ky liye web_search tool ka use kiya. Ab wo billing agent ko handoff kar raha hay. Billing agent ko Triage Agent ki search history se matlab nahi hay.

Is case mein, hum `HandoffInputFilter` use karky Triage Agent ki sari tool calls ko `HandoffInputData` se hata sakty hain.

---

### 🔸Recommended prompts

Handoff Prompts wo **instructions** hain jo hum agent ki instructions field mein shamil karte hain, taki LLM Agent SDK ke multi-agent system ko thek sy samajh saky.

Iska do purpose hain:

1. LLM ko System Samjhana: LLM ko batana ki wo sirf ek chat bot nahi hay, balki ek bary multi-agent system (Agents SDK) ka hissa hay jahan wo dusry agents ko kam handoff kar sakta hay.

2. User Interface Maintain Karna: LLM ko ye batana ki jab handoff ho, tw woh user se koi technical baat na kary. Handoffs background mein hone chahiye, aur user ko pata bhi na chale ki agent badal gaya hay.

---

### 🔸What is RECOMMENDED_PROMPT_PREFIX?

RECOMMENDED_PROMPT_PREFIX ek pre-written system instruction ka long string hay jo Agent SDK ne apke liye ready keya hay. Jab ap isko apne agent ki instructions mein shamil karte hain, toh yeh LLM ko zarori information de deta hai.

#### Prefix Ke Main Points

- **Prefix Statement:** You are part of a multi-agent system called the Agents SDK...
- **Meaning:** LLM ko batata hay keh wo ek complex system ka part hay.

- **Prefix Statement:** Agents uses two primary abstraction: Agents and Handoffs.
- **Meaning:** System ke main components (hisse) introduce karta hai.

- **Prefix Statement:** Handoffs are achieved by calling a handoff function, generally named transfer_to_<agent_name>.
- **Meaning:** LLM ko action sikhata hay: handoff karny ke leye usko tool call karna hay, aur tool ka naam kaisa dikhega.

- **Prefix Statement:** Transfers between agents are handled seamlessly in the background; do not mention or draw attention to these transfers in your conversation with the user.
- **Meaning:** Ye sabse zaruri instruction hay. Ye user experience ko aacha rakhta hay. LLM ko confidentiality maintain karny ko bolta hai.

--- 

### 🔸Resources & Practical Demos

- [OpenAI Agent Handoffs & Handoff Colab Notebook](https://colab.research.google.com/drive/15LO536YAQHTrIOGl5qA3xmS9x7LVumAZ?usp=sharing) → Interactive example with code and usage.

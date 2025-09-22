## 🔹 Straeming
Streaming allow karta hay keh ap agent ko run karty time hi uski updates ko subscribe kar saken. Bina pury final output aane ky wait keye huwy.  
Ya user ko **live progress updates** aur **partial responses** dikhany ky leye useful hota hay.

- `Runner.run_streamed()` → Ye ek streaming start karta hay aur RunResultStreaming object return karta hai.
- `RunResultStreaming` → Streaming ka result wrapper hay jo RunResultBase ko inherit karta hay. jisme methods aur attributes hoty hain (jaise stream_events(), cancel(), is_complete etc.).
- `result.stream_events()` → Ek async iterator deta hay jo **StreamEvent** objects emit karta hay. Har event ka ek type hota hai (e.g. raw_response_event, run_item_stream_event) aur uske revelant data hota hay.

### 🔸 What is streaming? Simple Analogy
Streaming kya hoti hay. Imagine karo, ap aur apka dost video call par bat kar rahy ho. Apke dost ko puri video file ek sath download karny ki zarurat nahi parti, right? Bulky video choty choty chunks mein ati rehti hay, aur ap usko foran dekh sakty hain. Yeh hi streaming hai.

### 🔸 StreamEvent
StreamEvent ek type hay jo batata hay streaming sy aane waly events ki kya type hogi.

```python
StreamEvent: TypeAlias = Union[
    RawResponsesStreamEvent,
    RunItemStreamEvent,
    AgentUpdatedStreamEvent,
]
```
**StreamEvent** 3 tarha ky events ko represent kar sakta hay:

1. `RawResponsesStreamEvent` → Har chota text (token) ya raw LLM response.
2. `RunItemStreamEvent` → High-level output jaise message, tool call ya tool output.
2. `AgentUpdatedStreamEvent` → Jab agent ka state change hota hay yani (e.g., handoff ya updated instructions).

---

### 🔸 Why is Streaming Important?
Socho apny ek agent banaya aur user ne usko article likhne ko kaha. Normally wo agent pura article apne paas likhega aur phir ek sath output dega. Jab tak wo likh raha hoga tab tak user ko sirf `"Loading..."` ya `"Thinking..."` dekhaye dega.  
Ye experience acha nh lagta right ? aesa lagta hay computer hang hogaya hay.

### 🔸 Benefits of Streaming:
1. **UX bhetar hota hay:** Jesy hi agent 1 lafz ya sentence likhta hay wo foran screen per nazar any lagta hay keh agent kam kar raha hay.

2. **Fast feel hota hai:** Technically pura response any mein utna hi time lagega lekin user ko aesa feel hoga keh response jaldi aa raha hay. kyun ki kuch part foran nazar aa jata hay.

3. **Transparency:** User ko bata sakty hain AI kya kar raha hay (jesy tool call kiya ya text likh raha hai).

---

### 🔸 Normal Run vs Streamed Run
- **Normal Run** Jab ap ek normal `Runner.run()` chalaty hain tw ye method kam shuru karta hay aur tab tak wait karta hay jab tak pura output generate nh ho jata. Jab generate h jata hay tw `RunResult` object return karta hay.

- **Streamed Run:** Jab ap `Runner.run_streamed()` chalaty hain tw ye method kam shuru karta hay. Lekin ye wait nh karwata pury kam kahatm hony tak bulky ek `RunResultStreaming` object return karta hay.  


### 🔸 RunResultStreaming Simple Pipeline Analogy
- Jab ap stream_events() call karty ho tw ye pipeline open ho jata hay.
- Ab jesy hi AI agent apna kam shuru karta hay (output banana, tools call karna, etc.), wo events pipeline me bhejta rehta hay.
- Apka code in events ko pipeline se receive karta hay aur user ko screen par dikhata hay. Bilkul wesy hi jesy video call me har frame foran ata hay.

---


### 🔸 Raw response events
- Jab LLM (AI model) output banata hay, wo pura ek sath nahi bhejta.
- Wo output ko choty choty tukro (tokens) me tor ke bhejta hay.
- Har tukra = ek **event** hota hay.

**Example**: Agar AI likh raha hai "Hello"

- Event 1 → "H"
- Event 2 → "e"
- Event 3 → "l"
- Event 4 → "l"
- Event 5 → "o"

Isko code me **event.data.delta** ke zariye access karte ho.

- **event** → Har update ka wrapper object hota hay (uska type + data).
- **event.data** → Us event ka actual payload/data hota hai (e.g. ResponseTextDeltaEvent).
- **event.data.delta** → Wo naya chota text/tukra hota hay jo abhi generate huwa hai. 

### 🔸 Real World Analogy
Socho ap ek **chef** ho jo cooking karty waqt har choti cheez ki report deta hay:

- "Maine ab namak dala"
- "Ab mirch daali"
- "Ab thoda paani daal raha hoon"

Yani bilkul detailed updates, step by step.

### 🔸 RawResponsesStreamEvent
`dataclass`
- Ye ek dataclass hai.
- Ye LLM (AI model) se directly aane wale **raw streaming events** ko represent karta hai.
- Upper jis task ki hum bat kar rahy choty choty tukron mein output aane ki, wo kam ye karte hay. ye class **raw streaming events** wrap karke aapko code mein accessible banati hai.

#### Attributes
- **data**
    - Isme AI ka actual output hota hay (token ya partial text).
    - Type → TResponseStreamEvent (OpenAI Responses API ka format).
    - Example: "H", "He", "Hel" ...

```python
data: TResponseStreamEvent
```

- **type**
    - Is attribute ki value hamesha fix hoti hay → "`raw_response_event`".
    Ye basically ek **label / tag** hay jo batata hay keh ye event kis type ka hay.
    - Iska kam sirf itna hay keh code ko samajh aye keh yeh event **raw_response_event** hai, na ke **tool call** ya **agent update**.

```python
type: Literal['raw_response_event'] = 'raw_response_event'
```

---

### 🔸 Run item events and agent events
- Ye high-level events hoty hain Yani (bary steps ki updates).
- Ye batate hain:
    - Jab AI koi **tool call** karta hay (tool_call_item).
    - Jab tool ka **result** ata hay (tool_call_output_item).
    - Jab AI ka **final message** taiyar ho jata hay (message_output_item).

### 🔸 Real World Analogy
Socho wahi cook ab har bari stage ki report deta hai:

- "Sabziyaan kaat li hain"
- "Ab main khana paka raha hoon"
- "Khana tayaar ho gaya hai"

Yeh choti choti detail ke bajaye bary milestones bataty hain.

### 🔸 RunItemStreamEvent
`dataclass`
- Ye class un events ko represent karti hai jo agent ke processing ke dauran bante hain.
- Jab LLM ka raw output agent ke logic se guzarta hay, us process mein alag alag chezen hoti hain (jaise message create hona, tool call hona, handoff hona etc.) → un sabko RunItemStreamEvent wrap karta hay.

#### Attribute
**name**
- Ye ek label / string hota hay jo batata hay keh kis type ka event bana aur kya process huwa.
- Iski posible values hain:

    - "`message_output_created`" → Jab ek naya message create huwa tab ye event create hota hay.
    - "`handoff_requested`" → Jab agent keh raha ho keh ab next agent handle kary.
    - "`handoff_occured`" → Jab actual handoff ho gaya ho.
    - "`tool_called`" → Jab agent ne koye tool call kiya.
    - "`tool_output`" → Jab tool ka result aa gaya.
    - "`reasoning_item_created`" → Jab agent ne reasoning step create kiya yani LLM ny thinking ki ho.
    - "`mcp_approval_requested`" → Jab agent kisi external approval ki request kary.
    - "mcp_list_tools" → Jab agent available tools ki list ly raha ho.

Matlab name basically event ki category hay.

**name**
- Ye actual RunItem object hota hai jo event ke andar create hua.
- Example:
    - Agar event "tool_called" hay → tw item mein tool ki detail hogi.
    - Agar event "message_output_created" hai → tw item mein naya message ka data hoga.

Matlab item = event ka actual content / data hota hay.

**type**
- Iski Type hamesha fix hoti hay → "run_item_stream_event".
- Iska kam sirf yeh batana hay ke ye event RunItemStreamEvent type ka hay (na ki raw response aur agent update).

<details>
<summary>Source code in <code>src/agents/stream_events.py</code></summary>

```python
@dataclass
class RunItemStreamEvent:
    """Streaming events that wrap a `RunItem`. As the agent processes the LLM response, it will
    generate these events for new messages, tool calls, tool outputs, handoffs, etc.
    """

    name: Literal[
        "message_output_created",
        "handoff_requested",
        # This is misspelled, but we can't change it because that would be a breaking change
        "handoff_occured",
        "tool_called",
        "tool_output",
        "reasoning_item_created",
        "mcp_approval_requested",
        "mcp_list_tools",
    ]
    """The name of the event."""

    item: RunItem
    """The item that was created."""

    type: Literal["run_item_stream_event"] = "run_item_stream_event"
```
</details>

### 🔸 AgentUpdatedStreamEvent
- Ye event tab trigger hota hai jab ek naya agent run karna start hota hay.
- Matlab agar apky workflow me ek agent sy dusre agent ko control handoff ho jaye (ya koi new agent load ho jaye), to ye event batata hay keh ab konsa agent active hay.

#### Attributes
- **`new_agent`**
    - Ye wo Agent object hay jo btata hay abhi naya agent run kar raha hay.
    - Isky andar us agent ke rules, instructions aur settings hoti hain.

- **`type`**
    - Iski type fix hoti hay: "`agent_updated_stream_event`"
    - iska sirf ye hay keh ap identify kar sako ke yeh event agent update ka hai, na ke raw response ya run item ka.

### 🔸 AgentUpdatedStreamEvent
Imagine karen apki ek project team hay.
- Pehly **Agent A** kam kar raha tha.
- Ab wo apna kam karky **Agent B** ko dedeta hay.
- Jesy hi Agent B active hota hay, apko ek notification ati hay: "Naya agent start ho gaya hay."
Wo notification `AgentUpdatedStreamEvent` Hay.

---

### Resources & Practical Demos

- [OpenAI Agent Streaming Method Colab Notebook](https://colab.research.google.com/drive/1pDhr55JygmQisEa9nsDFGuBenZVpwal8?usp=sharing) → Interactive example with code and usage.

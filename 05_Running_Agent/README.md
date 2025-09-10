## 🔹 Running Agents

Ap agent ko Runner class ki madad se chalate hain. Apke pass 3 options hoty hain Agent run karwany ky leye:
- `Runner.run()` → Ye asynchronous method hay jo agent ko run karta hai aur `RunResult` return karta hay.
- `Runner.run_sync()` → Ye synchronous method hai jo internally `.run()` ko call karta hai, matlab jab tak process complete na ho, control wapas nahi aata.
- `Runner.run_streamed()` → Ye asynchronous method hai jo streaming mode mein run karta hai. `RunResultStreaming` return karta hai. ye LLM ko streaming mode mein call karta hay. jesy hi events receive hota hay user ko foran bhj deta hay. (jesy ChatGPT mein hota hai).


```python
from agents import Agent, Runner

async def main():
    agent = Agent(name="Assistant", instructions="You are a helpful assistant")

    result = await Runner.run(agent, "Write a haiku about recursion in programming.")
    print(result.final_output)
    # Code within the code,
    # Functions calling themselves,
    # Infinite loop's dance
```

---

### 🔸 Runner
Runner ek class hay jismy 3 methods hoty hain jo agent ko alag–alag tarha se run karwaty hain.
- Ap jo bhi define karty ho (uska behavior, tools, output schema) wo sab `Runner` ke through execute hota hay


**Role:**
- Agent ke liye **event loop** manage karta hay.
- Input → Agent → LLM + Tools → Output ka pura flow handle karta hay.
- Agent ko ek tarah sy “driver” provide karta hay.

---

### 🔸 .run()
Workflow start hota hay ek Agent sy jo Ap `Runner.run()` mein dety hain wo Agent loop mein chalyga jab tak final output generate nh hota.

#### Loop ka Flow
1. **Agent Invocation:** Ap agent ko input ky sath call karty hain.
2. **Final Output Check:** Agar agent ny ek final output produce keya (yani agent kuch aesa produce kary jo agent.output_type ho) tw loop terminate ho jata hay.
3. **Handoff Case:** Agar agent ne handoff kiya (yani bola keh ab dusra agent handle kary) → loop new agent ke sath restart hota hay.
4. **Tool Calls Case:** Agar Agent ny tool calls keye un tools ko run karky phir loop continue hota hay.

#### Exception
Do cases mein agent exception raise kar sakta hay:
1. **MaxTurnsExceeded:** Agar `max_turns` exceed ho jaye tw `MaxTurnsExceeded` exception raise hoti hay.
2. **GuardrailTripwireTriggered:** Agar koi guadrials tripwire triger ho, tw `GuardrailTripwireTriggered` exception raise hoti hay.

> **Note:** Guardials sirf starting agent ky apply hongy.

#### Arguments
- `starting_agent`: Jis agent se run start karna hay.
- `input`: Agent ko deya gaya initial input (string ya list)
- `context`: Wo context jisme agent run hota hai.
- `max_turns`: Kitne dafa run karna hay ek turn ka matlab hay ek AI invocation (chahe usmein tool calls ho ya na ho).
- `hooks`: Ek object jo alag alag lifecycle events par callbacks receive karta hay.
- `run_config`: Pury agent run ke liye global settings.
- `previous_response_id`: Pichly response ka ID, agar ap OpenAI models Responses API ke zariye use kar rahe ho. Tw ye allow karta hay keh ap pichle turn ka input pass na karo.
- `conversation_id`: Conversation ka ID, Agar diya jaye, tw conversation use hoti hay items ko read aur write karne ke liye. Har agent ko ab tak ka conversation history access hota hai, aur uska output conversation mein likha jata hai. openai recommend karta hay keh sirf OpenAI models ke sath isko use karo; dusry model providers Conversation object mein write nahi karte, is liye partial conversations store ho sakti hain.
- `sessions`: Ek session object, agar ho.

#### Returns
Ek run result object jisme:
- sab inputs
- guardrail results
- aur last agent ka output hota hai

---

### 🔸 .run_sync()
Ye ek synchronous wrapper hay jo `Runner.run()` ko call karta hay. Matlab ap agent ko blocking mode me run kara sakty ho.
- Ye async environments (e.g. Jupyter Notebook asyncio loop) me work nahi karega.

#### Loop ka Flow
- Same as `run` method loop flow.

#### Exception
- Same as `run` method Exceptions.

#### Arguments
- Same as `run` method Arguments.

#### Returns
- Same as `run` method result.

### 🔸When to Use
- Jab ap normal Python script ya CLI tool likh rahe ho jahan async support nahi chahiye.
- Example: cron jobs, batch processing, ya simple blocking execution.

### 🔸When NOT to Use
- Jupyter Notebook, FastAPI, ya kisi bhi asyncio-based environment me nahi use karna chahiye.
- Wahan pe aapko `Runner.run()` hi use karna padega.

---

### 🔸 .run_streamed()
Ye method workflow ko streaming mode mein chalata hay.
Matlab Agent ka pura process ek hi bar result return karny ky bajaye, step-by-step **events** ko stream karta hay.
jo result object return hota hay usme `stream_events()` method hota hay jiska use karky ap sementic events ya partial output ko realtime mein receive kar sakty hain.
- stream() ek async iterator return karta hay, isleye isko **async for loop** ke sath use karna hota hay.

#### Loop ka Flow
- Same as `run` method loop flow.

#### Exception
- Same as `run` method Exceptions.

#### Arguments
- Same as `run` method Arguments.

#### Returns
Ek run result object jisme:
- Jisme run ka metadata hota hay.
- Aur ek streaming method hota hay jiska use karky ap realtime mein semantic events consume kar sakty hain (jesy model ka partial output, tool call/end events, etc).

---

### 🔸 RunResultBase
`dataclass`
Ye ek abstract base class hay (ABC) jo sabhi `RunResult` ky leye common structure banati hay. Matlab isky andar wo **fields** hain aur **methods** hain jo har result mein zaroor hona chahiye.

### 🔸 Instance Attributes

#### input
- Jo apny Agent ko deya koi sawal ya query.
- String ho sakta hay ya list of `TResponseInputItem`.
- Agar apny handsoff input filters use keye hain tw ye mutated bhi ho sakta hay (yani input mein changing).
- Type: `str | list[TResponseInputItem]`.

#### new_items
- Agent run ky time  generate hony waly events (like: tool calls, tool outputs, message, etc).
- Type: `list[RunItem]`.

#### raw_responses
- Jo LLM sy **raw responses** aye (unparsed, Jesy API ny bheja)
- Debugging ke leye useful hay.
- Type: `list[ModelResponse]`.

#### final_output
- Jo last agent ka final validated ouput aya ho.
- Type: `Any`.

#### input_guardrail_results
- Agar Input guardrials hay tw usky results (pass/fail ya koi trigger).
- Type: `list[InputGuardrailResult]`.

#### output_guardrail_results
- Agar Output guardrials hay tw usky results (pass/fail ya koi trigger).
- Type: `list[OutputGuardrailResult]`.

#### context_wrapper
- Context wrapper jo agent run ky state aur content ko manage karta hay.
- Type: `RunContextWrapper[Any]`.


### 🔸 Properties & Methods
#### last_agent
`abstractmethod property`
- Ye method batata hay keh konsa agent last tha.

#### last_response_id
- Last model ki response ki ID.
- Converation ka flow track karny ka shortcut.

#### final_output_as(cls, raise_if_incorrect_type=False)
Agar ap `raise_if_incorrect_type` ko `True` set karen tw final output deye gaye type ka na ho tw **TypeError** raise keya jayega.

#### to_input_list()
Ek nayi input list banata hay jo, merge karta hay orignal input aur new items ko.

---

### 🔸 RunResult
`dataclass`
Ye ek dataclass hay jo actual mein run ka result provide karta hay, Ye class `RunResultBase` sy inherit karky banaye gaye hay. Yani ismein wo sary attribute aur method hain jo `RunResultBase` base mein hain. Aur is mein isky ilawa bhi properties hain.

### 🔸Instance Attributes
#### last_agent
- Ye property batate hay keh last agent konsa run huwa.
- Agar workflow mein multiple Handsoffs huwy tw ye property batati hay keh last agent konsa run huwa.

#### input
- Inherited from RunResultBase.
- See the RunResultBase **input** Property for details.

#### new_items
- Inherited from RunResultBase.
- See the RunResultBase **new_items** Property for details.

#### raw_responses
- Inherited from RunResultBase.
- See the RunResultBase **raw_responses** Property for details.

#### final_output
- Inherited from RunResultBase.
- See the RunResultBase **final_output** Property for details.

#### input_guardrail_results
- Inherited from RunResultBase.
- See the RunResultBase **input_guardrail_results** Property for details.

#### output_guardrail_results
- Inherited from RunResultBase.
- See the RunResultBase **output_guardrail_results** Property for details.

#### context_wrapper
- Inherited from RunResultBase
- See the RunResultBase **context_wrapper** Property for details.

#### last_response_id
- Inherited from RunResultBase.
- See the RunResultBase **last_response_id** Property for details.


### 🔸 Properties & Methods
#### final_output_as(cls, raise_if_incorrect_type=False)
- Inherited from RunResultBase.
- See the RunResultBase **final_output_as** method for details.

#### to_input_list()
- Inherited from RunResultBase.
- See the RunResultBase **to_input_list()** method for details.

---

### 🔸 Simple Analogy
RunResultBase ek **blue print/skeleton** hay jisko RunResult inherit karky output banata hay


| Feature                                             | RunResultBase | RunResult   |
| --------------------------------------------------- | ------------- | ----------- |
| last\_agent                                         | abstract      | implemented |
| input / new\_items / raw\_responses / final\_output | defined       | inherited   |
| context\_wrapper                                    | defined       | inherited   |
| final\_output\_as / to\_input\_list                 | defined       | inherited   |
| last\_response\_id                                  | defined       | inherited   |


---

### 🔸RunResultStreaming  
`dataclass` Bases: `RunResultBase`  
`RunResultStreaming` extends **RunResultBase** Ye Runner ka result hota hay streaming mode mein kam karta hay. Ye agent ke run ke time step-by-step **semantic events** ko receive karny ki facility deta hai.

#### Exception
- Same as `run` method Exceptions.

### 🔸Instance Attributes
#### current_agent
- Wo agent jo currently run ho raha ho.
- Type: `current_agent: Agent[Any]`

#### current_turn
- Konsa turn abhi chal raha hay uska number.
- Type: `int`

#### max_turns
- Maximum turns limit jitni dafa agent run ho sakta hay. Agar exceed hui to `MaxTurnsExceeded` exception raise hogi.
- Type: `int`

#### final_output
- Agent ka final output. Ye None hota hay jab tak agent run complete na ho jaye.
- Type: `Any`

#### is_complete
- Ye check karta hay streaming khatam huwi ya nh
- Type: `bool (default False)`

#### input
- Inherited from RunResultBase.
- See the RunResultBase **input** Property for details.

#### new_items
- Inherited from RunResultBase.
- See the RunResultBase **new_items** Property for details.

#### raw_responses
- Inherited from RunResultBase.
- See the RunResultBase **raw_responses** Property for details.

#### input_guardrail_results
- Inherited from RunResultBase.
- See the RunResultBase **input_guardrail_results** Property for details.

#### output_guardrail_results
- Inherited from RunResultBase.
- See the RunResultBase **output_guardrail_results** Property for details.

#### context_wrapper
- Inherited from RunResultBase.
- See the RunResultBase **context_wrapper** Property for details.

### 🔸 Properties & Methods
#### last_agent
- Inherited from RunResultBase.
- See the RunResultBase **last_agent** Property for details.

#### last_response_id
- Inherited from RunResultBase.
- See the RunResultBase **last_response_id** Property for details.

#### cancel()
- Ye method tab use hota hay jab apny ek streaming agent run chalaya huwa hay `.run_streamed()` sy, aur ap chahty ho ke us process ko bech me hi stop kar do.
- Type: `() -> None`

**Use Case Example:** User ek long-running streaming process chala raha ho jo continuously events produce kar raha ho. Agar user manually stop button press kary tw → ap cancel() method ko call kar ky run ko terminate kar sakty ho.

#### stream_events (async)
- Ye naye items ke deltas ko stream karta hay jab wo generate hoty hain.
- Ye OpenAI Responses API events format ka use karta hay. Har event ka ek type aur data hota hay jo event ka type batata hai, aur us event ka data.
- **Exceptions**:
    - MaxTurnsExceeded → agar turns limit cross ho jaye.
    - GuardrailTripwireTriggered → agar guardrail trigger ho jaye.
- Type: `AsyncIterator[StreamEvent]`

#### final_output_as(cls, raise_if_incorrect_type=False)
- Inherited from RunResultBase.
- See the RunResultBase **final_output_as** method for details.

#### to_input_list()
- Inherited from RunResultBase.
- See the RunResultBase **to_input_list()** method for details.

---

### 🔸 The agent Loop
jab ap `Runner` mein `run` method use karty ho, tw ap 2 arugument pass karty hain, **starting agent** aur **input**, input ya tw ek string hoga (jesy user message) ya phr item ki list jo OpenAI Responses API ke items hoty hain,

Phir Runner ek loop chalata hay:

1. Hum current agent ky leye LLM ko call karty hain current input ky sath.
2. LLM apna output generate karta hay.
    - Agar LLM `final_output` return karta hay. tw loop stop hojata hay aur result return karta hay.
    - Agar LLM Handoff karta hay, tw current agent aur input update hoty hain, aur loop again run hota hay.
    - Agar LLM tool calls karta hay, tw wo tool calls run hoty hain result input mein append hota hay aur phr dubara sy loop chalta hay.
3. Agar hum `max_turns` set karty hain aur wo exceed ho jata hai, to `MaxTurnsExceeded` exception milti hai.

**Notes:** LLM ka output tab "final_output" mana jata hay jab wo desire type ka text output produce karta hay, aur koi tool call nh hota.

---

### 🔸 Run_Config
`run_config` parameter apko kuch global setting configure karny deta hay agent run ky leye:

- **`model`:** Apko ye global model set karny ki permission deta hay chahy har agent ka apna model kyun na ho.

- **`model_provider`:** Ek model provider hay jo model names ko look up karta hay, jo default tor per OpenAI hota hay.

- **`model_settings`:** Ye agent ki specific-settings override karta hay. For example, ap set kar sakty hain global `temperature` or `top_p`.

**Temperature:**
- Ye control karta hay randomness ya creativity ko.
- agar temperature low hay (jesy 0.1) tw output zyada determeinistic (predictable) hoga yani exact jawab hoga
- agar temperature high hay (jesy 0.8 ya 1.0) tw output random aur creative hoga, yani model naya ya unpredictable ans dy sakta hay

**top_p:**
- ye ek probability threshold (had) hota hay.
- Model sirf un words mein sy choose karta jinka cumulative (majuma) probability (imkaan) `top_p` (jesy 0.9) ke andar aati hay.
- Is sy model ka zyada focus rehta hay, aur extreme rear words exclude ho jaty hain.
- top_p ko low rakhny sy model ka output controlled hota hay, aur high rakhny sy thora diverse.

**`input_guardrails, output_guardrails`:** wo rules ya filter hoty hain jo ap har run mein input ya output per lagaty hain taky kuch specific chezen control ho saky.

- **`handoff_input_filter`:** Ek global input filter hay jo sary agent per apply hota hay, Lekin sirf tab, jab handoff ke pass phely sy koi filter na ho. Ye input filter apko input edit karny ki ijazat deta hay mean keh koi aese specific word ya language dusry agent tak na pochy jo ap nh chahty.

**`tracing_disabled`:** Ye pury run ky leye tracing disbale kardega.

**`trace_include_sensitive_data`:** Ye decide karta hay ke tracing (jo run ky time track karta hay) mein sensitive data shamil hoga ya nh

**Sensitive data jesy:**
- LLM ke inputs aur outputs (jo user ne likha ya model ne jawab diya)
- Tool calls ke inputs aur outputs (jo extra tools use hote hain run mein)
Agar ye setting **True** hay tw traces mein sensitive data bhi record hoga.
Agar ye setting **False** hogi, tw sensitive data trace sy exclude kardeya jayega taky privacy aur safety barh jaye.

**`workflow_name, trace_id, group_id`:** ye tracing ky leye important identifiers hain jo run ko uniquely track karny aur organize karny mein help karta hay.

- **workflow_name:** Ye ek naam hota hay jo ap tracing workflow ko dety hain, isy set karna zaroori hota hay taky apko pata ho ye tracing kis workflow ya process ki hay.

- **trace_id:** Ye har run ky leye unique ID hote hay jo specific run ko identify karte hay.

- **group_id:** Ye optional hota hay aur multiple runs ke traces ko ek group mein link karny ky leye hota hay. Matlab agar apky pass zyada runs hain jo related hain tw unhy group_id sy ek sath track kar sakty hain.

**`trace_metadata`:** Sary traces mein shamil karne ke liye metadata.
- wo extra information hoti hay jo ap har tracing record ky sath add karty hain.
- Iska matlab: Jab bhi tracing data save hota hai (jaise run ke details), aap kuch additional details (metadata) bhej sakte hain jo har trace mein shamil ho.

---

### Resources & Practical Demos

- [OpenAI Agent Running Method Colab Notebook](https://colab.research.google.com/drive/1vMRWG0cT0X0q5o6roez40Gr2UbeAFXEb?usp=sharing) → Interactive example with code and usage.

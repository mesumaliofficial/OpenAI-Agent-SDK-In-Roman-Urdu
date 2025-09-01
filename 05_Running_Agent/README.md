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
3. **Handoff Case:** Agar agent ne handoff kiya (yani bola keh ab dusra agent handle kary) → loop new agent ke sath restart hota hai.
4. **Tool Calls Case:** Agar Agent ny tool calls keye un tools ko run karky phir loop continue hota hay.

#### Exception
Do cases mein agent exception raise kar sakta hay:
1. **MaxTurnsExceeded:** Agar `max_turns` exceed ho jaye tw `MaxTurnsExceeded` exception raise hoti hay.
2. **GuardrailTripwireTriggered:** Agar koi guadrials tripwire triger ho, tw `GuardrailTripwireTriggered` exception raise hoti hay.

> **Note:** Guardials sirf starting agent ky apply hongy.

#### Arguments
- `starting_agent`: Jis agent se run start karna hay.
- `input`: Agent ko diya gaya initial input (string ya list)
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
Ye ek synchronous wrapper hay jo Runner.run() ko call karta hay. Matlab ap agent ko blocking mode me run kara sakty ho.
- Ye async environments (e.g. Jupyter Notebook, FastAPI, asyncio loop) me work nahi karega.

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
- Wahan pe aapko Runner.run() hi use karna padega.

---

### 🔸 .run_streamed()
Ye method workdlow ko streaming mode mein chalata hay.  
Matlab Agent ka pura process ek hi bar result return karny ky bajaye, step-by-step **events** ko stream karta hay.  
jo result object return hota hay usky andar ek method hota hay jis ka use karky ap sementic events ko ly sakty hain

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
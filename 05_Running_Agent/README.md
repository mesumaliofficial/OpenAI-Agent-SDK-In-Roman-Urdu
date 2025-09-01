## 🔹 Running Agents

Ap agent ko Runner class ki madad se chalate hain. Apke pass 3 options hoty hain Agent run karwany ky leye:
- `Runner.run()` → Ye asynchronous method hay jo agent ko run karta hai aur `RunResult` return karta hay. 
- `Runner.run_sync()` → Ye synchronous method hai jo internally `.run()` ko call karta hai, matlab jab tak process complete na ho, control wapas nahi aata.
- `Runner.run_streamed()` → Ye asynchronous method hai jo streaming mode mein run karta hai. `RunResultStreaming` return karta hai. ye LLM ko streaming mode mein call karta hay. jesy hi events receive hota hay user ko foran bhj deta hay. (jaise ChatGPT mein hota hai).

**Notes:**
- **Asynchronous** (async) methods mein await use karty hain, jissy ap dusry kam bhi sath sath kar sakty hain.
- **Synchronous** Synchronous methods mein ap ko wait karna parta hai jab tak kam complete na ho jaye, tab tak koi aur kam nahi kar sakty.

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
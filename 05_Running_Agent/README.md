## 🔹 Running Agents

Ap agent ko Runner class ki madad se chalate hain. Apke pas 3 options hoty hain Agent run karwany ky leye:
- `Runner.run()` → Ye asynchronous method hai jo agent ko run karta hai aur RunResult return karta hai. 
- `Runner.run_sync()` → Ye synchronous method hai jo internally `.run()` ko call karta hai, matlab jab tak process complete na ho, control wapas nahi aata.
- `Runner.run_streamed()` → Ye asynchronous method hai jo streaming mode mein run karta hai. `RunResultStreaming` return karta hai. ye LLM ko streaming mode mein call karta hay jesy hi events receive hota hay user ko foran bhj deta hay. (jaise ChatGPT mein hota hai).

**Notes:**
- **Asynchronous** (async) methods mein await lagate hain, jisse ap dusra kam bhi kar sakta hai jab tak agent ka response aa raha ho.
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
- a. Agar LLM `final_output` return karta hay. tw loop stop hojata hay aur result return hojata hay.
- b. Agar LLM Handoff karta hay, tw current agent aur input update hoty hain, aur loop again run hota hay.
- c. Agar LLM tool calls karta hay, tw wo tool calls run hoty hain result input mein append hota hay aur phr dubara sy loop chalta hay.
3. Agar hum `max_turns` set karty hain aur wo exceed ho jata hai, to `MaxTurnsExceeded` exception milti hai.

**Notes:** LLM ka output tab "final_output" mana jata hay jab wo desire type ka text output produce karta hay, aur koi tool call nh hota.

### 🔸 Streaming
Streaming apko ye bhi allow karte hay ap LLM run ky time streaming events receive kar sakty hain. jesy hi stream complete hote hay, `RunResultStreaming` mein run ki sari information hoti hay, jis mein sary naye outputs hoty hain, Ap `.stream_events()` call karky streaming events ly sakty hain, Streaming matlab choty choty chunks mein output lena.


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


### 🔸 Conversations/chat threads
Kisi bhi `run` methods ky zariye, ek ya ek sy zyada agents run ho sakty hain, (aur is tarha ek ya ek sy zyada LLM bhi calls ho sakti hain), For Example:

1. **User turn:** user text enter karta hay.
2. **Runner run:** Phely agent LLM calls karta hay, tools chalata hay, phr dusry agent ko handoff karta hay. phr dusra agent mazeed tools chalata hay aur phr output produce karta hay.

Agent run ka matlab hai ek poora round jisme agent apka sawal leta hai, kam karta hai, phr jawab deta hai. End mein ap decide karte ho ke user ko sary steps ka result dikhana hai ya sirf final output. Agar user dobara sawal kare to naya round start hota hai, aur ap phir run method calls ho sakty hain.


### 🔸 Manual conversation management
Ap manually conversation history manage kar sakty hain, isky leye ap `RunResultBase.to_input_list()` method ka use karty hain, jo apko pichle run ka data data aise format mein deta hay keh ap usi ko next turn input ky tor per agent ko feed kar saken.  
Is tarha conversation jahan sy chori thi, wahan sy dubara shuru ki ja sakti hay.

```python
async def main():
    agent = Agent(name="Assistant", instructions="Reply very concisely.")

    thread_id = "thread_123"  # Example thread ID
    with trace(workflow_name="Conversation", group_id=thread_id):
        # First turn
        result = await Runner.run(agent, "What city is the Golden Gate Bridge in?")
        print(result.final_output)
        # San Francisco

        # Second turn
        new_input = result.to_input_list() + [{"role": "user", "content": "What state is it in?"}]
        result = await Runner.run(agent, new_input)
        print(result.final_output)
        # California
```


### 🔸 Automatic conversation management with Sessions
Agar ap simple tareqy sy history manage karna chahty hain, tw Sessions ka use kar sakty hain, Ye automatically conversation history ko handle karta hay, bina manually `.to_input_list():` ka use kary. 

```python
from agents import Agent, Runner, SQLiteSession

async def main():
    agent = Agent(name="Assistant", instructions="Reply very concisely.")

    # Create session instance
    session = SQLiteSession("conversation_123")

    with trace(workflow_name="Conversation", group_id=thread_id):
        # First turn
        result = await Runner.run(agent, "What city is the Golden Gate Bridge in?", session=session)
        print(result.final_output)
        # San Francisco

        # Second turn - agent automatically remembers previous context
        result = await Runner.run(agent, "What state is it in?", session=session)
        print(result.final_output)
        # California
```

**Sessions automatically:**
- Har run sy phely conversation automatically retrieve karta hay.
- Har run ky bad naye messages store karta hay.
- Different seesions IDs ke leye alag conversation maintain hote hay.


### 🔸 Long running agents & human-in-the-loop
Ap Agents SDK ko [Temporal](https://temporal.io/) integration ky sath use karky durable, long-time aur long running workflows run kar sakty hain, is mein human-in-the-loop tasks bhi shamil hain,

**Temporal kya hay ?**  
Temporal ek workflow orchestration platform hay. Iska kam ye hota hay keh jab apky pass long-running (zyada time tak chalny waly) tasks hote hain jesy:
- File processing jo hours/days ly sakti hay.
- Order processing pipelines.
- AI agents jo multiple steps mein kam karty hain
Temporal ensure karta hay keh workflow fail na ho, chahy system crash hojaye ya fail hojaye, yeh **retry,** **state save** aur **resume** ka system deta hay.

**human-in-the-loop(HITL) kya hay ?**  
human-in-the-loop ka matlab hay keh workflow ky bech mein human ka decission lena ya input lena agar zaroori ho tw. For example:
- AI koi report banata hay.
- Workflow rukh jata hay aur Human (yani user) sy puchta hay, Approve karun ye edit karu ?
- Approval milny ky bad workflow agy chalta hay.

**Note:** Agents SDK long-term use ky leye nh hay, agar long running use karna hay tw temporal use karna hoga.


### 🔸 Exceptions
SDK kuch specific cases mein expection leta hay, Ye exceptions `agents.exceptions` module mein milti hain.

**`AgentsException`:** Ye base class hay jo SDK ky andar leye jany waly all exceptions ky leye hay, ye ek generic type hay jissy all exceptions derive (hasil karna) kiye jaty hain.

**`MaxTurnsExceeded`:** Ye error tab show hota hay jab `max_turns` limit `Runner.run`, `Runner.run_sync` ya `Runner.streamed` ko diya gaya ho aur wo end hogaye ho.

**`ModelBehaviorError`:** Ye exception tab show hote hay jab underlying model (LLM) unexpected ya invalid output deta hay, is mein include hay:

- Malformed JSON: jab model tool calls ky leye apna directly output malformed JSON structure deta hay, khas tor per agar koi specific output define ho... Matlab, jab LLM apny ouput ya tool karty waqt JSON data bhjta hay, Wo bilkul sahi aur complete hona chahiye kuch missing hoga tw `ModelBehaviorError` ayega.

- Unexpected tool-related failures: jab model ko expected tareqy sy use karny mein fail hojata hay 

**`UserError`:** Ye exception tab leta hay jab ap (jo SDK use karty huwy code likh rahy hain) koi galti karty hain, Ye usually Wrong code implemnetation, invalid configuration, ya SDK ki api ka galat use ki wajah sy hota hay.

**`InputGuardrailTripwireTriggered, OutputGuardrailTripwireTriggered`:** Ye exception tab leta hay jab input guardrial ya output guardrial ke conditions puri hoti hain, input guardrial any waly message ko process karny sy phely check karta hay, aur output guardrials agent ky final response sy phely check karta hay.

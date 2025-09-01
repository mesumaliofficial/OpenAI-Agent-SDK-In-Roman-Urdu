## 🔹Context Management
Context ek overloaded term hay. (yani ye lafz diffrent jaga aur diffrent purposes ke liye use hota hai.). Context do main types ka hota hay:

1\- **Local Context:** Ye wo data aur dependencies hain jo sirf apke code ke liye available hoti hain. Iska use ap:

- Tool functions
- `on_handoff`
- Lifecycle hooks

wagera mein karty hain.

**Important:** Ye data direct LLm ko nh jata. agar apny ye tool ky zariye pass keya ho tw LLM ko ye srf tool context mein hi milyga.

2\- **LLM Context:** Ye wo context hay jo LLM response generate karty waqt dekhta hay:

- User ka message
- Conversation history
- Instruction prompt

LLM isi data ko use karky apna response generate karta hay.


### 🔸Context ka purpose ?
OpenAI SDK mein context ka role agent ko stateful banane ke liye hota hai. Iska matlab hai ke agent apni purani baatein ya information yaad rakhta hai, taki har dafa se shuru se batana na pary agent ko.


### 🔸 Local Context
Ye `RunContextWrapper` class aur isky andar mojud context property ky zariye represent keya jata hay. is ka kam karny ka tareqa ye hay:

- Ap koi bhi python object bana sakty hain jo ap chahen, common tareqa ye hay dataclass ya pydantic ka use karen.

- Phir ap wo method run methods mein pass kardety hain jesy keh `Runner.run(...,context=whatever)`.

- Apky sary tools calls, lifecycle hooks wagera ko ek wrapper object milyga, `RunContextWrapper[T]`, jahan `T` ap ky context object ka type hota hay jo ap `wrapper.context` ky zariye access kar sakty hain.

**Important:** Sab sy important bat kisi bhi agent run ky leye har agent, tool function, lifecycle wagera ko ek hi type ka context use karna chahiye.

Ap context ka use in chezo ky leye kar sakty hain:

- **Contextual Data:** apky run ky leye (jesy username/uid ya user ky bary mein dusri information).

- **Dependencies:** jesy logger objects, data fetcher services, ya koi aur external resources jiski zaroorat apko kam ky waqt pary.

- **Helper Functions:** Wo functions jo ap apny tools, hooks wagera ya wo data jo ap code mein bar bar use karna chahty ho.

**Simple Example for real world analogy:**  
Maan lo LLM ne bola: "User ka current balance check karo".

- LLM sirf instruction de raha hai.
- Apka Local Context me `db_connection` aur `user_id` para hai.
- Tool function in dono ka use karke database se balance fetch karta hai.
- LLM ko sirf final result milta hai: "Your balance is $150"

**Note:** Context object LLM ko nh bheja jata. ye sirf ek local object hota hay jo ap read/write kar sakty hain aur is per methods call kar sakty hain.

```python
import asyncio
from dataclasses import dataclass

from agents import Agent, RunContextWrapper, Runner, function_tool

@dataclass
class UserInfo:  
    name: str
    uid: int

@function_tool
async def fetch_user_age(wrapper: RunContextWrapper[UserInfo]) -> str:  
    """Fetch the age of the user. Call this function to get user's age information."""
    return f"The user {wrapper.context.name} is 47 years old"

async def main():
    user_info = UserInfo(name="John", uid=123)

    agent = Agent[UserInfo](  
        name="Assistant",
        tools=[fetch_user_age],
    )

    result = await Runner.run(  
        starting_agent=agent,
        input="What is the age of the user?",
        context=user_info,
    )

    print(result.final_output)  
    # The user John is 47 years old.

if __name__ == "__main__":
    asyncio.run(main())
```


### 🔸 LLM Context
Jab LLM ko call keya jata hay tw wo srf conversation history ka data dekh sakta hay, Agar ap LLM ko koi naya data dena chahty hain tw ap ko usy aesy provide karna hoga jo usy us history mein available banady is ky kuch tareqy ye hain:

1. **Instructions:** Ap isy Agent instruction mein add kar sakty hain jeesy system promt ya developer message bhi kaha jata hay. system prompt static string bhi ho sakte hay aur dynamic functions bhi jo context receive karky ek string output dety hain. ye usually is leye use keya jata hay jo information hamesha kam ki ho jesy user ka naam ya current date.

2. **Input:** Isy ap `Runner.run` ky andar ap input mein dy sakty hain. ye instructions tactic ki tarha hota hay lekin is tareqy sy ap aesy message rakh sakty hain jo command chain mein nechy hota hain. Yani LLM usko dekh sakta hay, lekin priority level instruction sy kum hota hay.

3. **Tools Functions:** Isy ap tool function ky zariye bhi expose kar sakty hain, iska faida ye hota hay keh LLM jab chahy tab wo data ly sakta hay yani on-demand context.

Is case me:
- Ap ek tool banaty hain jo required data fetch karta hai (jesy database se, API se, ya Local Context se).
- LLM apni zaroorat ke mutabiq us tool ko call karta hai.
- Tool ka output conversation history me add ho jata hai, aur LLM usko use karke agla response generate karta hai.

4. **Retrieval ya Websearch:** Aap retrieval ya web search tools ka use karke LLM ko relevant data de sakte ho:

- **Retrieval:** File system, database ya knowledge base ya revelant data get karna ho.
- **Web Search:** Internet se latest information fetch karna.

is ka faida ye hota hay LLM ka jawab "grounded" hota hay, yani wo apni bat ko revelant aur sahi context ky sath deta hay, guessing kum hoti hay

---

### Resources & Practical Demos

- [OpenAI Agent Tools Colab Notebook](https://colab.research.google.com/drive/1SSi-uJsIXl59FoOWZnUqsD_AmqgcdoQe?usp=sharing) → Interactive example with code and usage.

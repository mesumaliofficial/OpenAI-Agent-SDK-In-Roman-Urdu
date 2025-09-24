## 🔹 Agents → Output Types

### 🔸What is Output Types?

Agent output types wo hoti hain jo batati hain keh agent ka final output kis tarha ka hoga. By default, agent plain text yani (str) output deta hay. Lekin Kabhi-Kabhi hume ek special organize format mein data chahiye hota hay.

**For Example:**  
Hume ek email sy sary phone numbers nikalny hain, tw plain text (str) mein number ko find karna mushkil hoga. Isky bajaye, ager Agent output ko ek **list of string** (`list(str)`) mein dy, tw kam asan hojata hay.

---

### 🔸Structured Outputs

Jab hum `output_type` parameter ka use karty hain, tw **Agent Structured output** mode mein chala jata hay, iska matlab hay wo plain text ky bajaye **JSON** object return karyga.

```python
from pydantic import BaseModel
from google.generativeai.agent import Agent

# Yeh humne ek Pydantic model banaya hai. Yeh humara blueprint hay.
class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

agent = Agent(
    name="Calendar extractor",
    instructions="Extract calendar events from text",
    output_type=CalendarEvent, # Yahan humne blueprint de diya
)
```

Is code mein, jab hum Agent ko is jesa "Friday ko meeting hay 3 bjy Haider aur Ali ky sath" koi text dengy, tw wo is text sy data nikal kar `CalendarEvent` ky format mein dega. Yani ek JSON object jis mein `name`, `date` aur `participants` fields honge.

---

### 🔸What is AgentOutputSchemaBase?

`AgentOutputSchemaBase` ek **abstract base class** hay. Abstract class ka matlab hay hum isko directly use nahi kar sakty, bulky iski properties dusri class use karte hain. Ye ek tarha sy **contract** hay.

Is contract ke andar kuch rules (abstract method) hain jesy:

- `is_plain_text() -> bool:` ye batata hay keh output plain text hoga ya nh

- `name() -> str:` Output type ka naam batata hay.

- `json_schema() -> dict[str, Any]:` Output type ka JSON schema (blueprint) batata hai.

- `is_strict_json_schema() -> bool:` Ye batata hay keh Agent is time **strict mode** mein hay ya nahi. Jab yeh `True` return karega, iska matlab hai ke Agent ko **strict rules** follow karny hain.

- `validate_json(json_str: str) -> Any:` Ye check karta hay keh LLM ny jo JSON diya wo blueprint ky according hay ya nahi. agar nahi hay tw error dega. 

Ye sab properties Agent SDK ke andar automatically handle hote hain, jab hum output_type mein koi Pydantic model ya koi aur type pass karty hain.

---

### 🔸Konsi Output Types Use Kar Sakte Hain?

Agent SDK bohut flxible hay, Ap `output_type` mein bohut si tarha ki python types dy sakty hain, jo pydantic `TypeAdapter` mein wrap ho saken

1. **Pydantic Models (BaseModel):** Ye sab sy best aur common tareqa hay **structured data** ky leye Yeh validation aur schema generation provide karta hay.

2. **Dataclasses:** Agar hum pydantic use nahi karna chaht, tw python ki built-in `dataclass` bhi use kar sakty hain.

3. `TypedDict:` Ye simple dictionaries ke leye type define karny mein kam ate hay.

4. **Built-in types:** Simple cases mein ap Python ki built-in types bhi dy sakty hain, jesy `list[str]`, `dict[str, int]`, ya `int`.

---

### 🔸What is AgentOutputSchema?
Ek worker hay jo `AgentOutputSchemaBase` ky sary rules ko implement karta hay. Iska kam apky deye huwy **output type** ko leker LLM ke leye ek actionable blueprint banana aur phir us blueprint ky according LLM ky output ko validate karna hay.

Sochen, Jab ap agent ko `output_type=CalendarEvent` hain tw, backend per ye hota hay.

1. Ek `AgentOutputSchema` ka object banta hai. Is object ko `CalendarEvent` class de di jati hai.

2. Ye object `json_schema()` method ko call karta hay. Isse `CalendarEvent` ka JSON blueprint ban jata hai. Yeh blueprint LLM ko batata hay keh output mein name (string), date (string), aur participants (list of strings) hona chahiye.

3. LLM is blueprint ke according JSON banata hay.

4. AgentOutputSchema phir validate_json() method ka use karta hay. Ye method LLM se aaye huwy JSON ko check karta hay. Agar JSON sahi hay, tw ye usy Python object (`CalendarEvent`) mein convert kar deta hay. Agar mistake hay, tw ye ek error dega.

---

### 🔸Resources & Practical Demos

- [OpenAI Agent Output Type/Structured output Colab Notebook](https://colab.research.google.com/drive/11svP2C0G6--FNdxFzD2QVWtIfg_5ZGmM?usp=sharing) → Interactive example with code and usage.

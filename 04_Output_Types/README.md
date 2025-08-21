## 🔹 Agents → Output Types


### 🔸Output Types
- By default, agent plain text yani (str) output deta hai.
- Lekin agar aap chahen ke agent kisi specific type ka output de, to aap `output_type` parameter ka use karke output ki type define kar sakte hain.
- Ek common choice hoti hai ke `Pydantic object` ka use kiya jaye.
- Iske ilawa, agent kisi bhi type ka object support karta hai jo Pydantic TypeAdapter mein wrap ho sakta ho. jesy: dataclasses, lists, TypedDict, etc.

```bash 
from pydantic import BaseModel
from agents import Agent


class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

agent = Agent(
    name="Calendar extractor",
    instructions="Extract calendar events from text",
    output_type=CalendarEvent,
)
```
**Note:** Jab output type ka use karty hain, tw yeh model ko batata hay ke wo regular plain text ky bajaye [structured output](https://platform.openai.com/docs/guides/structured-outputs) ka use kare.


### 🔸What is Structured Output ?
Structure Output ek feature hay jis ka matlab hay ap model ko force karty ho keh uska jawab **ek spcific struture/schema** ky according generate ho jesy: (JSON object, dictionary, list, etc). is tarha apko ye fiqar nh hoti keh model koi required key chordy ya phir koi invalid enum value hallunicate kardy.

**Usecase:** Ye feature APIs, agents, aur applications banate waqt use hota hai jahan apko reliable aur valid data chahiye hota hai.

### 🔸Simple Analogy: The Order Form  
**Without Structured Output (Messy):**  
- User: "Mujhy Newyork ky weather ky bary mein btao"
- Agent: "Aj kafi acha mausam hay, shayad 75 degrees aur dhoop hai New York mein"
- User: "Main temperature kaise nikalun? Kaunsa shehar hai? Ye Fahrenheit hai ya Celsius?"

**With Structured Output (Perfect):**  
- User: "Mujhy Newyork ky wather ky bary mein btao"
- Agent: `{"location": "New York", "temperature_c": 24, "summary": "sunny"}`
- User: "Perfect! Ye data ab mein directly apni app mein use kar sakta hun!"

---

### 🔸The Core Concept
**Case 1: Agent returns plain text (hard to parse)**
```python
# Agent returns free-form text
result = Runner.run_sync(agent, "What's the weather in Karachi?")
print(result.final_output)
# Example Output:
# "The weather in Karachi is quite warm today, around 30 degrees Celsius with clear skies."

# ❌ Problem:
# Output text hai → location, temperature ya summary ko extract kesy karen!
```

**Case 2: Agent returns structured data (easy to use)**
```python
# Agent returns structured data
class WeatherAnswer(BaseModel):
    location: str
    temperature_c: float
    summary: str

agent = Agent(output_type=WeatherAnswer)
result = Runner.run_sync(agent, "What's the weather in Karachi?")

# Perfect! Ab output structured format mein milta hai → directly access kar sakte ho.:
print(result.final_output.location)      # "Karachi"
print(result.final_output.temperature_c) # 30.0
print(result.final_output.summary)       # "clear skies"
```

---

### 🔸How Structured Output Works ?
Structured output use karny par agent backend mein yeh kam karta hay:
1. **Schema samajhta hai:** Agent ko Exactly malum hota hay keh konsi fields mein kya fill karna hay.
2. **Data validate karta hai:** Ensure karta hay jo bhi apny fields banaye hain wo sary present ho.
3. **Correct format deta hai:** Data usi structure mein return hota hay jo apny specify keya ho.
4. **Type check karta hai:** Jo types apne apny object mein dee hay fully ensure karta hay usi type ka ho.

```python
# What happens automatically:
class PersonInfo(BaseModel):
    name: str          # Must be text
    age: int           # Must be a whole number
    email: str         # Must be text
    is_student: bool   # Must be True/False

# Agent MUST return data in this exact format
# If it tries to return invalid data, it gets corrected automatically!
```

### 🔸Pydantic Models → Your Data Blueprint
| **Component**        | **Kya karta hai**                   | **Example**                               |
| -------------------- | ----------------------------------- | ----------------------------------------- |
| **Class Definition** | Data structure define karta hai     | `class WeatherInfo(BaseModel):`           |
| **Field Types**      | Batata hai field ka type kya hoga   | `temperature: float`                      |
| **Required Fields**  | Ye fields lazmi hone chahiye        | By default, sab fields required hoti hain |
| **Optional Fields**  | Ye fields missing bhi ho sakti hain | `rainfall: Optional[float] = None`        |


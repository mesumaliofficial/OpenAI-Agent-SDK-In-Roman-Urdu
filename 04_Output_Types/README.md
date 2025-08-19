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


### 🔸Structured model outputs
Ensure karo ky model ky text responses us JSON schema ky according ho jo ap define karty hain.  

JSON world mein sabsy zyada use hony wala format mein sy ek hay jo applications data exchange karny mein use hota hay.

### 🔸What is Structured Output ?
Structure Output ek feature hay jis ka matlab hay ap model ko force karty ho keh uska jawab **ek spcific struture/schema** ky according generate ho jesy: (JSON object, dictionary, list, etc). is tarha apko ye fiqar nh hoti keh model koi required key chordy ya phir koi invalid enum value hallunicate kardy.

se aap model ko restrict kar sakte ho ke har dafa output predictable aur parsable format mein aaye.

**Usecase:** Ye feature APIs, agents, aur applications banate waqt use hota hai jahan apko reliable aur valid data chahiye hota hai.

### 🔸Simple Analogy: The Order Form  
**Without Structured Output (Messy):**  
- User: "Mujhy Newyork ky wather ky bary mein btao"
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


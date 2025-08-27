## 🔹 Openai SDK Api Refrences → Agent Output

### 🔸AgentOutputSchemaBase
Bases: `ABC`
- Ye ek base class hai jo agent ke output ko control karta hai.
- Jab agent koi response generate karta hai (LLM se), to ye class ensure karti hai:
    - **Schema Capture:** Default format kya hoga (e.g., string, dict, list).
    - **Validation:** Kya LLM ka response valid JSON hai aur schema ke according hai ya nahi.
    - **Parsing:** Agar valid hay, tw isy Python object (dict, list, model) mein convert kar deta hai.


### 🔸Behind the Scenes Flow (Easy Words)
1. **User Input:** "100 USD ko PKR mein convert karo"
2. **Agent Thinking:** LLM answer generate karta hai → {"usd": 100, "pkr": 28000}

3. *AgentOutputSchemaBase ka Kaam:*
    - `is_plain_text()` → ye determine karta hay kya ye output text hai ya JSON object.
    - Agar `False` hay tw `json_schema()` sy schema retrive karo aur validate json sy validate karo keh keys/ values format sahi hay ya nh.
    - Agar sahi hai, convert karky Python object return karo.
4. **Final Result:** Agent user ko safe, structured output deta hai.

<details>
<summary><b>Source code in </b>src/agents/agent_output.py</summary>

```python
class AgentOutputSchemaBase(abc.ABC):
    """An object that captures the JSON schema of the output, as well as validating/parsing JSON
    produced by the LLM into the output type.
    """

    @abc.abstractmethod
    def is_plain_text(self) -> bool:
        """Whether the output type is plain text (versus a JSON object)."""
        pass

    @abc.abstractmethod
    def name(self) -> str:
        """The name of the output type."""
        pass

    @abc.abstractmethod
    def json_schema(self) -> dict[str, Any]:
        """Returns the JSON schema of the output. Will only be called if the output type is not
        plain text.
        """
        pass

    @abc.abstractmethod
    def is_strict_json_schema(self) -> bool:
        """Whether the JSON schema is in strict mode. Strict mode constrains the JSON schema
        features, but guarantees valid JSON. See here for details:
        https://platform.openai.com/docs/guides/structured-outputs#supported-schemas
        """
        pass

    @abc.abstractmethod
    def validate_json(self, json_str: str) -> Any:
        """Validate a JSON string against the output type. You must return the validated object,
        or raise a `ModelBehaviorError` if the JSON is invalid.
        """
        pass
```
</details>

---

### 🔸is_plain_text
`abstractmethod`  
Ye ek method hay jo har output schema class ko implement karna parte hay (kyunki abstractmethod hai).  
Iska kam ye check karna hay keh Agent ka simple **plain text** hay ya ek **structured json object** hay


### 🔸Behind the Scenes Flow
1. Jab koi Agent output generate karta hay tw wo ya tw:
    - **Plain Text:** Sirf ek string (jesy "Exchange rate is 280 PKR").
    - **JSON Object:** structured data (jesy { "currency": "PKR", "amount": 280 }).

2. **AgentOutputSchemaBase:** ke andar `is_plain_text()` decide karta hai:
    - `True` → iska matlab output sirf string/text hai.
    - `False` → iska matlab output ek JSON schema ke structure mein hoga.

---

### 🔸name
`abstractmethod`
- Ye method agent output schema ko ek unique naam deta hai.
- Use Case Example:
    - Apne ek agent banaya jo kabhi text output dega ya kabhi structured JSON.
    - Debugging ke time pe agar agent kis schema se output le raha tha, ye naam usko clearly identify karta hay.
    - Yani logs mein ya multi-schema setups mein kaam aata hai.

**Code Example:**  
```python
from openai
 import AgentOutputSchemaBase

class MyTextOutput(AgentOutputSchemaBase):
    def name(self) -> str:
        return "plain_text"

    def is_plain_text(self) -> bool:
        return True

```
Yahan:  
`name()` → `"plain_text"` return kar raha hai (iska matlab ye output schema **plain text** handle karta hai).

Agar Json output banaty hain tw:
```python
class MyJSONOutput(AgentOutputSchemaBase):
    def name(self) -> str:
        return "json_output"

    def is_plain_text(self) -> bool:
        return False

```
#### Use case of name():
Jab ap **multiple output schemas** registar karty hain ek agent ky sath tw har ek ko pechan ky leye name ki zaroorat hoti hay.


### 🔸Quick Summary
- AgentOutputSchemaBase → Base class jo output ko control, validate aur parse karti hai.
- is_plain_text() → Decide karta hai plain text ya JSON schema.
- name() → Schema ka unique naam (debugging / multi-schema mein helpful).
- Default → Agar output_type na do to plain text use hota hai.

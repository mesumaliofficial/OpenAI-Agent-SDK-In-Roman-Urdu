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

3. **AgentOutputSchemaBase ka Kaam:**
    - `is_plain_text()` → ye determine karta hay kya ye output text hai ya JSON object.
    - Agar `False` hay tw `json_schema()` sy schema retrive karo aur `validate_json()` sy validate karo keh keys/ values format sahi hay ya nh.
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

### 🔸is_plain_text() -> bool
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

### 🔸name () -> bool
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

---

### 🔸json_schema() -> dict[str, Any]
`abstractmethod`  
json_schema() Ek abstract method hai jo tabhi call hota hai jab apka output plain text nahin hai (i.e., `is_plain_text() == False`).

#### In Simple words:
- Agar ap ka AgentOutput sirf text return kar raha hay tw → schema ki zaroorat nahi → json_schema() call nhin hoga.
- Agar ap ka AgentOutput ek structured JSON return kar raha hai tw → system ko pehly pata hona chahiye ki JSON ka structure kaisa hoga. Ye structure json_schema() method define karta hai.

---

### 🔸is_strict_json_schema() -> bool
`abstractmethod`
Ye batata hai keh JSON schema kis mode mein chalyga:

#### Strict Mode (True): 
- Guarantee deta hai ke output hamesha valid JSON hoga.
- Use case: jab tumhe 100% valid aur predictable JSON chahiye (jaise API integration, database entry).

#### Non-Strict Mode (False):
- Zyada flexible schema hota hai.
- Lekin kabhi kabhi LLM thoda free-style JSON generate kar sakta hai (invalid ya loose structure).
- Use case: jab tumhe thoda flexibility chahiye aur exact strictness zaroori nahi.

#### Default Behavior
By default ye apko hi implement karna hota hai (abstract method hai). Ap decide karty ho True ya False.

---

### 🔸validate_json(json_str: str) -> Any
`abstractmethod`  
Yeh ek abstract method hai jo tum custom output schema banate waqt implement karna parta hay.

Jab model se JSON string ata hay (jesy {"name": "Ali", "age": 22}), tw yeh method check karta hay keh:  
1. Kya JSON sahi format mein hay ?
2. Kya wo apky schema ke rules follow karta hai ?  
Agar sab sahi hota hay tw wo validated object return kardeta hay(jesy ek dict ya parsed data).  
Agar kuch galat hota hay tw `ModelBehaviorError` raise kar deta hay.


> **Note:** Agents library ke andar jo Agent / Runner / RunConfig classes hain, wo internally pydantic v2 ka use karti hain (models ke input/output ko validate karne ke liye). tw apko `AgentOutputSchemaBase` ky sath pydantic `BaseModel` ka use karna paryga.
 
---

### 🔸Resources & Practical Demos

- [OpenAI Agent Output Type/Structured output Colab Notebook](https://colab.research.google.com/drive/1HEz71EXQri48vW1hD8BHXVxk6iLV8skB?usp=sharing) → Interactive example with code and usage.

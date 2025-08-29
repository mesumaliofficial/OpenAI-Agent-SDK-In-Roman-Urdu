## 🔹 Openai SDK Api Refrences → Agent Output

### 🔸AgentOutputSchemaBase
Bases: `ABC`
Ye ek **base class** hai jo agent ke output ko control karta hai.

Jab agent koi response generate karta hai (LLM sy), tw ye class ensure karti hay keh:

- **Schema Capture:** Output ka default format kya hoga (e.g., string, dict, list)..
- **Validation:** Kya LLM ka response valid JSON hai aur schema ke according hay ya nahi.
- **Parsing:** Agar valid ho tw, isy Python object (dict, list, model) mein convert kar deta hai.


### 🔸AgentOutputSchemaBase ka kam kya hay ?  
Socho apky pass robots (agents) hain jo har sawaal ka jawab dety Lekin har robot apna jawab alag tareqy sy deta hay:

- Koi robot sirf text likhta hay → `"Hello!"`  
- Koi robot list deta hay → `[1, 2, 3]`  
- Koi robot JSON deta hay → `{ "text": "hello", "hobbies": [1,2,3] }`

Ab humein robots ko rule dena hai:
- Tum sab ek jesa jawab do, ya plain text doge ya structured JSON.
- Is tarha sy output predictable aur easy to use ho jata hai. 


### 🔸Behind the Scenes Flow (Easy Words)
1. **User Input:** `"100 USD ko PKR mein convert karo"`
2. **Agent Thinking:** LLM answer generate karta hay → `{"usd": 100, "pkr": 28000}`
3. **AgentOutputSchemaBase:**
    - `is_plain_text()` → decide karta hay keh output text hay ya JSON object.
    - Agar `False` ho tw → `json_schema()` se schema retrieve karo aur `validate_json()` se validate karo.
    - Agar sahi ho → Python object return karo.
4. **Final Result:** Agent user ko safe aur structured output deta hay.

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

### 🔸Method Explanations

1. `is_plain_text() -> bool`  
`abstractmethod`  
Ye check karta hai ke agent ka output plain text (str) hai ya structured JSON.
- **True:** Agar output True hay tw plain text hay.
- **False:** Agar output False hay tw JSON object hai (dict, list, Pydantic model).


2. `name () -> str`  
`abstractmethod`
Ye method agent output schema ko ek unique naam deta hay.  
**Use case:** Jab ap **multiple output schemas** registar karty hain ek agent ky sath tw har ek ko pechan ky leye naam ki zaroorat hoti hay.


3. `json_schema() -> dict[str, Any]`  
`abstractmethod`
- Agar output structured JSON hay tw, ye tabhi call hogi.
- Plain text output ke liye ye call nahi hoti.


4. `is_strict_json_schema() -> bool`  
`abstractmethod`
- **Strict Mode = True:** Output hamesha 100% valid JSON hoga, koi extra field ya wrong type allowed nahi hay.
- **Strict Mode = False:** Thori flexibility, Lekin kabhi kabhi LLM thora free-style JSON generate kar sakta hay (invalid ya loose structure) tw us case mein Error nahi ayega.


5. `validate_json(json_str: str) -> Any`  
`abstractmethod`  
- JSON string validate karta hay schema ke according.
- Agar valid hay → parsed Python object return hota hay.
- Agar invalid hay → ModelBehaviorError raise hota hay.

---

### 🔸AgentOutputSchema
Bases `AgentOutputSchemaBase`  
AgentOutputSchema ek dataclass wrapper hay jo AgentOutputSchemaBase ko inherit karky banaye gaye hay.

**Purpose:**
- Ye class LLM ke structured output ko handle karny ke liye use hoti hay.
- Apko manually parsing aur validation ke liye methods likhne ki zarurat nahi hoti hay.
- Ye automatically apky deye gaye model (Pydantic, dataclass ya TypedDict) sy JSON schema generate kar deti hay.

### 🔸How it works
1. Ap ek schema model define karty ho (Pydantic model ya dataclass).
2. Us model ko simply AgentOutputSchema(MyModel) mein wrap kar do.
3. Ye class automatically:
    - JSON schema generate karyge.
    - Output ko validate karyge.
    - Parsing handle karyge.

### 🔸Why use it
- Jab apko structured outputs chahiye (like JSON response, specific fields), bina boilerplate likhe.
- Ye recommended option hay kyunki simple aur direct usage deta hay.

---

**Note:** AgentOutputSchemaBase class pydantic ka use karty hay tw hame iska use karty waqt Pydantic BaseModel ko bhi sath inherit karna hoga


### 🔸Resources & Practical Demos

- [OpenAI Agent Output Colab Notebook](https://colab.research.google.com/drive/1HEz71EXQri48vW1hD8BHXVxk6iLV8skB?usp=sharing) → Interactive example with code and usage.

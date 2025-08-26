## 🔹 Openai SDK Api Refrences → Agent Output

### 🔸AgentOutputSchemaBase
Bases: `ABC`
Ye base class hay jo Agent ky output ko control karte hay.
Jab Agent koi Answer deta hay (LLM ka raw text ya json schema) tw ye class ensure karte hay:
- **Schema Capture:** Konsa format/schema follow karna hay output ka (e.g., dictionary, list, string) bt default string hota h.
- **Validation** Jo LLM ny output deya, wo **valid JSON** aur **sahi format** mein hay ya nh.
- **Parsing:** output valid hai, tw isy **Python object** (dict/list etc.) mein convert kar deta hai.


### 🔸Behind the Scenes Flow (Easy Words)
1. **User Input:** "100 USD ko PKR mein convert karo"
2. **Agent Thinking:** LLM answer generate karta hai → {"usd": 100, "pkr": 28000}

3. *AgentOutputSchemaBase ka Kaam:*
    - Check karega keh output JSON hai ya nahi.
    - Again Check karyga kya output schema (keys/values) sahi format follow karta hai.
    - Agar sab sahi ho to isy Python object bana ke return karega.
4. **Final Result:** Agent confidently user ko valid structured result deta hai.

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
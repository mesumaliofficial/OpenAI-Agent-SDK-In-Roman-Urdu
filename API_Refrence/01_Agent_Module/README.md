## 🔹 Agent Module

### 🔸 `set_default_openai_key`
```python
def set_default_openai_key(key: str, use_for_tracing: bool = True) -> None:
```
**Purpose:** Is function ka kaam OpenAI API key set karna hai. Isse ap apni API key set kar sakte hain aur tracing ko bhi enable ya disable kar sakte hain. By default, tracing enabled hota hai.

**Kab use hota hay:**
- Agar apny system mein environment variable OPENAI_API_KEY set nahi kiya, toh aap is function se directly API key set kar sakte hain.
- Ap is key ko tracing ke liye bhi use kar sakte hain.

**Parameters:**
-   `key:str` → Wo OpenAI API key jo aap use karna chahte hain (required).
- `use_for_tracing: bool` → Agar True hai, toh tracing ke liye bhi yehi key use hogi. Agar False hai, tw apko tracing ke liye alag se API key set karni padegi `set_tracing_export_api_key()` ke zariye. (Default: True).

### 🔸 `set_default_openai_client`
```python
def set_default_openai_client(client: AsyncOpenAI, use_for_tracing: bool = True) -> None:
```
**Purpose:** Ye function ek OpenAI client object set karta hai jo LLM requests aur tracing ke liye use hota hai. Agar ap client provide karte hain, toh ye default OpenAI client ki jagah use hoga.

**Parameters:**
- `client: AsyncOpenAI` → Wo OpenAI client object jo ap use karna chahte hain (required).
- `use_for_tracing: bool` → Agar True hai, toh client ki API key tracing ke liye bhi use hogi. Agar False hai, toh tracing ke liye alag key set karni hogi. (Default: True)


### 🔸 `set_default_openai_api` 
```python
def set_default_openai_api(api: Literal["chat_completions", "responses"]) -> None:
```
**Purpose:**  Is function se ap decide kar sakte hain ke OpenAI ki konsi API default LLM requests ke liye use hogi.

- Default API `chat_completions` hoti hai.
- Ap isko `responses` bhi set kar sakte hain agar ap response-based API use karna chahty hain.


### 🔸 `set_tracing_export_api_key`
```python
def set_tracing_export_api_key(api_key: str) -> None:
```
**Purpose:** Ye function ek alag API key set karta hai jo sirf tracing ke liye use hoti hai.


### 🔸 `set_tracing_disabled`
```python
def set_tracing_disabled(disabled: bool) -> None:
```
**Purpose:** Ye function tracing system ko enable ya disable karne ke liye use hota hai.
- Agar ap tracing ko temporarily band karna chahte hain to disabled=True set karen.
- Tracing ko phir se shuru karne ke liye disabled=False set karen.


### 🔸 `set_trace_processors`
```python
def set_trace_processors(processors: list[TracingProcessor]) -> None:
```
**Purpose:** Tracing data ko process karne ke liye chhote chhote functions (processors) hote hain. Ye processors decide karte hain ke tracing data ko kaise handle kiya jayega.

- Tracing ke liye ek “team” (processors ki list) banaye jati hai.
- Ye “team” yani wo functions ya objects jo tracing data ko process karty hain.
- Jab ap set_trace_processors() call karte hain, to purani “team” hata di jati hai aur sirf naye processors hi tracing data ko handle karenge.
- functions dety hain, sirf wo hi tracing data ko handle karenge.
- Pehle ke processors is ke baad kaam nahi karenge.


### 🔸 `enable_verbose_stdout_logging`
```python
def enable_verbose_stdout_logging() -> None:
```
**Purpose:**  Ye function apke program mein verbose logging enable karta hai jo standard output (console) par detailed logs print karta hai.

**Verbose logging kya hoti hai ?** <br>
Verbose logging wo hoti hai jab ap isko enable karte hain to har choti bari cheez detail mein console par dikha deti hai, jo debugging ke liye bohat helpful hoti hai.

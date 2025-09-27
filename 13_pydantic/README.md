## 🔹 Pydantic

### 🔸What is pydantic?

Pydantic ek Python library hay jo:

1. **Data Validation:** check karti hay data sahi type aur format mein hay ya nahi

2. **Parsing:** agar possible ho to data ko automatically sahi type mein convert kar deta hay

3. **Type Hints** Ye apky code ko batata hay ke "Is jagah par kis tarha ka data expect kiya ja raha hay."

---

### 🔸What is pydantic Models?

Jab bhi humein user se data lena ho (jesy API request body, config file, ya form input), hamein ensure karna hota hay keh:

- Data sahi type ka ho (string, int, bool, etc.)
- Agar koi required field missing ho tw error aaye
- Extra fields ignore ya reject ho sakein

Ye sab kam manually likhny ki bajaye **Pydantic Models** automatically kar lete hain.

---

### 🔸BaseModel

Pydantic ek BaseModel class provide karta hay.
Jab hum apna model (class) banaty hain aur usy `BaseModel` se inherit karty hain, tw hamein usky andar **automatic validation** aur **parsing** ki power mil jati hay.

---

### 🔸Fields Aur Types

Pydantic Models ke andar hum jo cheezein likhty hain, unhein **Fields** kehty hain, aur unky sath jo `str`, `int`, `list` likhty hain, wo **Types** hote hain.

#### Common Types

| **Type** | **Matlab (Urdu)**           | **Example**                  |
| -------- | --------------------------- | ---------------------------- |
| `str`    | Text (Alfaaz)               | `"Lahore"`, `"Hello"`        |
| `int`    | Pura Number                 | `10`, `500`                  |
| `float`  | Point Waly Numbers          | `3.14`, `1.5`                |
| `bool`   | Sahi Ya Galat               | `True`, `False`              |
| `list`   | Cheezon Ka Group            | `["apple", "banana"]`        |
| `dict`   | Key-Value Data (Dictionary) | `{"name": "Ali", "age": 20}` |
| `tuple`  | Ordered Fixed Group         | `(1, 2, 3)`                  |
| `set`    | Unique Values Ka Group      | `{"apple", "banana"}`        |

---

### 🔸Optional Aur Required Fields

Jab hum Pydantic Model mein fields define karty hain, tw hamein batana parta hay ke konsa data hamesha ana chahiye (Required) aur konsa data agar na bhi aaye tw masla nahi (Optional).

#### Required Fields

- Jab hum bas `field_name: type` likhty hain, tw Pydantic usko required samajhta hay.
- Matlab: wo data aana hi chahiye, warna error dega.

```python 
from pydantic import BaseModel

class Order(BaseModel):
    item_name: str      # Required
    quantity: int       # Required

# Error, kyunki "quantity" missing hai
order1 = Order(item_name="Mobile")

```

#### Optional Fields

Agar ap chahty ho keh field na bhi aaye tw error na aaye, uske 2 tareeqy hain:

- **Default Value:** Agar ap field ko ek default value de dogy, tw wo optional ban jati hay.

```python
from pydantic import BaseModel

class Order(BaseModel):
    item_name: str
    quantity: int
    discount: float = 0.0   # Optional (default value)

order1 = Order(item_name="Mobile", quantity=2)
print(order1)
# item_name='Mobile' quantity=2 discount=0.0
```

- **Optional:** Agar tum chahty ho keh field aaye ya na aaye, aur default None ho jaye, tw ap Optional ka use karty ho.

```python
from typing import Optional
from pydantic import BaseModel

class Order(BaseModel):
    item_name: str
    quantity: int
    discount: Optional[float] = None   # Optional

order1 = Order(item_name="Mobile", quantity=2)
print(order1)
# item_name='Mobile' quantity=2 discount=None

```

---

### 🔸Enums (Fixed Choices)

Enum ka matlab hota hay "Ek Fixed List Mein Sy Choice Karna"

Imagine karen, apki ice-cream shop per sirf 3 flavours hain: Vanilla, Chocolate, aur Strawberry. Agar koi "Pizza" flavour maangy, tw wo galat hay.


```python
from pydantic import BaseModel
from enum import Enum

class UnitType(Enum):
  VANILLA = 1
  CHOCOLATE = 2
  STRAWBERRY = 3

class IceCream(BaseModel):
  flavour: UnitType
  price: float
  size: str

icecream1 = IceCream(flavour=UnitType.VANILLA, price=100, size="small")
print(icecream1)

# Output: flavour=<UnitType.VANILLA: 1> price=100.0 size='small'
```

---

### 🔸What is Default Factory?

default_factory ek tareeqa hay jisky zariye hum Pydantic ko bataty hain keh: "Agar user is field ka data na dy, tw tum khud yeh function chalao, aur us function ka result default value ke taur par use karo.

```python
from pydantic import BaseModel, Field
from typing import list

# list ko default value deny ke leye hum default_factory use karte hain
class TaskSet(BaseModel):
    owner_name: str
    
    # Humne Pydantic ko kaha: "Jab koi naya TaskSet bano, tw list() function chalao"
    tasks: list[str] = Field(default_factory=list) 

set1 = TaskSet(owner_name="Ali")
set2 = TaskSet(owner_name="Sara")

# Agar hum set1 ki list mein kuch daalte hain
set1.tasks.append("Research Topic A")

print(f"Ali ke Tasks: {set1.tasks}") # Output: ['Research Topic A']
print(f"Sara ke Tasks: {set2.tasks}") # Output: [] - Yeh khaali hai!

# Agar hum 'default=[]' istemal karte, toh set2 mein bhi 'Research Topic A' aa jaata, jo galat hota.
```

---

### Resources & Practical Demos

- [Python Pydantic Colab Notebook](https://colab.research.google.com/drive/1HMAq6lsLtWVzsPO5zjdJawCuklD6Ktor?usp=sharing) → Interactive example with code and usage.


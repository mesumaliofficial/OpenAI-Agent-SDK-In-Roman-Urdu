## 🔹 Dynamic Instructions

### 🔸What is Instructions?

Instruction ek tarha ka master plan hay jo agent ko batata hay keh:

- Ap kon ho ?  (e.g., "Ap ek customer support agent ho.")
- Apka kam kya hay? (e.g., "Ap sirf technical questions ky answer dogy.") 
- Apko kya nahi karna hay? (e.g., "Ap personal batein nahi karogy.")

---

### 🔸What is Dynamic Instructions?

Normal Instruction tw ek hi bar dydi jati hain, Lekin **dynamic instructions** har bar change ho sakti hain.  
Iska matlab hay keh Agent ko har naye user ya har naye sawal ke liye alag-alag instructions mil sakti hain.

**Example:**
- Apka Agent ek online store ka **sales Perosn** hay.
- Agar koi kapry dekh raha hay, tw ap dynamic instructions do gy keh "isko kapro keh bary mein batao." 
- Agar koi user **electronics** item dekh raha hay, tw ap instructions change karky do gy keh "isko gadgets ky bary mein batao."  

--- 

```python
def dynamic_instructions(context: RunContextWrapper[UserContext], agent: Agent[UserContext]) -> str:
    return f"The user's name is {context.context.name}. Help them with their questions."

agent = Agent[UserContext](
    name="Triage agent",
    instructions=dynamic_instructions,
)
```

Yahan `dynamic_instructions` ek function hay. jab bhi koi user ayega ye function run hoga aur check karyga user ka naam kya hay. Aur phr us user ke naam ke according instruction dega. jesy "is user ka naam Mesum hay. Mesum ki help karo.

---

### 🔸konsi Instruction Kab Use Karni Chahiye?

- **Static/Simple instruction** (jo change nh hoti): Ye tab use karen jab apky agent ka kam hamesha ek jesa ho.
    - **Real-world example:** Apka agent ek grammar correction bot hay. Iska kam hamesha user ke text ko correct aur polished karna hay.

- **Dynamic Instructions** (Jo situation ke according change hoti hay): Ye tab use karen jab apka Agent alag-alag kam kar sakta ho ya usko user ke context (jesy: user ka naam, uski location etc.) ke according response dena ho.
    - **Real-world example:** Apka Agent ek **personalized shopping assistant** hay. Ye har user ki shopping history ko dekhta hay aur usky according **suggestions** deta hay. Yahan dynamic instructions perfect hay.

---

### 🔸Resources & Practical Demos

- [OpenAI Agent SDK **dynamic_instructions** Colab Notebook](https://colab.research.google.com/drive/1To77G-6wWfK5luDe45NeciJL0yCZwWIy?usp=sharing) → Interactive example with code and usage.

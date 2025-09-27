## 🔹Guardrails

### 🔸What are Guardrails?

Sochen apka Agent (ek smart robot) ek factory mein kam karta hay.  
Guardrails us factory ke security guards ki tarha hoty hain jo har cheez check karty hain keh sab safe aur sahi hay.

---

### 🔸What is the role of Guardrails?

1. **Input Check (Aane Wali Cheez):**  
Jab koi customer apky robot se bat karta hay (yani user input deta hay), tw Guardrails us bat ko pehly check karty hain. jesy security guard dekhta hay ki koi badmashi tw nahi kar raha.

2. **Parallel Mein Kam:**  
Guardrails aur Agent ek hi time par apna kam start karty hain.

    - Agent apna task shuru karta hay.
    - Guardrail apni verification karta hay.
    - Agar Guardrail ko koi issue mil jaye tw foran error raise kar dega aur Agent ko rok dega.


3. **Output Check (Jany Wali Cheez):**  
Jab agent apna final output deta hay, tw Guardrails us output ko last time check karty hain.
Jesy security guard gate par check karta hai ke sab thek hay tw hi koi bahir jaye ga.

---

### 🔸Purpose of guardrails?

Guardrails ka main purpose hay safety aur control! Agar apka Agent customer support agent hay, tw ap nahi chahogy keh wo:

- Kisi ko gali dy (safety).
- Kisi ka math homework kary (control aur paisy bachana, kyunki wo expensive model use karengy)
- Bina kisi reason ke off-topic batein kary (relevance check)

---

### 🔸Types of Guardrails

Guardrails sirf do types ke hoty hain:

1. **Input Guardrails:** jo user ke input par lagty hain.
2. **Output Guardrails:** jo agent ke final output ya handoff par lagty hain.

---

### 🔸Input Guardrails

Input Guardrail agent ke start hone se pehle hi input check karta hai.

#### Input Guardrails run in 3 steps:

1. **Receive Input:**  
Guardrail ko wahi input milta hay jo Agent ko milny wala hota hay.

2. **Run Function:**  
Guardrail function run hota hay aur ek `GuardrailFunctionOutput` banata hay. Phir ye result `InputGuardrailResult` mein wrap hota hay.

3. **Check Tripwire:**  
    - Agar `tripwire_triggered = True` ho tw ek `InputGuardrailTripwireTriggered` exception raise hoti hay.

    - Agar `tripwire_triggered = False` ho tw Agent apna kam normal tareeqy se continue karta hay.

---

> **Note:**  
Input guardrails sirf tab chalty hain jab yeh Agent flow mein sabsy pehla agent ho.
Guardrails property Agent ke andar hi hoti hay (na ke Runner.run mein), kyunki Guardrails usually ek specific Agent ke liye design hoty hain.
Isse code readable aur maintainable rehta hay.

---

### 🔸Output guardrails

Output Guardrail Agent ka kam khatam hony ke bad usky final output ko dekhta hay. Ye ensure karta hay ki Agent ny koi uljhi hui ya galat cheez tw nahi keh di.

#### Output Guardrails run in 3 steps:

1. **Receive Output:**  
Guardrail ko wo output milta hay jo Agent ne banaya hay.

2. **Run Function:**  
Apka guardrail function chalta hay, aur `GuardrailFunctionOutput` deta hay.

3. **Check Tripwire:** 
    - Agar `tripwire_triggered = True` ho tw ek `OutputGuardrailTripwireTriggered` exception raise hoti hay.
    - Agar `tripwire_triggered = False` ho tw Agent apna kam normal tareeqy se continue karta hay.

---

### 🔸What is Tripwires?

Agar input ya output par guardrail fail ho jaye, tw Guardrail isy **tripwire** ke zariye signal karta hay.  
Jesy hi guardrail tripwire trigger karta hay, foran `{Input,Output}GuardrailTripwireTriggered` exception raise hoti hay aur Agent execution stop ho jati hay.

---

### Resources & Practical Demos

- [OpenAI Guardrails Colab Notebook](https://colab.research.google.com/drive/1KQebml5D3-wfxiyXEAXXXTCrU34MfBSj?usp=sharing) → Interactive example with code and usage.

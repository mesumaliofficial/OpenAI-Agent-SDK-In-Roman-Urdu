## 🔹 Model Settings

### 🔸 What Are Model Settings?

Ye settings wo hoti hain jo LLM call karty time use ki jati hain.

Jab hum Agent ko koi sawal dety hain, tw wo direct jawab nahi deta. Pehly hum usko bataty hain keh "Tum kitna creative output dy sakty ho?" ya "Tum kitna lamba jawab de sakty ho?"

`ModelSettings` class inhi sari **optional configuration parameters** ko hold karti hay.


---

### 🔸Model Settings Parameters

### 🔸Temperature (Creativity)

Ye control karta hay keh Agent ka jawab kitna naya aur creative hoga.

#### When to use:

- Low (0.1-0.3): Math, facts, precise instructions
- Medium (0.4-0.6): General conversation, explanations
- High (0.7-0.9): Creative writing, brainstorming

---

### 🔸Top_p (Focus/Attention)

Ye setting decide karti hai model kitne words consider karega.

- **Low top_p (0.1–0.5)** → Sirf top words (deterministic, safe).
- **Medium top_p (0.6–0.9)** → Mix of top + kuch random (balanced).
- **High top_p (1.0)** → Sab words consider (zyada random).

---

### 🔸Frequency_penalty (Repetition Control)

Ye setting model ko naye lafz aur topics introduce karny per force karti hay.

- **Low (0.0)** → Repetition allow, naye lafz ke leye force nahi.
- **Medium (0.1 – 0.5)** → Thora balance, kabhi repeat kabhi naya.
- **High (0.6 – 1.0)** → Zyada naye lafz aur ideas, repeat kum.

---

### 🔸Presence_penalty (Repetition Avoid / Variety)

Ye setting dekhti hay agar koi lafz phely sy jawab mein aa chuka hay, tw usko dubara use karny ka chance kum kar deti hay.

- **Low (0.0)** → Repeat allow.
- **Medium (0.1 – 0.5)** → Thora variety, thoda repeat.
- **High (0.6 – 1.0)** → Repeat kum, naye lafz aur ideas zyada.

---

### 🔸Tool_choice (Tool ka decision)

Ye setting decide karti hay keh Agent tool use kary ya nahi, aur agar kary to konsa tool use kary.

- **auto** → Agent khud decide karega.
- **required** → Tool use karna zaroori hoga.
- **none** → Bilkul tool use nahi hoga.

---

### 🔸Parallel Tool Calls (Ek Saath Tools)

Yeh setting decide karti hay keh Agent ek hi waqt mein ek sy zyada tools use kar sakta hay ya nahi.

- **False** → Ek hi turn mein sirf 1 tool use hoga.
- **True** → Multiple tools ek hi turn mein call ho sakty hain.
- **None** → Provider ka default chalega (OpenAI = True).

---

### 🔸Max Tokens (Jawab Ki Limit)

Yeh setting decide karti hat keh Agent ka jawab kitna lamba ho sakta hay. Token = word ka chota hissa (3–4 characters).

- **Example:** `max_tokens=200` → sirf 200 tokens tak generate karyga.

---

### 🔸Resolve Function (Settings Merge)

Ye function do alag-alag jagah ki Model Settings ko merge karta hay. Agar dono mein same setting ho tw override wali value use hoti hay.
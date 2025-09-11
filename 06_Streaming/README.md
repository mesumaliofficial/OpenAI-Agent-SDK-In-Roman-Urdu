## 🔹 Straeming
Streaming allow karta hay keh ap agent ko run karty time hi uski updates ko subscribe kar saken. Bina pury final output any ky wait keye huwy.  
Ya user ko **live progress updates** aur **partial responses** dikhany ky leye useful hota hay.

- `Runner.run_streamed()` → Ye ek streaming start karta hay aur RunResultStreaming object return karta hai.
- `RunResultStreaming` → Streaming ka result wrapper hay jo RunResultBase ko inherit karta hay. jisme methods aur attributes hoty hain (jaise stream_events(), cancel(), is_complete etc.).
- `result.stream_events()` → Ek async iterator deta hay jo **StreamEvent** objects emit karta hay. Har event ka ek type hota hai (e.g. raw_response_event, run_item_stream_event) aur uske revelant data hota hay.

---

### 🔸 Simple Analogy
Streaming kya hoti hay. Imagine karo, ap aur apka dost video call par baat kar rahy ho. Apke dost ko puri video file ek sath download karny ki zarurat nahi parti, right? Balkay video choty choty chunks mein aati rehti hay, aur ap usko foran dekh sakte hain. Yeh hi streaming hai.

---

### 🔸 Why is Streaming Important?
Socho apny ek agent banaya aur user ne usko article likhne ko kaha. Normally wo agent pura article apne paas likhega aur phir ek sath output dega. Jab tak wo likh raha hoga tab tak user ko sirf `"Loading..."` ya `"Thinking..."` dekhaye dega.  
Ye experience acha nh lagta right ? aesa lagta hay computer hang hogaya hay.

### 🔸 Benefits of Streaming:
1. **UX bhetar hota hay:** Jesy hi agent 1 lafz ya sentence likhta hay wo foran screen per nazar any lagta hay keh agent kam kar raha hay.

2. **Fast feel hota hai:** Technically pura response any mein utna hi time lagega lekin user ko aesa feel hoga keh response jaldi aa raha hay. kyun ki kuch part foran nazar aa jata hay.

3. **Transparency:** User ko bata sakty hain AI kya kar raha hay (jesy tool call kiya ya text likh raha hai).

---

### 🔸 Normal Run vs Streamed Run
- **Normal Run** Jab ap ek normal `Runner.run()` chalaty hain tw ye method kam shuru karta hay aur tab tak wait karta hay jab tak pura output generate nh ho jata. Jab generate h jata hay tw `RunResult` object return karta hay.

- **Streamed Run:** Jab ap `Runner.run_streamed()` chalaty hain tw ye method kam shuru karta hay. Lekin ye wait nh karwata pury kam kahatm hony tak bulky ek `RunResultStreaming` object return karta hay.  


### 🔸 RunResultStreaming Simple Pipeline Analogy
- Jab ap stream_events() call karty ho tw ye pipeline open ho jata hay.
- Ab jesy hi AI agent apna kam shuru karta hay (output banana, tools call karna, etc.), wo events pipeline me bhejta rehta hay.
- Apka code in events ko pipeline se receive karta hay aur user ko screen par dikhata hay. Bilkul wesy hi jesy video call me har frame foran ata hay.

---

### 🔸 Raw response events
- Jab LLM (AI model) output banata hay, wo pura ek sath nahi bhejta.
- Wo output ko choty choty tukro (tokens) me tor ke bhejta hay.
- Har tukra = ek **event** hota hay.

**Example**: Agar AI likh raha hai "Hello"

- Event 1 → "H"
- Event 2 → "e"
- Event 3 → "l"
- Event 4 → "l"
- Event 5 → "o"

Isko code me **event.data.delta** ke zariye access karte ho.

### 🔸 Real World Analogy
Socho ap ek **chef** ho jo cooking karty waqt har choti cheez ki report deta hay:

- "Maine ab namak dala"
- "Ab mirch daali"
- "Ab thoda paani daal raha hoon"

Yani bilkul detailed updates, step by step.

---

### 🔸 Run item events and agent events
- Ye high-level events hoty hain Yani (bary steps ki updates).
- Ye batate hain:
    - Jab AI koi **tool call** karta hay (tool_call_item).
    - Jab tool ka **result** ata hay (tool_call_output_item).
    - Jab AI ka **final message** taiyar ho jata hay (message_output_item).

### 🔸 Real World Analogy
Socho wahi cook ab har bari stage ki report deta hai:

- "Sabziyaan kaat li hain"
- "Ab main khana paka raha hoon"
- "Khana tayaar ho gaya hai"

Yeh choti choti detail ke bajaye bary milestones bataty hain.

---

### Resources & Practical Demos

- [OpenAI Agent Streaming Method Colab Notebook]() → Interactive example with code and usage.

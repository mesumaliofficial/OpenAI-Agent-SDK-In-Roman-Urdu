## 🔹 Straeming
Streaming allow karta hay keh ap agent ko run ke time hi uski updates ko subscribe kar saken. Bina pury final jawab any ky wait keye. Ya user ko **live progress updates** aur **partial responses** dikhany ky leye useful hota hay.

- `Runner.run_streamed()` → Ye ek streaming start karta hay aur RunResultStreaming object return karta hai.
- `RunResultStreaming` → Streaming ka result wrapper hay jisme methods aur attributes hote hain (jaise stream_events(), cancel(), is_complete etc.).
- `result.stream_events()` → Ek async iterator deta hay jo StreamEvent objects emit karta hay. Har event ka ek type hota hai (e.g. raw_response_event, run_item_stream_event) aur uske corresponding data hota hay.

---

### 🔸 Simple Analogy
Streaming kya hoti hay. Imagine karo, ap aur apka dost video call par baat kar rahy ho. Apke dost ko puri video file ek sath download karny ki zarurat nahi parti, right? Balkay video choty choty chunks mein aati rehti hay, aur ap usko foran dekh sakte hain. Yeh hi streaming hai.

---

### Resources & Practical Demos

- [OpenAI Agent Streaming Method Colab Notebook]() → Interactive example with code and usage.

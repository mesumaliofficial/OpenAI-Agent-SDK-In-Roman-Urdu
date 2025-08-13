## 🔹 Context Management
Context ek overloaded term hay. (yani ye lafz diffrent jaga aur diffrent purposes ke liye use hota hai.). Context do main types ka hota hay:

1\- **Local Context:** Ye wo data aur dependencies hain jo sirf apke code ke liye available hoti hain. Iska use aap:

- Tool functions
- `on_handoff`
- Lifecycle hooks

wagera mein karty hain.

**Important:** Ye data direct LLm ko nh jata. agar apny ye tool ky zariye pass keya ho tw LLm ko ye srf tool context mein hi milyga.

2\- **LLM Context:** Ye wo context hay jo LLM response generate karty waqt dekhta hay:

- User ka message
- Conversation history
- Instruction prompt

LLM isi data ko use karke apna response generate karta hay.


### 🔸 Context ka purpose ?
OpenAI SDK mein context ka role agent ko stateful banane ke liye hota hai. Iska matlab hai ke agent apni purani baatein ya information yaad rakhta hai, taki har dafa se shuru se batana na pary agent ko.



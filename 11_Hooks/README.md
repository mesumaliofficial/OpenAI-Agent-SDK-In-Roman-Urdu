## 🔹 Life Cycle Hooks

### 🔸What are Hooks?

Agent jab koi kam shuru karta hay aur jab wo kam khatam karta hay, tw wo usky beech mein bohut saary steps leta hay.

Ye "Life Cycle Hooks" unhi steps par nazar rakhny aur us waqt kuch khaas kam karny ka chance dety hain.

Sochen, agar apka Agent 'chaye' bana raha hay, tw usky steps ye hongy:

- Start (Kam Shuru kiya)
- Pani Daala
- Chaye Ki Patti Daali
- Doodh Daala
- End (Kam Khatam huwa)

Hooks ka matlab hay, ap in har steps (ya event) ke bilkul shuru ya bilkul end mein apna ek chota sa kam karwa sakty ho. jesy, 'Pani Daala' step se pehlay tum note bana lo ki "Pani Dalne Wala Hay".


---

### 🔸Types of Hooks

Hooks do tarha ky hoty hain:

### 🔸AgentHooks

Ye wo events hain jo **sirf ek specific Agent** par lagty hain.

**For example:** Apki team ka Captain Agent hay. Jab ap `AgentHooksBase` lagaty ho, tw wo sirf **Captain_Agent** ke har kam ko dekhta hay, wo kab sochta hay, kab koi tool use karta hay. Ussy baki team se koye matlab nahi hota.

Yeh direct usi Agent par lagaya jata hay, uski `hooks` property mein.

---

### 🔸RunHooks:

Yeh pury runner per nazar rakhta hay, chahy kitny bhi agents kam kar rahy ho. 

**For example:** Apky dosto ki ek puri team hay. Jab ap `RunHooksBase` lagaty ho, tw wo dekhta hay keh **Team A** ny kab kam shuru keya, kab unhony **Tool** use keya, aur kab unhone kam **Team B** ko dy diya.

Ye Agent ko run karty time lagaya jata hai, taaky puri conversation ki monitoring ho saky.

---

### 🔸Agent Hooks Events

- `on_start`: Jab **Agent apna kam shuru karta hay**, tab ye trigger hota hay.

- `on_end`: Jab **Agent apna kam khatam karta hay**, tab ye trigger hota hay.

- `on_handoff`: Jab **Agent apna control kisi aur Agent ko handoff karta hay**, tab ye trigger hota hay.

- `on_tool_start`: Jab **Agent koi tool use karna start karta hay**, tab ye trigger hota hay.

- `on_tool_end`: Jab **tool ka kam complete ho jata hay aur output milta hay**, tab ye trigger hota hay.

- `on_llm_start`: Jab **Agent LLM ko call karta hay** (yani model se reasoning ya response mangta hay), tab ye trigger hota hay.  

- `on_llm_end`: Jab **LLM apna response wapas bhej deta hay**, tab ye trigger hota hay.

--- 

### 🔸Run Hooks Events

- `on_llm_start`: Jab **Agent LLM ko call karta hay** (yani model se reasoning ya response mangta hay), tab ye trigger hota hay.  

- `on_llm_end`: Jab **LLM apna response wapas bhej deta hay**, tab ye trigger hota hay.

- `on_agent_start`: Jab **Agent apna kam shuru karta hay**, tab ye trigger hota hay.

- `on_agent_end`: Jab **Agent apna kam khatam karta hay**, tab ye trigger hota hay.

- `on_handoff`: Jab **Agent apna control kisi aur Agent ko handoff karta hay**, tab ye trigger hota hay.

- `on_tool_start`: Jab **Agent koi tool use karna start karta hay**, tab ye trigger hota hay.

- `on_tool_end`: Jab **tool ka kam complete ho jata hay aur output milta hay**, tab ye trigger hota hay.

---

### Resources & Practical Demos

- [OpenAI Agent SDK **Hooks** Colab Notebook](https://colab.research.google.com/drive/1crKf1b4iTUxu69BiLMxVNbtGtrX7wTvq?usp=sharing) → Interactive example with code and usage.

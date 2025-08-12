## 🔹 OpenAI Agent SDK

OpenAI Sdk ek light-weight aur easy to use package hay, jo agent-bas apps banany mein help karta hay  iska design simple aur production-ready hay matlab ye testing sy nikal chuka hay aur reaql apps mein use keya jaa sakta hay.

SDK sy phely OpenAi ka 1 expermental-project tha jiska naam **Swarm** ye usi ka upgraded version hay, SDK kuch bunyadi chezen (premitives) ka use karta hay jissy powerfull apps banaye jaa sakty hain.

- **Agents** Ek LLM hota hay jo instructions aur tools per kam karta hay.
- **Handsoffs** Ye kisi ek agent sy task dusry agent ky Delegate (hawaly) kardeta hay.
- **Guardials** Ye use ka input check karta hay.
- **Sessions** Ye automatically coversation history store karta hay.

### 🔸 Why use the Agents SDK
SDK ky 2 Aehm design Principles hain.
- Isme features to kaafi hain, lekin abstraction kam hai, is wajah se seekhna asaan hai.
- de gaye chezen achi tarha kam karti hain lkein ap chane tw unhy customized bhi kar sakty hain.

### 🔸 Main Features of SDK
- **Agent Loop:** Agent loop built-in hota hay, ye khud hi tools call karta hay, result LLM ko bhjta hay, aur ye process tab tak chalta rehta hay jab tak koi final result tak nh poch jata.

- **Python-First:** Ye python base hay Agent ko chalany ky leye naye chezen sikhna nh parte hain, Python ki built-in chezen hi use hoti hain.

- **Handoffs:** Ye ek bohut powerful feature hay jo ek agent dusry agent koi kam asign kar sakta hay.

- **Guardrails:** Ye agents ke sath hi parallel chalte hain. Agar input mein koi masla ho, jaise user ne koi irrelevant ya unnecessary cheez puch raha ho ya kam karwa raha hu, to kaam wahin par rok diya jata hai. 

- **Sessions:** Ye automatically conversation history store karta hay agents ky run ky time manually karna nh parta.

- **Function tools:** Kisi bhi Python function ko tool bana sakte hain aur SDK automatically Schema generate karta plus pydantic ky zariye validation bhi karta hay

- **Tracing:** Tracing se dekh saktay hain ke agent kya kar raha hai, problems ko dhoondh saktay hain, aur system ka track rakh saktay hain. Saath hi aap OpenAI ke training aur testing tools jesy evaluation, fine-tuning, aur distillation bhi use kar saktay hain.
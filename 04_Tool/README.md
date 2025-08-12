## 🔹 Tools
### 🔸 Tools
Tools ek helper hota hay jo agent ko koi kam karny ki permission deta hay:
 data fetching.
- running code
- external APIs ko call karna jab need ho.
- aur Computer ka istemal bhi karna

### 🔸 What is tool calling ?
Tool calling ka matlab hay jab AI kisi conversation ke waqt decide karta hay usy kis tool ka use karna chahiye.

**Flow:**
- **User:** user ny pucha karachi ka weather kesa hay.
- **AI:** AI ny decide keya mujhy yahan weather Tool ki zaroorat hay.
- **AI calls tool:** Fetch Weather
- **Tool return:** {"temp": 33, "condition": "Cloudy"}
- **AI Answer:** Karachi ka mausam cloudy hay aur temperature 33°C hay.

### 🔸Why do we need tool ?
- LLMs ky pass live data nh hota hay is leye zaroorat parte hay.
- LLMs khud sy action nh ly sakta jesy mail send karna code run karwana etc.
- Tools se accuracy aur safety barhti hai, taake hum data ko sahi tareqy se hasil kar saken.

### 🔸Hosted Tools:
OpenAI kuch built-in hosted tools provide karta hai, jo OpenAIResponsesModel ka use kar ke access kiye ja sakte hain. In tools ki madad se AI real actions le sakta hai, bina manually intervene kiye.
- `WebSearchTool` Is tool se agent internet par search kar sakta hai.

- `FileSearchTool` Ye tool OpenAI ke Vector Store se documents dhoondhne mein madad karta hai. Agar aap ne kuch data store kar rakha ho (PDFs, policies, patterns waghera), to AI unko retrieve kar sakta hai.

- ` ComputerTool` AI ko apke computer ka limited access deta hai, taky files ko access ya open kiya ja sake (controlled environment mein). 

- `CodeInterpreterTool` AI ko code run karne ki ability deta hai, lekin ek safe sandboxed environment mein.
Sandboxed environment ka matlab hai secure jagah jahan AI sirf limited access ke sath code chala sakta hai.

-  `HostedMCPTool` Is tool se AI kisi remote MCP server ke tools ko use kar sakta hai.

- `ImageGenerationTool` Is tool se AI images generate kar sakta hai based on text prompts.

- `LocalShellTool` Ye AI ko permission deta hai ke woh aapke computer ke command line (terminal) par shell commands run kare.

```bash
from agents import Agent, FileSearchTool, Runner, WebSearchTool

agent = Agent(
    name="Assistant",
    tools=[
        WebSearchTool(),
        FileSearchTool(
            max_num_results=3,
            vector_store_ids=["VECTOR_STORE_ID"],
        ),
    ],
)

async def main():
    result = await Runner.run(agent, "Which coffee shop should I go to, taking into account my preferences and the weather today in SF?")
    print(result.final_output)
```

### 🔸Function Tool:
Ap kisi bhi python function ko ek tool ki tarha use kar sakty hain Agent SDK usko khud automatically setup kardyga.

- Tool ka naam wohi hoga jo function ko dengy.
- Tool ki description function ky docstring sy le jati hay recommended hay lazmi dein.
- Function ky aruguments se input schema LLM khud ready karta hay.
- Har input ki description docstring sy le jati hay (agar disable na keya gaya ho).

**is process mein:** 
- inspect module: Ye Python ka built-in module hai jo function ke signature ka structure nikalta hai. (signature ka matlab hay, function ka naam, arguments aur return type)
- griffe: Tool ko yeh samajhne mein help karta hay, keh har field ka matlab kya hai.
- pydantic: Ye ek library hai jo inputs ka data schema banata hai aur unki validation karta hai. for example: Agar koi user `amount="not_a_number"` bhej de, tw `pydantic` kahega: ❌ “Yeh float number hona chahiye!”

```bash
import json

from typing_extensions import TypedDict, Any

from agents import Agent, FunctionTool, RunContextWrapper, function_tool


class Location(TypedDict):
    lat: float
    long: float

@function_tool  
async def fetch_weather(location: Location) -> str:
    
    """Fetch the weather for a given location.

    Args:
        location: The location to fetch the weather for.
    """
    # In real life, we'd fetch the weather from a weather API
    return "sunny"


@function_tool(name_override="fetch_data")  
def read_file(ctx: RunContextWrapper[Any], path: str, directory: str | None = None) -> str:
    """Read the contents of a file.

    Args:
        path: The path to the file to read.
        directory: The directory to read the file from.
    """
    # In real life, we'd read the file from the file system
    return "<file contents>"


agent = Agent(
    name="Assistant",
    tools=[fetch_weather, read_file],  
)

for tool in agent.tools:
    if isinstance(tool, FunctionTool):
        print(tool.name)
        print(tool.description)
        print(json.dumps(tool.params_json_schema, indent=2))
        print()
```
<details>
<summary>Expand to see output</summary>

fetch_weather  
Fetch the weather for a given location.

```json
{
  "$defs": {
    "Location": {
      "properties": {
        "lat": {
          "title": "Lat",
          "type": "number"
        },
        "long": {
          "title": "Long",
          "type": "number"
        }
      },
      "required": ["lat", "long"],
      "title": "Location",
      "type": "object"
    }
  },
  "properties": {
    "location": {
      "$ref": "#/$defs/Location",
      "description": "The location to fetch the weather for."
    }
  },
  "required": ["location"],
  "title": "fetch_weather_args",
  "type": "object"
}
fetch_data
{
  "properties": {
    "path": {
      "description": "The path to the file to read.",
      "title": "Path",
      "type": "string"
    },
    "directory": {
      "anyOf": [
        {
          "type": "string"
        },
        {
          "type": "null"
        }
      ],
      "default": null,
      "description": "The directory to read the file from.",
      "title": "Directory"
    }
  },
  "required": ["path"],
  "title": "fetch_data_args",
  "type": "object"
}
```
</details>

### 🔸Custom Function Tool
Kabhi kabhi ap python function ko tool ky tor per use nh karna chahty, agar ap chahen tw direct tool bhi bana sakty `FunctionTool` ki madad sy apko srf ye chezen provide karni honge.

- **name →** Tool ka naam.
- **description →** Tool kya karta hai, LLM ko samjhane ke liye.
- `params_json_schema →` Ap apny Tool ke input parameters ko JSON schema mein define karty hain.
- `on_invoke_tool`  jo ek async function hota hai jo ek `ToolContext` aur arguments ko JSON string ke tor par receive karta hai, aur tool ka output string ke tor par return karna hota hai.

```bash
from typing import Any

from pydantic import BaseModel

from agents import RunContextWrapper, FunctionTool



def do_some_work(data: str) -> str:
    return "done"


class FunctionArgs(BaseModel):
    username: str
    age: int


async def run_function(ctx: RunContextWrapper[Any], args: str) -> str:
    parsed = FunctionArgs.model_validate_json(args)
    return do_some_work(data=f"{parsed.username} is {parsed.age} years old")


tool = FunctionTool(
    name="process_user",
    description="Processes extracted user data",
    params_json_schema=FunctionArgs.model_json_schema(),
    on_invoke_tool=run_function,
)
```

### 🔸Automatic argument and docstring parsing
Jaise pehle bataya gaya, tool automatically function ka signature parse karta hai taky tool ke arguments ka schema ban sake, aur hum docstring parse karte hain taky LLM ko tool ki description mil saky.

- `inspect module →` inspect Python ka built-in module hai jo function ya class ka structure read kar sakta hai. Iska use karke arguments ka naam aur type annotations extract kiye jaty hain, aur unse dynamically ek Pydantic model banaya jata hai. Pydantic model data validation ke liye hota hai. yeh ensure karta hai ke jo data pass ho raha hai wo required schema ke mutabiq ho. Yeh parser zyada tar types ko support karta hai, including: Python primitives (str, int, bool, float), Pydantic models, TypedDicts, Aur complex nested types. 

- `griffe →` ek library hai jo Python code se docstrings read karky samajh sakti hay. Docstring ke teen formats supported hain: Google style, Sphinx style, NumPy style. System koshish karta hai ke docstring ka format automatically detect kare, lekin hum isse manually bhi set kar sakty hain jab `function_tool` call karty hain. agar ap chahen tw Docstring parsing disable bhi kar sakty hain `use_docstring_info=False` set karky.

### 🔸Agents as tools
Normally jab hum ek agent sy dusry agent ko kam handoff karty hain tw control usky pass chala jata hay lekin kuch workflows me hum ye nh chahty ek central yani boss agent ho jo har spcialize agent sy kam karwaly lekin control apny pass rakhy.

``` bash
from agents import Agent, Runner
import asyncio

spanish_agent = Agent(
    name="Spanish agent",
    instructions="You translate the user's message to Spanish",
)

french_agent = Agent(
    name="French agent",
    instructions="You translate the user's message to French",
)

orchestrator_agent = Agent(
    name="orchestrator_agent",
    instructions=(
        "You are a translation agent. You use the tools given to you to translate."
        "If asked for multiple translations, you call the relevant tools."
    ),
    tools=[
        spanish_agent.as_tool(
            tool_name="translate_to_spanish",
            tool_description="Translate the user's message to Spanish",
        ),
        french_agent.as_tool(
            tool_name="translate_to_french",
            tool_description="Translate the user's message to French",
        ),
    ],
)

async def main():
    result = await Runner.run(orchestrator_agent, input="Say 'Hello, how are you?' in Spanish.")
    print(result.final_output)
```

### 🔸Custom output extraction
kuch cases mein hum chahty hain tool-agents ka output boss agent ko return karny sy phely modify keya jaye.

#### Kyun zaroorat padti hai?**
- **Specific data extract karna** Sub-agent ne apni chat history mein bohut sara text bhej deya. lekin hame srf (json payload) chahiye.

- **Format change karna** Sub-Agent ne Markdown format mei output deya. lekin hame plain text ya CSV mein chahiye. 

- **Output validate karna** Sub-Agent ka response empty ya galat format mein ho aur tum ek fallback/default value dena chahty ho

ap ye kam `as_tool` method mein `custom_output_extractor` argument provide karky kar sakty hain.

```bash
async def extract_json_payload(run_result: RunResult) -> str:
    # Scan the agent’s outputs in reverse order until we find a JSON-like message from a tool call.
    for item in reversed(run_result.new_items):
        if isinstance(item, ToolCallOutputItem) and item.output.strip().startswith("{"):
            return item.output.strip()
    # Fallback to an empty JSON object if nothing was found
    return "{}"


json_tool = data_agent.as_tool(
    tool_name="get_data_json",
    tool_description="Run the data agent and return only its JSON payload",
    custom_output_extractor=extract_json_payload,
)
```

### 🔸Handling errors in function tools
jab hum `@function_tool` docorator ka use karky ek function tool banaty hain. tw hum optionally error provide kar sakty hain ek `failure_error_function` provide karky

- **Default Behavior** Agar kuch pass nh karty tw by default `default_tool_error_function` run hota hay

- **Custom Error Function** Agar apna custom `failure_error_function` pas karty hain tw system hamara function run karyga. aur ap custom message ya error bhj sakty hain.

- **Passing None** Agar tum explicitly `failure_error_function=None` pass karte ho tw ko koi error hanling nh hoti. error ko re-raise keya jata hay. taky apny code mein manually handle kar saken. jesy: `ModelBehaviorError` Agar model ne invalid JSON produce kiya, `UserError` Agar tumhara code crash ho gaya.

- **Manual FunctionTool creation** Agar decorator ka use nh kar rahy aur manually object bana rahy tw khud sy error handling karna paryge `on_invoke_tool` ky andar by default nh hoge.


### 🔸Forcing tool use
- Tool ki list dena hamehsa ye matlab nh hota keh agent tool ka istemal karyga, agar ap chahen tw zabardasti bhi tool ka istemal karwa sakty hain `ModelSettings.tool_choice` set karky.

- `auto` By default, ye option LLM ko decide karne deta hai ke tool use karna hai ya nahi.
- `required` Ye LLM ko force karta hai ke lazmi tool use kare lekin konsa tool use karna hai, ye wo khud decide karta hai.
- `none` Ye LLM ko force karta hai ke koi bhi tool use na kare.
- `my_tool` Koi specific tool ka naam dena, jo LLM ko force karega ke wohi tool lazmi use kare. 

```bash
from agents import Agent, Runner, function_tool, ModelSettings

@function_tool
def get_weather(city: str) -> str:
    """Returns weather info for the specified city."""
    return f"The weather in {city} is sunny"

agent = Agent(
    name="Weather Agent",
    instructions="Retrieve weather details.",
    tools=[get_weather],
    model_settings=ModelSettings(tool_choice="get_weather") 
)
```

### 🔸Tool Use Behavior
- `tool_use_behavior` jab agent koi tool use karta hay tw uska output LLM ko kesy deya jaye usko control karta hay.
- `run_llm_again` Ye default setting hay, jab Tools run hota hay aur phr LLM unka result process karky final response banata hay.
- `stop_on_first_tool` Sirf Tool run hota hay, aur usi ka output user ko deya jata hay LLM dubara nh chalta.

```bash
from agents import Agent, Runner, function_tool, ModelSettings

@function_tool
def get_weather(city: str) -> str:
    """Returns weather info for the specified city."""
    return f"The weather in {city} is sunny"

agent = Agent(
    name="Weather Agent",
    instructions="Retrieve weather details.",
    tools=[get_weather],
    tool_use_behavior="stop_on_first_tool"
)
```

`StopAtTools(stop_at_tool_names=[...])` Agar koi spcific tool call hota hay tw ye usi waqt rukh jata hay aur usi tool ka output final response ky tor per use karta hay.

```bash
from agents import Agent, Runner, function_tool
from agents.agent import StopAtTools

@function_tool
def get_weather(city: str) -> str:
    """Returns weather info for the specified city."""
    return f"The weather in {city} is sunny"

@function_tool
def sum_numbers(a: int, b: int) -> int:
    """Adds two numbers."""
    return a + b

agent = Agent(
    name="Stop At Stock Agent",
    instructions="Get weather or sum numbers.",
    tools=[get_weather, sum_numbers],
    tool_use_behavior=StopAtTools(stop_at_tool_names=["get_weather"])
)
```

`ToolsToFinalOutputFunction`Ek custom function hota hay jo tools ky results ko process karta hay aur yeh decide karta hay keh LLm ko continue karna hay ya nh.

```bash
from agents import Agent, Runner, function_tool, FunctionToolResult, RunContextWrapper
from agents.agent import ToolsToFinalOutputResult
from typing import List, Any

@function_tool
def get_weather(city: str) -> str:
    """Returns weather info for the specified city."""
    return f"The weather in {city} is sunny"

def custom_tool_handler(
    context: RunContextWrapper[Any],
    tool_results: List[FunctionToolResult]
) -> ToolsToFinalOutputResult:
    """Processes tool results to decide final output."""
    for result in tool_results:
        if result.output and "sunny" in result.output:
            return ToolsToFinalOutputResult(
                is_final_output=True,
                final_output=f"Final weather: {result.output}"
            )
    return ToolsToFinalOutputResult(
        is_final_output=False,
        final_output=None
    )

agent = Agent(
    name="Weather Agent",
    instructions="Retrieve weather details.",
    tools=[get_weather],
    tool_use_behavior=custom_tool_handler
)
```

**Note:** Infinite Loops sy bachny ky leye framework automatically `tool_choice` ko "auto" per reset kar deta hay har tool ky bad, Infinite loop is wajah sy  hota hay tool ka result LLM ko wapas deya jata hay aur phr LLM `tool_choice` ki wajah sy dubara tool call generate karta hay aur yeh chalta rehta hay (ad infinitum).


---

### Resources & Practical Demos

- [OpenAI Agent Tools Colab Notebook](https://colab.research.google.com/drive/1JdnrvEEsK2pAIE6jmB5RAOXM6iunr15j?usp=sharing) → Interactive example with code and usage.

# OpenAI_sdk_agent_context_manager

[GoogleColab](https://colab.research.google.com/drive/1p4UnK59zsC5NxnmsQ49lRi3atK5i6wzv?usp=sharing)

## Context management

Context is an overloaded term. There are two main classes of context you might care about:

1. Context available locally to your code: this is data and dependencies you might need when tool functions run, during callbacks like on_handoff, in lifecycle hooks, etc.
2. Context available to LLMs: this is data the LLM sees when generating a response.


## Local context
This is represented via the RunContextWrapper class and the context property within it. The way this works is:

1. You create any Python object you want. A common pattern is to use a dataclass or a Pydantic object.
2. You pass that object to the various run methods (e.g. Runner.run(..., **context=whatever**)).
3. All your tool calls, lifecycle hooks etc will be passed a wrapper object, RunContextWrapper[T], where T represents your context object type which you can access via wrapper.context.
4. The most important thing to be aware of: every agent, tool function, lifecycle etc for a given agent run must use the same type of context.

You can use the context for things like:

- Contextual data for your run (e.g. things like a username/uid or other information about the user)
- Dependencies (e.g. logger objects, data fetchers, etc)
- Helper functions

Note:

The context object is not sent to the LLM. It is purely a local object that you can read from, write to and call methods on it.






<code>

import asyncio
from dataclasses import dataclass

from agents import Agent, RunContextWrapper, Runner, function_tool

@dataclass
class UserInfo:  
    name: str
    uid: int

@function_tool
async def fetch_user_age(wrapper: RunContextWrapper[UserInfo]) -> str:  
    return f"User {wrapper.context.name} is 47 years old"

async def main():
    user_info = UserInfo(name="John", uid=123)

    agent = Agent[UserInfo](  
        name="Assistant",
        tools=[fetch_user_age],
    )

    result = await Runner.run(  
        starting_agent=agent,
        input="What is the age of the user?",
        context=user_info,
    )

    print(result.final_output)  
    # The user John is 47 years old.

    if __name__ == "__main__":
    asyncio.run(main())
 

</code>



[for more detail](https://openai.github.io/openai-agents-python/context/)

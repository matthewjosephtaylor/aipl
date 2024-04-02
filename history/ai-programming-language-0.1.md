# AI Programming Language
[blame: Matt Taylor](https://mjt.dev)

[previous versions](./history/)

current version: 0.1

A simple programming language for LLM driven AI Agents.

## Definition of terms

LLMs (Large Language Models) are _effectively_ AI (Artificial Intelligence) as most people imagine and use that term.

Agents are LLMs personified. 

The chat/message-passing paradigm for intereacting with LLMs is currently the best mechanism for interacting with LLMs. This 'chat' style and allows users to have intelligent thoughtful conversations with Agents, or give direction to Agents to perform tasks, allowing the Agent to have a sense of the conversational context.

The industry has standardized on three main 'roles' for each message: `system`, `user`, and `assistant`.

The `user` and `assistant` role messages are self describing. The `system` role messsage is the message responsible for giving direct instruction to the LLM.

There are typically one or more system messages within the context of the conversation.

System message content can be as simple as 'You are a helpful AI that answers all questions truthfully' or as complex as semi-structures RAG query results the LLM should use for answering questions. The only true limits are those imposed by context size (how many tokens the LLM can use) and the power of the LLM to correctly understand and interpret the information.

## Why

Agents are uncanny in their ability to adopt a given personality if it is described well enough. LLMs in a way are the ultimate mimics.

This mimicry is often good enough for simple conversations and tasks as the user will provide the LLM with enough motivation and conext in their messages they send through the 'chat window' for the LLM to respond in a reasonable manor.

The problem comes when the Agent author (the one who crafted the Agent) wants to be in more control of the Agent.

Often the Agent author will want the Agent to behave in different ways depending on how the conversation progresses.

Perhaps the Agent is to be a salesperson, and the author has a clear idea of how a salesperson working on their behalf should go about the business of selling.

In the beginning stages of the conversation the author wants to insure that the Agent follows a kind of 'sales script' as many human salespeople currently follow.

Perhaps the sales script looks like:
- Greet the prospect
- Gather information about how the prospects day is going, and if they have a favorite sports team
- Gather information on what the prospect's needs are
- Communicate how the Agent's employer can help with their needs.
- Setup a future meeting with a specific time and date of the prospects choosing within a certain availability window.

Today with standard prompt engineering it is possible to create an Agent that performs some of these tasks but likely not all.

Agents will often 'wonder' and stray from the Agent authors intent during the conversation as the only feedback the Agent has during the conversation is from the user, NOT the Agent author.

AIPL attempts to address this imbalance, and give the Agent author more of a direct presense and better control as the conversation is happening. This is achieved by giving the Agent author the tools to update the Agent as the conversation progresses.

## What

Agents use LLMs with sophisticated system prompts that are rendered in 'real time'.

The AI Programming Language (AIPL) is a means by which to efficiently manipulate system prompts as an Agent conversation evolves over time.

A simple mustache-like case-insensitive templating language is used for rendering variables for prompt-templates. These rendered prompt templates are used within the live chat context via system-messages.

AIPL provides a simple syntax for updating state variables and executing conditional logic as the conversation is happening. This real-time updated state can be used inside system message prompts to update the chat context, to provide additional information and guidance, similar to how one might query a vector database for RAG, or change the Agents personality or goals by updating the system messages that describe these details.

There is a balance in AIPL between `hard code` (where syntax rules must be obeyed and the outcome is predictable) and `soft code` (where the LLM is used to interpret programmer intent or answer questions, and the outcome is creative and somewhat unpredictable)

AIPL allows the Agent author the ability to create an Agent that can respond to the user in different _controllable_ ways, depending on how a conversation evolves with the user, or other outside events that happen during the course of the conversation.

## State, Events, Logic

State, events, logic handling and syntax are the core aspects of any programming language. AIPL is no different. Unlike traditional programming languages however it strives to utilize the power of LLMs as much as possible to 'flesh out' intentions of the developer where it makes sense to do so. 

## State

State is captured in as series of values associated with a given conversation. It can be read from as template variables, and can be updated by calling out to the LLM to answer questions, or evaluate 'soft code'.

- State is global and mutable within the conversation
  - All state is stored within the conversation and ordered by creation time. The last stored value within a given namespace/key pair is used within prompt templates.
- State is referened inside a prompt-template as `{namespace.key}`
  - all values are accessed via namespace and key
  - all values are natively stored as strings
  - values can be interpreted within the language as a string, number, or boolean depending on context, by asking the LLM how the data should be coerced.

### Updating State

The basic AIPL state update syntax follows the forms:

#### conditionally update the state:
`(expression ? question -> identifier)`

#### update the state unconditionally:
`(= question  -> identifier)`

`identifier` is the full name of the state variable including namespace and key.

`expression` follows c-style expression semantics where boolean algebra and comparison operators are available.

`questions` are simple questions asked to the LLM given the context of the conversation message history, excluding any other system messages. The answers to these questions are stored in the chat state. 

Update State Example:

```
(# Examples of setting the chat state)

(# if the character has children then figure out what the wife's name is)
((userInfo.children > 0) ? "What is {char}'s wife's name?" -> userInfo.wifeName)

(# always attempt to figure out where the user went to school)
(= "Where did {user} go to school?" -> userInfo.school)

```

Note that AIPL assumes an LLM with sufficient power to answer the question:

```
How would the following function evaluate if it MUST return true or false?

happy("John is upset at Diane after she stole his milk")


```

NOTE that these LLM driven 'soft code' evaluations are provided WITHOUT full conversation context, with the expectation that they could be evaluated on multiple LLM inferences _simultainiously_ with a majority-wins style strategy to improve reliability.

AIPL leverages the power of the LLM to be creative in its responses and interperate the _meaning_ of function names without prior definition -or- with some minimal natural language definition if provided.

This mixture of 'hard' and 'soft' code allows for fast development as no 'real code' is ever written for 'soft code'. This is especially useful for what might be a difficult nuanced evaluation of the present state of a conversation. As LLMs improve over time it is expected that the same code would become more and more reliable.


### Prompt Template

A simple mustache-inspired forgiving syntax style. Note that either `{namespace.key}` or `{{nameSPACE.KeY}}` are valid and equivalent.

If no default value is present then undefined values are rendered as an empty string.

Default values are optional and are declared via a colon character after the variable like `{namespace.key:default value}`.

Only alpha-numeric characters are allowed for namespaces or keys following the regex pattern: `[a-zA-Z0-9]`.

The following template variables are special system provided variables:

{char} = the character/agent name attached to the prompt template
{user} = the current user role chat participant name
{assistant} = the current assistant role chat participant name 
{date} = the current date
{time} = the current time

#### Basic Prompt Template Example:

```
{char}'s mood is {mood.recent:happy enough} and they currently have {cookie.count:12} cookies. They are aware of the following dangers: {environment.dangers}

{char} is aware that the current date is {date} and the current time is {time}

```

#### Advanced Prompt Template Example:

```
This sentence will always be rendered.

(# this is a comment that will not be rendered)

((isAngryConversation({conversation.summary}) && isCalmCharacter({userInfo.personality})

This will only be rendered if the condition above is true
The LLM will evaluate the above 'soft code' functions and provide a boolean response

  ((isEvening("The current time is {time}")

    This nested sentence will only be rendered if both conditions are true

    (# attempt to figure out where the user went to school this will only be updated if both conditions above are true)
    (= "Where did {user} go to school?" -> userInfo.school)

  )


)

This sentence will also always be rendered


(# This will update the state constantly as the conversation progresses if the expression is true)
((userInfo.children > 0) ? "What is {char}'s wife's name?" -> userInfo.wifeName)

(# note that userInfo.children will be coerced into a number by the LLM as best as it is able)

```


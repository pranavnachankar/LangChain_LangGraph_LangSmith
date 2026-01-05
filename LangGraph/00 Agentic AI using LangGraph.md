# Agentic AI using LangGraph
----------------------------

# LangChain Recap:
------------------
- LangChain is an open-source library designed to simplify the process of building LLM based applications.
- It provides modular building blocks that let you create sophisticated LLM-based workflows with ease.


# LangChain consists of multiple components:
--------------------------------------------
1. Model components gives use a unified interface to interact with various LLM providers
2. Prompts component helps you engineer prompts
3. Retrievers component helps you fetch relevant documents from a vector store

But the biggest offering of LangChain is Chains.

# What can you do with LangChain
1. Simple conversational workflows like Chatbots, Text Summarizers
2. Multistep workflows
3. RAG applications
4. Basic level agents


here we understood how LangChain works, let see can we able to implement automated workflows with this LangChainor not

consider `Automated Hiring workflows`
```start -> hiring request -> create JD -> JD approved / not -> post JD -> wait for few days to collect applications ->  are enough job applications received? yes/no -> if no, then modify JD -> shortlist resumes -> conduct interview -> selected/rejected -> offer letter rolled out yes/no -> renegotiate -> onboarding -> end```


# Agentic AI vs workflows:
-------------------------
**Workflows** are systems where LLMs and tools are orchestrated through predefined code paths.
**Agents**, on the other hand, are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks.


LangChain we are consider to make a linear workflow but for Agentic application this is non-linear or dynamic


# Challenges in Langchain:
-------------------------
1. Control Flow Complexity:
    - conditional branching
    - loops
    - jumps (no fix flow, it could be forward/backword based on the multiple situations)
    - because of this you have write a custom code which leads a glue code into the picture and its not a good practice or it should be very less

- With LangGraph we can create a graph structure of the flow (i.e. nodes) and easy to manage
- Can connect nodes using edges

2. Handling State:
    - no direct facility to store `states` default in LangChain

- LangGrpah offer built in feature to manage `states`

3. Event Driven Execution:
    - Sequencial {LangChain is based on this functionality}
    - Event Driven (wait for trigger of approval/scheduled timeline)  <-- {LangGraph offer this}


4. Fault Tolerance:
    - small (like particular flow failed then that only needs to retry instead entire flow)
    - big (system crashed then LangGraph offer recovery with checkpoints so all time state is getting managed and which ever flow got failed from that execusion will get resume)


5. Human-in-the-loop:
    - stateless in LangChain so can't store the state
    - stateful in LangGraph and can be there for longer duration 
    - Check https://docs.langchain.com/oss/python/langchain/human-in-the-loop
    - fun example : you are playing video game and suddenly paused it then you are resume at any moment after few minutes/hours/days

6. Nested Workflows:
    - challenging implementation in LangChain
    - can use sub-graphs in LangGraph -->> https://docs.langchain.com/oss/python/langgraph/use-subgraphs

7. Observability
    - Observability refers to how easily you can monitor, debug, and understand what your workflow is doing at runtime.
    - No tracking of Glue Code
    



# LangGraph Core Concepts
---------------------------
## What is LangGraph
    - LangGraph is an orchestration framework for building intelligent, stateful, and multi-step LLM workflows.
    - It enables advanced features like parallelism, loops, branching, memory, and resumability - making it ideal for agentic and production-grade Al applications.
    - It models your logic as a graph of nodes (tasks) and edges (routing) instead of a linear chain.


# LLM Workflows
---------------
1. LLM workflows are a step by step process using which we can build complex LLM applications.
2. Each step in a workflow performs a distinct task - such as prompting, reasoning, tool calling, memory access, or decision-making.
3. Workflows can be linear, parallel, branched, or looped, allowing for complex behaviours like retries, multi-agent communication, or tool-augmented reasoning.
4. Common workflows:\
    a. **Prompt Chaining**\
    b. **Routing**\
    c. **Parallelization :** here the LLM know the way to perform the task like if agent to upload video over YouTube then one LLM will check for video fits in terms of aspect ratio, bits, etc etc; another will check the community guidelines\
    d. **Orchestrator Worker :** no fixed flow, like if query is related to research then Orchestrator LLM will decide which LLM will run first and what rest parallel actions are needed.\
    e. **Evaluator Optimizer :** Here flow is like user i/p will come and based on that LLM call will be generated and solution is provided but LLM Evaluator will evaluate and either reject or give feedback and proceed further. 


# Graph, Nodes, Edges:
lets take a example\
The system generates an essay topic, collects the student's submission, and evaluates it in parallel on depth of analysis, language quality, and clarity of thought. Based on the combined score, it either gives feedback for improvement or approves the essay.
1. **Generate Topic**
    - System generates a relevant UPSC-style essay topic and presents it to the student.
2. **CollectEssay**
    - Student writes and submits the essay based on the generated topic.
3. **EvaluateEssay (Parallel Evaluation Block)**
    - Three evaluation tasks run in parallel:
        - **EvaluateDepth** - Analyzes depth of analysis, argument strength, and critical thinking.
        - **EvaluateLanguage** - Checks grammar, vocabulary, fluency, and tone.
        - **EvaluateClarity** - Assesses coherence, logical flow, and clarity of thought.
4. **AggregateResults**
    - Combines the three scores and generates a total score (e.g., out of 15).
5. **ConditionalRouting**
    - Based on the total score:
        - If score meets threshold - go to ShowSuccess
        - If score is below threshold go to GiveFeedback
6. **GiveFeedback**
    - Provides targeted suggestions for improvement in weak areas.
7. **CollectRevision (optional looр)**
    - Student resubmits the revised essay.
    - Loop back to Evaluate Essay
8. **ShowSuccess**
    - Congratulates the student and ends the flow.


# State
    - In LangGraph, state is the shared memory that flows through your workflow
    - It holds all the data being passed between nodes as your graph runs.

# Reducers
    - Reducers in LangGraph define how updates from nodes are applied to the shared state.
    - Each key in the state can have its own reducer, which determines whether new data replaces, merges, or adds to the existing value.


# LangGraph Execution Model
-----------------------------
insprired by [Google Pregal](https://www.google.com/search?q=google+pregel)

1. Graph Definition
You define:
    - The state schema
    - Nodes (functions that perform tasks)
    - Edges (which node connects to which)
2. Compilation
    - You call `.compile()` on the `StateGraph`.
    - This checks the graph structure and prepares it for execution.
3. Invocation
    - You run the graph with `.invoke(initial_state)`.
    - LangGraph sends the initial state as a message to the entry node(s).
4. Super-Steps Begin
    - Execution proceeds in rounds.
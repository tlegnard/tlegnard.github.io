---
title: "Building My First AI Agent"
date: 2025-06-03T14:24:46-04:00
draft: false
---

### Diving into Agentic Workflows

There's nothing quite like the quiet hours of parental leave, when you can get them at least. When either feeding my newborn son or catching up on housework, my mind tends to wanter to interesting technical problems. I had read been reading blog posts about Anthropic's Model Context Protocol (MCP) and the fast-growing world of AI agents. Between diaper changes and naps, I decided to build something practical with AI, help me study to go on Jeopardy!

I had downloaded and stored some historical data from J-Archive! in a SQLite database, using the SQLite UI or [duckdb](https://duckdb.org) to write queries to pull facts and quiz myself in hopes of one day being the next Ken Jennings or James Holzhauer. And while I knew the database I had built well enough to be able to write the queries to get questions of specific categories, it was clunky to have to constantly update where clauses, I wanted an easier appraoch to extract information out of my data store using natural language. I had found a perfect use-case to build my first AI "agent" / agentic workflow / LLM assistant. It was time to go down the MCP rabbit hole. I figured it might be useful for anyone aspiring to wrap their heads around this new ecosystem.

I set off on building a basic text-to-sql agent. Thinking I had bit off more than I could chew by learning a new protocol to try and solve one of the most ambitious problems in the data world. Here's what I learned building the system, and how you can try it yourself.

### What is MCP?

[Model Context Protocol (MCP)](https://github.com/modelcontextprotocol) is an open standard that enables LLMs to connect to external data sources and tools. Unlike traditional setups where you might send your database queries to external services, MCP works like an API to connect an LLM to a "server" that allows the model to make tool and functions calls to some other system in the backend. MCP serves as the glue between an LLM and the software you want it to interact with.

The MCP "servers' work as interfaces that allow the LLM to take a user prompt and translate it to JSON that can be sent to a server to call functions, access a data source, or run custom functions. It's the leading "API" for LLMs in the current AI landsacape.

### What is an AI Agent?
That's a good question, and there seems to be multiple answers to it right now. Right now I like the definition that David Crawshaw [gives on his blog](https://crawshaw.io/blog/programming-with-agents), as it's the most straightforward and devoid of the AI hype. 
>For someone with an engineering background there is now a straightforward definition: an agent is 9 lines of code. That is, an agent is a for loop which contains an LLM call. The LLM can execute commands and see their output without a human in the loop.
>
>That’s it. Like all simple things your instinct, quite reasonably, could be to say “so what?” It is a for loop. But the results are an astonishing improvement on the power of a raw large language model.

The Aaents you see doing coding projects out there seem to be LLMs that are given tools and logic flows, with a UI slapped on to interface with a user. They are deterministic programs with a probabilistic compliler (the LLM) used to interpret subjecctive human input into objective output (code). This particular agent that I've built is more than 9 lines of code, but it is also utilizing more than one "agentic" workflow, that is the LLM is being called upon to use a tool to complete a task, similar to how you would have a Data Analyst write some SQL for you.


## Building the agent.
For this demo I used MCP's sqlite-server*. I also wanted a local solution, running from my desktop. I did not want to leverage cloud services like AWS's Bedrock or GCP's Vertex AI platforms, before I start churning thorugb tokens like it's no tomorrow I wanted to make sure I could run prompts to my heart's content, even if I had to compromise on speed and reliability a little bit. Full disclosure, I did "vibe code" parts of it, but I used Claude to facilitate writing a lot of the intitial code, it was long-winded, so I spent a bit of time refactoring and packaging tool calls into functions to make it more readable and less redundant, as it goes when using LLMs to code.

I installed `ollama` on my macbook and downloaded Meta's `llama3.1` model, and started it up with `ollama serve`. I then borrowed some code from a repo [Mike Chambers]((https://www.youtube.com/watch?v=u6444EjemKo&t=4s)) had put together, but removed the bits connecting to Amazon Bedrock service so I could leverage a small model locally.

The key bit from here was writing some functions and logic flow that sends prompts to the LLM, and provides guardrails that give the LLM more instruction on what to do when specific commands are given, or when a natural lanaguage prompt is provided.

I decided to use streamlit as a frontend UI for this prototype, so I included do separate input boxes, splitting the app into two main modes.

Direct SQL Command - For when you want to execute specific SQL operations
Natural Language Query - For conversational database interactions



  \* As of 5/29/2025, this server has been archived on github and is now read-only, it should still work for the demo however, but I will also include a copy of the code in my [demo-app link](https://github.com/tlegnard/local-text-to-sql)


### Understanding the Workflow
The demo is intentionally transparent about its process. You can see exactly how the LLM interprets your natural language and converts it into structured SQL commands. In the "Agent Response" section, you'll see the raw tool call that the model generates, showing the decision-making process.

### Getting Started: Essential Commands
The interface helpfully shows you exactly what commands you can use. Before diving into complex queries, start with these fundamental operations:
Direct Commands

`list_tables` - List all tables in the database

`describe_table` table_name - Show schema for the specified table

`read_query SELECT * FROM table LIMIT 10;` - Execute SQL query

### Natural Language Examples
The demo includes sample questions that work well:

"What are all the categories in the database?"

"Show me the first 5 questions from the Double Jeopardy round."

"How many unique contestants are in the database?"

The LLM can connect the dots of what tables to use, and what fields to include in the results based on the keywords given. Using "the first 5" implied a `LIMIT` clause of 5 records.

We can see from the screenshot a specific use-case with my Jeopardy dataabase:

![The app in action.](/media/text-to-sql-modes.png)
![The query results.](/media/query-results.png)
  
### Why SQL Keywords Matter
The LLM has been trained to recognize SQL-like language patterns. When you use familiar database terminology in your prompts, you're essentially giving the model context clues about what tools it should invoke. Words like "describe," "list," "show," "select," "join," etc., help the model understand your intent and choose the right approach.

Given I'm using a LLama 3 model on the lower end of tokens, prompts have to be more specific. "Give me three clues fromt the Science category?" should be shifted to sound more "SQL-like", and referencing specific columns and tables in the database the MCP server is connected to.

> "Provide three clues from the clues table with category like "science"

Yes, it may be easier to just write the SQL myself: `SELECT * FROM clues where category like '%SCIENCE%' limit 3`. But I don't want to have to modify it every time if I'm exploring other categories, or if I don't know any SQL. I should be able to riff with the model and have it serve the data back to me, even if in its current state it takes a few tries or a bit of crafting of the original prompt. There certainly could be improvements here by using a larger model and plugging into agent, or distilling a customer model that's been trained on historic SQL queries written against a database.

### What's Next?
This demo represents just the beginning of what's possible with this localized text-to-SQL agent, and gives me a lot of ideas for how I could improve this particular setup, and spawn some new agents in the future. Here's what I hope to tackle next.

- Allow for faster response times through model optimization
- More sophisticated query planning
- Better error handling and suggestions
- Integration with more database types other than SQLite
- Play around with distilling an LLM to be focused on writing SQL, I have more to research and learn there.

The goal is to make database querying as natural as having a conversation, while keeping your data completely private and secure. As far as I know llama running locally keeps data local.

Try the demo yourself and let me know what you think! The source code is available on [Github](https://github.com/tlegnard/local-text-to-sql).
Note you may have to shallow clone the archived SQLite MCP server from [here](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sqlite)

You'll also need to install `uv`, [setup an MCP server locally](https://docs.anthropic.com/en/docs/agents-and-tools/mcp-connector)  and have a SQLite database handy to connect the tool with.

### References
- [Will Anthropic's MCP work with other LLMs? - YES, with Amazon Bedrock.](https://www.youtube.com/watch?v=u6444EjemKo&t=4s)
- [amazon-bedrock-mcp](https://github.com/mikegc-aws/amazon-bedrock-mcp)
- [MCP sqlite server (archived)](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sqlite)
- [How I program with Agents](https://crawshaw.io/blog/programming-with-agents)
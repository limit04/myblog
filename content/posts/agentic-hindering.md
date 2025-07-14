---
title: "Agentic Hindering: Faulty First Principles"
date: 2025-07-14
draft: false
tags: ["agents", "systems"]
categories: ["meta"]
description: "Hindering is being unable to do something due to external factors."
---

# Agentic Hindering: Faulty First Principles.

“_Hindering_” is being unable to do something due to external factors.

Recently I was working on an AI Agent which has the ability to use a tool to query a database, which would return relevant chunks. Basically a RAG(Retrieval Augmented Generation) Agent. This agent only had one tool and one task which is to just query the database → get the chunks → Answer the users questions.

I made its focus to be on answering questions of users who are developers of the codebases. So a backend developer may ask questions about data handling in the frontend and vice versa. The Agent was supposed to help the developers who had knowledge about the codebase and wanted to do something new with it or understand it.

So for testing I indexed two repos: a React Frontend and a Go Backend. And I got a few questions from the Developers who actually worked on the codebases.

## Backend:

So while testing a question by the Backend dev which was about adding a new feature and one of the relevant files was a `booking.proto` file in which we were supposed to add a new variable. So the Agent while exploring the codebase which was primarily in GO has figured out the existence of a `.proto` file.

So as a query it gave “Booking Status booking.proto”.

Now in normal situations this is a good query but the Intention of the LLM was to query the file `booking.proto` and only the file. But querying embeddings works in such a way that the results returned are semantically similar to the query, which are not an answer to the query. The returned chunks had words such as Booking, Status etc but there was no booking.proto file as the LLM expected.

So it queried again “enum BookingStatus booking.proto”

Again It was a failure. Although it returned more chunks it did not return the chunk LLM wanted. This went on for a few iterations and the LLM gained so much knowledge that its queries started getting more detailed and more far away from optimal queries.

One query even had an exact full path like `pb/bookign.proto`. Still It was a failure.

## Frontend:

As expected with frontend results were no better. When received a complex question which points to a `Calendar.tsx` file, the LLM desperately tried to query the file `components/Calendar.tsx` with queries such as “Notification handling in Calendar.tsx" or “Notification components/Calendar.tsx”.

As we observe the queries kept getting more and more detailed to the point where if you and I have the codebase we would easily get to the answer.

## Wounded Agent:

The Agent in above cases is only limited because of the capability we provided. The tool, which it expected to work for the queries it generated and kept on making queries more detailed kept on failing. And after a few searches the Agent gave up and started providing answers with a tone such as **likely**, **maybe** which did not emphasize confidence in its response.

### Why?

**Why was this happening?**

While building an Agent which tries to help people we often focus on how the Agent should respond mainly thinking oh how the human may respond if faced with the same question.  
But the main point of concern should be the process this human takes in answering the question. What inherent knowledge does he have? How is he searching in the codebase?

We again come back to First Principles.

“_First Principles_” thinking breaks down true understanding into building blocks we can reassemble into something that simplifies our problem.

Here our humans may use String Search when they know the exact function to look for.  
They may look at README.md to understand the codebase.  
They may look at test files when faced with test case related questions.  
**They are targeting specific things.**

One primary issue is that my tool definition was too vague. The tool definition I provided was something similar to:

```text
Use this tool to search the codebase.

Inputs:
query : \[str\]
```

(Exact Definition was more Detailed than this)  
The vague info about the tool made the LLM assume it had a greater capability of search.

**Fix:**

Well Fix\#1 was to just update the tool definition and make it more detailed.

```test
User this tool to search the codebase **semantically**.
**DO NOT** use file names in queries.
```

Just some simple definition fixing fixed all the issues faced in Fronted queries. The LLM started targeting function names in queries and it got relevant chunks. Perfect.

But the backend proved tricky as the files generated from .proto also had same definitions.  
But Going back to First Principles teaches us that humans target specific files.  
Which is exactly what our agent tried to do but our system Hindered it.

**Fix:**

Allow the LLM to target file extensions such as .proto, .pb.go like humans. Simple and minimal but highly effective

Aside from this agent, we also see Agentic Hindering in **cursor** where time and time again the Agent tries to read a .env file or a .env.local file etc which are ignored by cursor so Agent cannot access it.

In a few cases the agent overcame it by using terminal commands to create it using touch commands, echo commands etc. This generally happens after multiple repeated failures of trying to read the file.

## Conclusion:

Originally this blog’s title was Agentic Scream as the repeated failures and confusion the Agent was put through made it seem like the Agent was screaming. I need to focus more on how the Agentic System differs from a Normal System(Humans). The difference should always produce a better, more efficient solution rather than providing a sub-par, imperfect imitation of the Normal System. I need to focus more on how Humans do it and where they lag behind. Both of these helps in making a better Agent.

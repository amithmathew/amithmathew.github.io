---
title: "Random Thoughts reflecting on 18 Years of Data and Analytics work"
date: "2025-03-11T13:54:52-04:00"
description: "Random Thoughts reflecting on 18 Years of Data and Analytics work"
tags: ["analytics", "reflections"]
---

I've seen all kinds of enterprises, analytical data platforms and data pipelines over the last 18 years or so. I thought it might be an interesting exercise to write down some lessons that I've learnt as well as some thoughts about where we're headed. Some (most?) of this is common sense, but common sense apparently takes a vacation when deadlines loom and everything is on fire. This __is__ a living list, and I expect to add more points to it over time.

## Thought 1: Data Platform Design - Stop Fighting Your Org Chart

Let's talk about Conway's law - 

> Organizations which design systems (in the broad sense used here) are constrained to produce designs which are copies of the communication structures of these organizations.
> - Melvin E. Conway, How Do Committees Invent?

I used to think Conway's Law was a Bad^tm^ thing - "Your dysfunctional enterprise architecture is because of your dysfunctional enterprise!". However, when actually designing enterprise wide systems, it's better to embrace Conway's Law rather than fight against it. Design these systems to reflect your existing org chart, warts and all. Enterprise architecture must follow organizational structure, not lead it. Aspirational architecture (_are Data Meshes still cool?_) is a recipe for disaster. Your IT team will constantly be battling your system's natural tendency to devolve to reflect organizational structure, often in unintended and inefficient ways. Much better to design for this up-front where you control the outcome. Stop trying to outsmart Conway's Law, embrace it.

## Thought 2: SQL will never die

Many have come for the King, and all have missed. 

SQL for "read" workloads will never die. DML and DDL are a different story, and might be replaced by alternatives at some point in the future (not now though - see BigQuery steadily [adding more DDL support](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language)!). SQL for Read is just efficient - you declare to the SQL engine what output you want and the engine figures out how to get it for you. Just look at how our industry is moving back to SQL -  
* Most Enterprise Spark workloads use SparkSQL
* The Hadoop ecosystem went from MapReduce and Java to Hive and other SQL-like interfaces
* Databricks is focussed on building a robust SQL layer with Databricks SQL
* Rapid adoption of DBT and other SQL based data pipeline tooling
* BigQuery and Snowflake remain wildly successful

The most common complaint against SQL that I've heard is that the actual SQL Engine is a magic black box, which turns some programmers off, and is sometimes a bad thing in system design. But assuming you're following most basic best practices around data modelling, it works for 99% of the use cases I've seen. And for the remaining 1%, you go find somebody who knows how to open up that box and tweak the knobs to make it perform.

## Thought 3: Happier on-call engineers, better system design

It's extremely easy to over-engineer and build data platforms and systems that consider all possible edge cases. "Maybe we'll need to stream all of our ERP data into our Datalake at some point, so let's just build out some streaming capabilities, just in case."

Shops that I've seen do this have usually seen limited success with their design - sometimes limited adoption due to higher use case onboarding friction, and more frequently, operational brittleness once they go to prod - all of which can be traced back to complexity. 

Unnecessary complexity in system design is a very bad word, it's funny how often this needs to be stated. Simplicity wins, every time. The Pareto principle is your friend - build for 80% of your use cases (weighted by some measure of importance to the enterprise). You can figure the remaining 20% out later, and your design will be better for it. Build the simplest thing that works. Let the use cases coming your way force you to walk backwards into complexity.

Lean Manufacturing includes a concept called Just-In-Time manufacturing, the idea being to minimize inventory within your supply chain. In-process inventory is just tied up capital that could potentially be better deployed elsewhere. Think Just-In-Time data architecture, stop hoarding features you don't really need.

Now you can take this too far and not leave enough room to grow. Simplifying your architecture to the point where you no longer have any optionality for future capabilities is foolish. But don't confuse 'optionality' with 'overkill'. The key job of a good enterprise data architect must be to find the sweet spot on this Complexity - Optionality curve.

## Thought 4: Garbage In, Garbage Out

I've occasionally seen companies that deemed their data platforms a failure because the quality of data coming out of the platform was garbage, and the data was unusable. In most of those cases, their users went rogue - bypassing the platform entirely and going straight to the source for their data, so that they could control the quality of the data they were receiving. The mistake these organizations made was to assume that technology was a magic wand that would fix their data governance challenges.

Clean data starts with people, not code. Clear data ownership and standardized definitions and measures of data quality is important and worth spending time and resources over. Once you have this, then you can look at a technology solution to enforce it. People and process come first here, technology should follow.

If you want an enterprise-wide, authoritative, data platform you must invest in people and process to support data governance. 

## Thought 5: Generative AI = Universal Data Literacy

Business Intelligence is a band-aid for data literacy - a translator for the data challenged. They're nothing but a way for an organization to provide "end users" - aka people who actually make material decisions or take action -  with the information they need to execute said decision or action. These people have traditionally had better things to do with their time than learn SQL or the underlying technical data model. BI tooling allows these users to ask questions of data, to make better decisions and take more effective action. Most larger enterprises I've worked with have had data democracy and data literacy initiatives. Data democracy was to give more "executors" access to the data. This had to go hand in hand with Data Literacy - teaching more "executors" HOW to use tooling, BI or otherwise, to be able to extract the data they need. 

Generative AI and Language models are an amazing enabler for this. Being able to query your data in natural language, which is truly the universal query language - eliminates the concept of data literacy entirely. All your employees are now data literate. Your short term focus should now shift to ramping up data governance and quality efforts to make sure that when these employees start accessing their data, they get high quality, governed results.

Natural Language to SQL isn't here yet, but it's coming - all major data platform providers are working on it. While we're not there yet, we're close enough that you can catch glimpses of it through the trees. When it happens, this will be a step change in how organizations use and consume data.


# Antigravity SDK — Hooks, MCP, and Policies Notes

 

This section covers three advanced/production-level concepts in Google Antigravity SDK:

 

```text

1. Policies  → control what the agent is allowed to do

2. Hooks     → observe, intercept, or modify agent lifecycle events

3. MCP       → connect external tools, APIs, databases, and services

```

 

Recommended learning order:

 

```text

Policies → Hooks → MCP

```

 

Why this order?

 

Because once an agent gets powerful external tools through MCP, you need **policies** and **hooks** to keep execution safe, auditable, and controllable.

 

---

 

# 1. Policies

 

## What Are Policies?

 

Policies define what an agent is allowed to do.

 

They control:

 

```text

Which tools can run automatically

Which tools need human approval

Which tools should be blocked

```

 

Think of policies as:

 

```text

Security guard for tool execution

```

 

Antigravity SDK includes safety/governance examples for deny-by-default, allowlisting, and `ask_user` behavior. 【1-e9821b】

 

---

 

## Why Policies Matter

 

Agents can call tools.

 

Some tools are safe:

 

```text

view_file

list_directory

calculate_roi

get_customer_details

```

 

Some tools are risky:

 

```text

run_command

write_to_file

update_database

send_email

delete_file

deploy_to_production

```

 

Policies decide whether these tools should be:

 

```text

Allowed

Denied

Allowed only after user approval

```

 

---

 

## Basic Policy Pattern

 

```python

from google.antigravity.hooks import policy

 

policies = [

    policy.deny("*"),

    policy.allow("view_file"),

    policy.allow("list_directory"),

    policy.ask_user("run_command"),

]

```

 

Meaning:

 

```text

deny("*")               → block everythin* by default

allow("view_file")    * → allow file reading

allow("list_*irectory") → allow directory listi*g

ask_user("run_command") → ask hu*an before shell execution

```

 

Thi* is called:

 

```text

Deny by defau*t

```

 

It is safer than allowing e*erything first.

 

---

 

## Policy Me*tal Model

 

```text

User asks task

* ↓

Agent decides to call a tool

  *

Policy checks the tool

  ↓

Policy*returns allow / ask / deny

  ↓

Too* executes only if permitted

```

 

-*-

 

## Policy Design Strategy

 

A go*d policy setup usually looks like *his:

 

```text

Safe read-only tools*       → allow

Calculation tools  *        → allow

File write tools  *         → ask user

Shell commands*             → ask user

Database u*date tools       → ask user

Produc*ion deployment tools → deny or ask*senior approval

Dangerous delete t*ols      → deny

```

 

---

 

## Examp*e Policy Design

 

```text

view_file*       → allow

list_directory   → *llow

calculate_roi    → allow

run_*ommand      → ask_user

write_to_fi*e    → ask_user

delete_file      →*deny

deploy_prod      → deny

```

 

*--

 

## When Should You Use Policie*?

 

Use policies whenever your agen* can:

 

```text

Read files

Write fi*es

Run shell commands

Update datab*ses

Send emails

Call external APIs*Deploy code

Modify production syst*ms

Access sensitive data

```

 

Anti*ravity SDK gives agents built-in c*pabilities such as file I/O, code *diting, shell execution, directory*search, and other tool-based actio*s, so permission control is import*nt.

 

---

 

# 2. Hooks

 

## What Are*Hooks?

 

Hooks allow you to run cus*om logic at specific points in the*agent execution lifecycle.

 

Think *f hooks as:

 

```text

Middleware fo* agent execution

```

 

They are use*ul for:

 

```text

Logging

Monitorin*

Auditing

Safety checks

Tool appro*al

Data sanitization

Diagnostics

E*ror handling

```

 

Antigravity hook* can intercept, observe, and modif* agent behavior at different lifec*cle stages. They are useful for ob*ervability, policy enforcement, da*a sanitization, and interactive de*ision-making. 【2-53aa6c】

 

---

 

## *ook Mental Model

 

A web framework *iddleware flow looks like this:

 

`*`text

Request

  ↓

Middleware

  ↓

H*ndler

  ↓

Middleware

  ↓

Response

*``

 

An agent hook flow looks like this:

 

```text

User message

  ↓

Hook

  ↓

Model call

  ↓

Hook

  ↓

Tool call

  ↓

Hook

  ↓

Final response

```

 

---

 

## Hook Categories

 

Antigravity hook architecture has three major hook categories:

 

```text

Inspect Hooks

Decide Hooks

Transform Hooks

```

 

The SDK hook architecture classifies hooks into inspect, decide, and transform categories. 【2-53aa6c】

 

---

 

## 2.1 Inspect Hooks

 

Inspect hooks are:

 

```text

Read-only

Non-blocking

Used for observation

```

 

Use cases:

 

```text

Logging

Monitoring

Metrics

Audit trails

Debugging

Tool call tracing

```

 

Example use case:

 

```text

Log every tool call after it finishes.

```

 

Inspect hooks should not modify or block execution.

 

---

 

## 2.2 Decide Hooks

 

Decide hooks are:

 

```text

Read-only

Blocking

Used for approval or denial

```

 

Use cases:

 

```text

Permission checks

Security rules

Guardrails

Human approval gates

Tool access control

```

 

Example use case:

 

```text

Before running a shell command, check if the command is safe.

```

 

Decide hooks can allow or deny execution.

 

---

 

## 2.3 Transform Hooks

 

Transform hooks are:

 

```text

Modifying

Blocking

Used to change input/output or recover from errors

```

 

Use cases:

 

```text

Sanitizing sensitive data

Redacting API keys

Cleaning tool outputs

Recovering from tool errors

Prompt transformation

```

 

Example use case:

 

```text

Remove secrets from logs before storing them.

```

 

---

 

## Common Hook Events

 

Antigravity hook docs describe lifecycle events such as:

 

```text

PreToolUse

PostToolUse

PreInvocation

PostInvocation

Stop

```

 

These events allow custom logic before or after model/tool execution points. 【3-23dd20】

 

---

 

## Hook Events Explained

 

### PreToolUse

 

Runs before a tool is executed.

 

Use cases:

 

```text

Block dangerous command

Ask approval

Validate input

Check permissions

```

 

---

 

### PostToolUse

 

Runs after a tool completes.

 

Use cases:

 

```text

Log result

Run linter

Capture diagnostics

Store audit trail

```

 

Antigravity hook docs show hooks can run scripts or commands at lifecycle points, such as running a linter after tool usage. 【3-23dd20】

 

---

 

### PreInvocation

 

Runs before the model is called.

 

Use cases:

 

```text

Add reminders

Validate prompt

Sanitize user input

Inject policy context

```

 

---

 

### PostInvocation

 

Runs after model/tool interaction finishes.

 

Use cases:

 

```text

Log final response

Collect metrics

Analyze behavior

Perform cleanup

```

 

---

 

### Stop

 

Runs when the execution loop terminates.

 

Use cases:

 

```text

Cleanup

Final logging

Close resources

Write final audit record

```

 

---

 

## Practical Hook Examples

 

### Example 1: Logging Hook

 

Goal:

 

```text

Log every tool the agent tries to run.

```

 

Useful for:

 

```text

Debugging

Security audits

Production monitoring

```

 

---

 

### Example 2: Linter Hook

 

Goal:

 

```text

After code is edited, automatically run lint/tests.

```

 

Useful for:

 

```text

Code quality

CI-like validation

Preventing broken code changes

```

 

---

 

### Example 3: Safety Hook

 

Goal:

 

```text

Before shell execution, block dangerous commands.

```

 

Dangerous commands may include:

 

```bash

rm -rf /

sudo rm

drop database

git push --force

kubectl delete

shutdown

```

 

---

 

# 3. MCP

 

## What Is MCP?

 

MCP stands for:

 

```text

Model Context Protocol

```

 

MCP is an open standard that lets AI agents connect to local developer tools, databases, file parsers, external APIs, and remote services. 【4-db3385】

 

Simple definition:

 

```text

MCP = Universal connector protocol for AI agents

```

 

A useful analogy:

 

```text

MCP is like USB-C for AI agents.

```

 

---

 

## Why MCP Matters

 

Without MCP:

 

```text

Agent can only use built-in tools and custom Python functions.

```

 

With MCP:

 

```text

Agent can connect to external systems and use their tools/resources.

```

 

MCP can expose:

 

```text

Tools

Resources

Prompts

External data

Safe actions

Live system context

```

 

Antigravity docs describe MCP as a bridge that lets Antigravity fetch structured context directly and execute safe actions through connected servers. 【4-db3385】

 

---

 

## MCP Use Cases

 

MCP can connect agents to:

 

```text

GitHub

Notion

Linear

BigQuery

Cloud SQL

Supabase

Postgres

Google Docs

Logs

Internal APIs

Developer documentation

Business systems

```

 

Example tasks:

 

```text

Create a Linear issue

Search GitHub code

Inspect database schema

Query BigQuery

Fetch deployment logs

Search Notion docs

Read API specifications

```

 

Antigravity MCP docs mention MCP can add context and custom tools such as creating Linear issues or searching Notion/GitHub for patterns. 【4-db3385】

 

---

 

## MCP Flow

 

```text

User asks task

  ↓

Agent sees available MCP tools

  ↓

Agent selects MCP tool

  ↓

MCP server executes request

  ↓

MCP server returns result

  ↓

Agent uses result in final answer

```

 

Example:

 

```text

User:

Create a Linear issue for this TODO.

 

Agent:

Calls Linear MCP tool.

 

MCP Server:

Creates issue.

 

Agent:

Returns issue URL.

```

 

---

 

## MCP Server Types

 

Antigravity SDK examples show MCP integration using different transports:

 

```text

stdio

SSE

Streamable HTTP

```

 

The SDK supports MCP server configurations such as `McpStdioServer`, `McpSseServer`, and `McpStreamableHttpServer`. 【5-fa1f99】【6-3606a6】

 

---

 

## 3.1 Stdio MCP Server

 

A stdio MCP server runs locally as a subprocess.

 

Flow:

 

```text

Agent

  ↓

Local subprocess

  ↓

MCP server over stdin/stdout

```

 

Good for:

 

```text

Local scripts

Local tools

Local database utilities

Developer tools

Internal automation

```

 

Example SDK-style pattern:

 

```python

import asyncio

from google.antigravity import Agent, LocalAgentConfig, types

 

async def main():

    mcp_server = types.McpStdioServer(

        name="my_local_tools",

        command="python3",

        args=["./my_mcp_server.py"]

    )

 

    config = LocalAgentConfig(

        mcp_servers=[mcp_server]

    )

 

    async with Agent(config) as agent:

        response = await agent.chat(

            "Use the available MCP tools to help me."

        )

 

        print(await response.text())

 

if __name__ == "__main__":

    asyncio.run(main())

```

 

The official MCP example demonstrates connecting an agent to MCP servers through `LocalAgentConfig(mcp_servers=[...])`. 【5-fa1f99】

 

---

 

## 3.2 SSE MCP Server

 

SSE means:

 

```text

Server-Sent Events

```

 

Good for:

 

```text

Remote streaming MCP integrations

Server-pushed updates

Long-running external tools

```

 

---

 

## 3.3 Streamable HTTP MCP Server

 

Streamable HTTP MCP servers communicate over HTTP.

 

Good for:

 

```text

Remote MCP APIs

Cloud tools

External services

Managed integrations

Enterprise systems

```

 

---

 

## MCP Mental Model

 

```text

Custom Python Tool = Tool written inside your app

 

MCP Tool = Tool exposed by an external MCP server

```

 

Example:

 

```text

calculate_roi()              → custom Python tool

github_search_code()         → MCP tool

bigquery_run_query()         → MCP tool

linear_create_issue()        → MCP tool

notion_search_documents()    → MCP tool

```

 

---

 

# 4. How Policies, Hooks, and MCP Work Together

 

This is the production mental model:

 

```text

MCP gives external power

Policies control permission

Hooks observe, modify, and guard execution

```

 

Together:

 

```text

MCP     = What external tools can the agent use?

Policy  = Is the agent allowed to use this tool?

Hook    = What should happen before/after tool usage?

```

 

---

 

## Example Production Flow

 

```text

Agent connected to GitHub MCP

  ↓

User asks: "Create PR and push changes"

  ↓

Agent decides to call GitHub MCP tool

  ↓

Policy checks whether push is allowed

  ↓

Hook logs attempted GitHub action

  ↓

If risky, ask_user approval happens

  ↓

MCP tool executes

  ↓

Post hook logs result

  ↓

Agent returns final response

```

 

---

 

# 5. Quick Comparison

 

| Concept | Meaning | Main Use |

|---|---|---|

| Policy | Permission rules | Allow, deny, or ask before tool execution |

| Hook | Lifecycle interceptor | Logging, monitoring, safety, modification |

| MCP | External tool protocol | Connect APIs, databases, tools, and services |

 

---

 

# 6. Simple Memory Trick

 

```text

Policy = Can I do it?

Hook   = What happens before/after I do it?

MCP    = What external thing can I use?

```

 

---

 

# 7. Recommended Learning Order

 

## Step 1: Learn Policies

 

Goal:

 

```text

Make the agent safe.

```

 

Learn:

 

```text

deny("*")

allow("safe_tool")

ask_user("risky_tool")

```

 

Mini project:

 

```text

Policy-Protected Agent

```

 

Rules:

 

```text

Deny everything by default

Allow calculator tool

Allow read-only tools

Ask user before shell command

Deny delete/deploy operations

```

 

---

 

## Step 2: Learn Hooks

 

Goal:

 

```text

Observe and control agent lifecycle.

```

 

Learn:

 

```text

PreToolUse

PostToolUse

PreInvocation

PostInvocation

Stop

```

 

Mini project:

 

```text

Agent Audit Logger

```

 

It should log:

 

```text

User prompt

Tool name

Tool arguments

Tool result

Final response

Execution status

```

 

---

 

## Step 3: Learn MCP

 

Goal:

 

```text

Connect external tools.

```

 

Learn:

 

```text

McpStdioServer

McpSseServer

McpStreamableHttpServer

```

 

Mini project:

 

```text

Local Math MCP Agent

```

 

Agent should call MCP tools such as:

 

```text

add

multiply

calculate

```

 

The official MCP example uses a `pirate_math` MCP server and asks the agent to use a `pirate_multiply` tool. 【5-fa1f99】

 

---

 

# 8. Production Design Pattern

 

A production-grade agent should usually follow this pattern:

 

```text

1. Start deny-by-default

2. Allow only safe tools

3. Ask approval for risky tools

4. Deny dangerous tools

5. Log all tool calls using hooks

6. Sanitize sensitive data using hooks

7. Connect external systems through MCP

8. Audit all important actions

9. Validate tool inputs

10. Keep least-privilege access

```

 

---

 

# 9. Final Summary

 

Policies, hooks, and MCP are advanced Antigravity SDK concepts.

 

Their roles are:

 

```text

Policies → Safety and permissions

Hooks    → Lifecycle control and observability

MCP      → External integrations and live tools

```

 

Best short explanation:

 

```text

MCP gives your agent more power.

Policies prevent misuse of that power.

Hooks let you monitor, audit, and customize how that power is used.

```

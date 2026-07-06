# Antigravity SDK — Tools Notes

 

## 1. What Are Tools?

 

In Antigravity SDK, **tools are Python functions/callables that an agent can invoke to perform actions or fetch information**.

 

A normal LLM can only generate text.

 

But an agent with tools can:

 

- Read files

- Run commands

- Call APIs

- Query databases

- Calculate values

- Fetch business data

- Store temporary state

- Execute domain-specific logic

 

Simple formula:

 

```text

Agent = LLM + Tools + Memory + Planning + Execution Loop

```

 

---

 

## 2. Why Tools Are Important

 

Without tools:

 

```text

User Question

  ↓

LLM

  ↓

Text Answer

```

 

With tools:

 

```text

User Question

  ↓

Agent understands intent

  ↓

Agent selects a tool

  ↓

Tool executes

  ↓

Tool returns result

  ↓

Agent generates final answer

```

 

Example:

 

```text

User:

Calculate ROI if investment is 100000 and revenue is 150000.

 

Agent:

Uses calculate_roi() tool.

 

Tool:

Returns ROI is 50.00%.

 

Agent:

Explains result to user.

```

 

---

 

## 3. Tool Calling Flow

 

The tool calling lifecycle looks like this:

 

```text

User Query

  ↓

Agent checks available tools

  ↓

Agent decides which tool to use

  ↓

Agent passes arguments to tool

  ↓

Tool executes

  ↓

Tool returns output

  ↓

Agent uses output in final response

```

 

This is what makes an LLM an actual agent.

 

---

 

## 4. Basic Custom Tool

 

A custom tool is just a normal Python function.

 

```python

def get_current_temperature(location: str) -> str:

    """

    Gets the current temperature for a given location.

 

    Args:

        location: City name, e.g. "Mumbai", "Delhi", "Mountain View".

    """

 

    return f"The temperature in {location} is 30°C."

```

 

This tool can now be given to the agent.

 

---

 

## 5. Registering a Tool with Agent

 

To register a custom tool, pass it inside `LocalAgentConfig`.

 

```python

import asyncio

from google.antigravity import Agent, LocalAgentConfig

 

def get_current_temperature(location: str) -> str:

    """

    Gets the current temperature for a given location.

 

    Args:

        location: City name, e.g. "Mumbai", "Delhi", "Mountain View".

    """

 

    return f"The temperature in {location} is 30°C."

 

async def main():

    config = LocalAgentConfig(

        tools=[get_current_temperature]

    )

 

    async with Agent(config) as agent:

        response = await agent.chat(

            "What's the temperature in Mumbai?"

        )

 

        print(await response.text())

 

if __name__ == "__main__":

    asyncio.run(main())

```
<img width="1363" height="506" alt="image" src="https://github.com/user-attachments/assets/8b278630-3829-4d74-840e-d6b3896ad661" />

 

Important line:

 

```python

tools=[get_current_temperature]

```

 

This makes the function available to the agent.

 

---

 

## 6. How Agent Understands a Tool

 

The agent understands tools mainly through:

 

```text

1. Function name

2. Function arguments

3. Type hints

4. Docstring

5. User intent

```

 

Example:

 

```python

def calculate_roi(investment: float, revenue: float) -> str:

    """

    Calculates ROI percentage.

 

    Args:

        investment: Total investment amount.

        revenue: Total revenue generated.

    """

```

 

From this, the agent can understand:

 

```text

Tool purpose: Calculate ROI

Input 1: investment as float

Input 2: revenue as float

Output: string

```

 

---

 

## 7. Tool Design Rules

 

### Rule 1: Use Clear Function Names

 

Good:

 

```python

def get_customer_details(customer_id: str) -> str:

    ...

```

 

Bad:

 

```python

def do_task(x):

    ...

```

 

Clear names help the agent decide when to use the tool.

 

---

 

### Rule 2: Use Type Hints

 

Good:

 

```python

def calculate_discount(price: float, discount_percent: float) -> float:

    ...

```

 

Bad:

 

```python

def calculate_discount(price, discount):

    ...

```

 

Type hints help the runtime understand the expected input types.

 

---

 

### Rule 3: Write Good Docstrings

 

Good:

 

```python

def get_order_status(order_id: str) -> str:

    """

    Gets the current delivery status of an order.

 

    Args:

        order_id: Unique order ID, e.g. "ORD123".

    """

```

 

The docstring tells the agent:

 

```text

What the tool does

When to use the tool

What arguments mean

```

 

---

 

### Rule 4: One Tool Should Do One Thing

 

Good:

 

```python

def calculate_roi(investment: float, revenue: float) -> str:

    ...

```

 

Bad:

 

```python

def do_finance_sales_marketing_everything(input_data: str) -> str:

    ...

```

 

Focused tools are easier for the agent to use correctly.

 

---

 

## 8. Example: ROI Calculator Tool

 

```python

import asyncio

from google.antigravity import Agent, LocalAgentConfig

 

def calculate_roi(investment: float, revenue: float) -> str:

    """

    Calculates ROI percentage.

 

    Args:

        investment: Total investment amount.

        revenue: Total revenue generated.

    """

 

    roi = ((revenue - investment) / investment) * 100

 

    return f"ROI is {roi:.2f}%"

 

async def main():

    config = LocalAgentConfig(

        tools=[calculate_roi],

        system_instructions="You are a finance assistant. Use tools for ROI calculations."

    )

 

    async with Agent(config) as agent:

        response = await agent.chat(

            "Calculate ROI if investment is 100000 and revenue is 150000."

        )

 

        print(await response.text())

 

if __name__ == "__main__":

    asyncio.run(main())

```

 

Expected result:

 

```text

ROI is 50.00%

```

 

---

 

## 9. Example: Lead Scoring Tool

 

```python

import asyncio

from google.antigravity import Agent, LocalAgentConfig

 

def calculate_lead_score(

    company_size: int,

    budget_lakhs: float,

    urgency_level: str

) -> str:

    """

    Calculates lead score for a potential customer.

 

    Args:

        company_size: Number of employees in the company.

        budget_lakhs: Estimated budget in lakhs.

        urgency_level: One of "low", "medium", or "high".

    """

 

    score = 0

 

    if company_size > 500:

        score += 30

    elif company_size > 100:

        score += 20

    else:

        score += 10

 

    if budget_lakhs > 50:

        score += 40

    elif budget_lakhs > 10:

        score += 25

    else:

        score += 10

 

    if urgency_level.lower() == "high":

        score += 30

    elif urgency_level.lower() == "medium":

        score += 20

    else:

        score += 5

 

    if score >= 80:

        category = "Hot Lead"

    elif score >= 50:

        category = "Warm Lead"

    else:

        category = "Cold Lead"

 

    return f"Lead Score: {score}/100. Category: {category}"

 

async def main():

    config = LocalAgentConfig(

        tools=[calculate_lead_score],

        system_instructions="You are a sales assistant. Use tools when scoring leads."

    )

 

    async with Agent(config) as agent:

        response = await agent.chat(

            "Score this lead: company has 250 employees, budget is 20 lakhs, urgency is high."

        )

 

        print(await response.text())

 

if __name__ == "__main__":

    asyncio.run(main())

```

 

---

 

## 10. Async Tools

 

Use async tools when the tool performs I/O operations like:

 

- API calls

- Database queries

- Network requests

- File operations

- Long-running external calls

 

Example:

 

```python

import asyncio

from google.antigravity import Agent, LocalAgentConfig

 

async def fetch_customer_status(customer_id: str) -> str:

    """

    Fetches customer status from an external system.

 

    Args:

        customer_id: Unique customer ID.

    """

 

    await asyncio.sleep(1)

 

    return f"Customer {customer_id} is active and has premium support."

 

async def main():

    config = LocalAgentConfig(

        tools=[fetch_customer_status]

    )

 

    async with Agent(config) as agent:

        response = await agent.chat(

            "Check status of customer CUST-101"

        )

 

        print(await response.text())

 

if __name__ == "__main__":

    asyncio.run(main())

```

 

---

 

## 11. Stateful Tools with ToolContext

 

Sometimes a tool needs to remember information across multiple tool calls in the same conversation.

 

For that, Antigravity provides `ToolContext`.

 

Use cases:

 

```text

Expense tracking

Running totals

Pagination cursor

Temporary scratchpad

Conversation-level state

Intermediate results

```

 

---

 

## 12. ToolContext Example: Expense Tracker

 

```python

import asyncio

from google.antigravity import Agent, LocalAgentConfig, ToolContext

 

def record_expense(item: str, amount: float, ctx: ToolContext) -> str:

    """

    Records an expense and maintains total expense in the current conversation.

 

    Args:

        item: Name of the expense item.

        amount: Expense amount.

        ctx: Tool context injected by Antigravity.

    """

 

    expenses = ctx.get_state("expenses", [])

 

    expenses.append({

        "item": item,

        "amount": amount

    })

 

    ctx.set_state("expenses", expenses)

 

    total = sum(expense["amount"] for expense in expenses)

 

    return f"Recorded {item}: ₹{amount}. Current total: ₹{total}."

 

async def main():

    config = LocalAgentConfig(

        tools=[record_expense]

    )

 

    async with Agent(config) as agent:

        response1 = await agent.chat("Record chai expense of 20 rupees.")

        print(await response1.text())

 

        response2 = await agent.chat("Record lunch expense of 180 rupees.")

        print(await response2.text())

 

        response3 = await agent.chat("What is my total expense so far?")

        print(await response3.text())

 

if __name__ == "__main__":

    asyncio.run(main())

```

 

---

 

## 13. ToolContext Mental Model

 

```text

ToolContext = State object injected into tools

```

 

It allows tools to store and retrieve per-conversation state.

 

Basic methods:

 

```python

ctx.get_state("key", default_value)

ctx.set_state("key", value)

```

 

Example:

 

```python

expenses = ctx.get_state("expenses", [])

ctx.set_state("expenses", expenses)

```

 

---

 

## 14. Normal Tool vs Stateful Tool

 

### Normal Tool

 

```python

def calculate_roi(investment: float, revenue: float) -> str:

    ...

```

 

Good for:

 

```text

One-time calculation

API call

Fetching data

Simple transformation

```

 

---

 

### Stateful Tool

 

```python

def record_expense(item: str, amount: float, ctx: ToolContext) -> str:

    ...

```

 

Good for:

 

```text

Remembering tool-specific state

Tracking totals

Maintaining temporary conversation memory

```

 

---

 

## 15. Sync Tool vs Async Tool

 

### Sync Tool

 

Use when logic is simple and fast.

 

```python

def add_numbers(a: int, b: int) -> int:

    """

    Adds two numbers.

 

    Args:

        a: First number.

        b: Second number.

    """

 

    return a + b

```

 

### Async Tool

 

Use when tool waits for something.

 

```python

async def fetch_data(user_id: str) -> str:

    """

    Fetches data from an external system.

 

    Args:

        user_id: Unique user ID.

    """

 

    await asyncio.sleep(1)

 

    return "Data fetched"

```

 

---

 

## 16. Good Tool Return Values

 

Good return types:

 

```text

string

dict

list

number

simple structured output

```

 

Examples:

 

```python

return "ROI is 50.00%"

```

 

```python

return {

    "roi": 50.0,

    "status": "profitable"

}

```

 

Avoid returning very complex objects unless required.

 

---

 

## 17. Bad Tool Design Examples

 

### Bad Example 1: Vague Name

 

```python

def process(data):

    ...

```

 

Problem:

 

```text

Agent does not clearly know what this tool does.

```

 

Better:

 

```python

def calculate_customer_lifetime_value(customer_id: str) -> str:

    ...

```

 

---

 

### Bad Example 2: No Type Hints

 

```python

def calculate_roi(investment, revenue):

    ...

```

 

Better:

 

```python

def calculate_roi(investment: float, revenue: float) -> str:

    ...

```

 

---

 

### Bad Example 3: No Docstring

 

```python

def get_order_status(order_id: str) -> str:

    return "Delivered"

```

 

Better:

 

```python

def get_order_status(order_id: str) -> str:

    """

    Gets the current delivery status of an order.

 

    Args:

        order_id: Unique order ID.

    """

 

    return "Delivered"

```

 

---

 

### Bad Example 4: Too Many Responsibilities

 

```python

def sales_marketing_finance_hr_tool(input_text: str) -> str:

    ...

```

 

Better:

 

```python

def calculate_lead_score(...):

    ...

```

 

```python

def calculate_roi(...):

    ...

```

 

```python

def get_customer_details(...):

    ...

```

 

---

 

## 18. Tool Best Practices Checklist

 

```text

✅ One tool = one responsibility

✅ Use clear function names

✅ Use type hints

✅ Write descriptive docstrings

✅ Keep return values simple

✅ Prefer deterministic logic where possible

✅ Use async for API/database/network calls

✅ Use ToolContext when state is needed

✅ Validate important inputs

✅ Avoid vague tools like do_everything()

✅ Avoid hidden side effects unless required

✅ Keep dangerous tools protected with policies

```

 

---

 

## 19. Tool Safety

 

Some tools can be dangerous.

 

Examples:

 

```text

delete_file()

run_shell_command()

send_email()

transfer_money()

update_database()

deploy_to_production()

```

 

These tools should not run freely.

 

For dangerous tools, use:

 

```text

Policies

Human approval

Allow/deny rules

Input validation

Logging

```

 

Example policy idea:

 

```text

Safe read-only tools: allow automatically

Dangerous write/delete tools: ask user first

Highly risky tools: deny by default

```

 

---

 

## 20. Tool Categories

 

Common tool categories:

 

```text

Read-only tools

Write tools

Calculation tools

API tools

Database tools

Shell/terminal tools

File tools

Business logic tools

Stateful tools

```

 

---

 

### Read-only Tool

 

```python

def get_customer_details(customer_id: str) -> str:

    """

    Fetches customer details without modifying anything.

 

    Args:

        customer_id: Unique customer ID.

    """

 

    return "Customer details"

```

 

Low risk.

 

---

 

### Write Tool

 

```python

def update_customer_status(customer_id: str, status: str) -> str:

    """

    Updates customer status.

 

    Args:

        customer_id: Unique customer ID.

        status: New customer status.

    """

 

    return "Customer status updated"

```

 

Higher risk.

 

Should usually require approval or validation.

 

---

 

### Calculation Tool

 

```python

def calculate_discount(price: float, discount_percent: float) -> str:

    """

    Calculates discounted price.

 

    Args:

        price: Original price.

        discount_percent: Discount percentage.

    """

 

    final_price = price - (price * discount_percent / 100)

 

    return f"Final price is {final_price:.2f}"

```

 

Usually safe.

 

---

 

## 21. Mini Interview Answers

 

### Q1. What is a tool in Antigravity SDK?

 

A tool is a Python callable that the agent can invoke to perform an action or fetch information.

 

---

 

### Q2. How do you register a custom tool?

 

By passing the function inside `LocalAgentConfig`.

 

```python

config = LocalAgentConfig(

    tools=[my_tool]

)

```

 

---

 

### Q3. Why are docstrings important?

 

Docstrings help the agent understand:

 

```text

What the tool does

When to use it

What each argument means

```

 

---

 

### Q4. Why are type hints important?

 

Type hints help define the expected input and output structure of the tool.

 

---

 

### Q5. What is ToolContext?

 

`ToolContext` is an injected object that allows a tool to maintain state across multiple invocations in the same conversation.

 

---

 

### Q6. When should I use async tools?

 

Use async tools for:

 

```text

API calls

Database calls

Network calls

Long-running I/O operations

```

 

---

 

### Q7. What makes a good tool?

 

A good tool is:

 

```text

Clear

Focused

Typed

Documented

Predictable

Safe

```

 

---

 

## 22. Quick Template

 

Use this template for creating tools:

 

```python

import asyncio

from google.antigravity import Agent, LocalAgentConfig

 

def my_tool(input_value: str) -> str:

    """

    Describe exactly what this tool does.

 

    Args:

        input_value: Explain what this input means.

    """

 

    return f"Processed: {input_value}"

 

async def main():

    config = LocalAgentConfig(

        tools=[my_tool],

        system_instructions="You are a helpful assistant. Use tools when needed."

    )

 

    async with Agent(config) as agent:

        response = await agent.chat(

            "Use the tool with input hello."

        )

 

        print(await response.text())

 

if __name__ == "__main__":

    asyncio.run(main())

```

 

---

 

## 23. Final Summary

 

Tools are the most important part of an AI agent.

 

Without tools:

 

```text

LLM = Text generator

```

 

With tools:

 

```text

LLM + Tools = Action-taking agent

```

 

The basic tool pattern is:

 

```python

def my_tool(...):

    """

    Explain what the tool does.

    """

    return result

 

config = LocalAgentConfig(

    tools=[my_tool]

)

```

 

The quality of your agent depends heavily on the quality of your tools.

 

A strong agent needs:

 

```text

Good tools

Good names

Good docstrings

Good type hints

Good safety rules

Good state management

```

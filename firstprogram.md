```plaintext
import asyncio
from google.antigravity import Agent,LocalAgentConfig

async def main():
    config=LocalAgentConfig(api_key="AQ.Ab8RN6JRKXR33eZVaggZCQiUJzgLwJotxHn5hKHA"
                            )
    async with Agent(config) as agent:
        response=await agent.chat("Summerise every file of my directory")
        print(await response.text())

asyncio.run(main())
```
<img width="1349" height="685" alt="image" src="https://github.com/user-attachments/assets/ad43fab9-75b1-4566-b2e2-fb3b7f18a121" />

<img width="1365" height="756" alt="image" src="https://github.com/user-attachments/assets/3bf43171-dfb5-438c-b3dd-307a0f139911" />

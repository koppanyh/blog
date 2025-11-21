# LLM Multi-In Multi-Out: Implementing group conversation dynamics for LLMs

Saturday, November 8, 2025

---

How do you get an LLM, which produces one output for each input, to work in a group chat environment? If you're just sending each message to the LLM as it comes in, then you don't...

However, what if you somehow collect the messages that are coming in and then decide when to process them all in one go? This was exactly what I set out to explore in my [LLM Discord Bridge](https://github.com/koppanyh/LLMDiscordBridge) experimental chatbot program.

## The Algorithm

## The Prompt

https://github.com/koppanyh/LLMDiscordBridge/blob/main/system_prompt.txt

## Future Improvements

use the typing indicator to change the delay (wait longer during coalescing step if others are typing), send each coallesce update to the LLM to decide whether it's time to respond or wait for more messages.

<sub>Posted by [koppanyh](https://github.com/koppanyh) on 2025/11/08 at 3:17 PM PST</sub>

<sub>[Permalink](https://blog.kh-labs.org/2025/11/llm-multi-in-multi-out)</sub>

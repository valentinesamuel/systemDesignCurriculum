# TradeSpark — Company Introduction

TradeSpark is a Series B algorithmic trading firm running 40 proprietary trading strategies across 12 exchanges globally. They're not a bank. They're not a hedge fund. They're the infrastructure layer that makes microsecond-latency execution possible for quantitative trading strategies — the systems that react to market movements faster than a human can blink.

You join as a **Senior Infrastructure Engineer** twelve months after leaving NexaCare. CTO **Alex Hartmann**, former Goldman Sachs VP of Engineering, sent the recruiting message himself: "We need someone who can hold correctness and performance at the same time. Most engineers optimize for one and break the other."

The office is in a converted warehouse in Frankfurt, 400 meters from a major internet exchange point (IXP). The latency to that exchange matters. Every nanosecond counts at TradeSpark.

On your first day, Alex hands you a manila folder. Inside: a single-page runbook labeled "EMERGENCY RECOVERY PROCEDURE — DO NOT SHARE." The last line reads: *"If all else fails, call Marcus at the number below."* There's a handwritten phone number. You don't need to look it up — you already have it.

Alex: "Your predecessor wrote that runbook. He left to start his own fund. The number is for Marcus Webb — he did the original system design in 2021. He's a good friend of mine. And apparently he already knows you."

By day three, you've found Diego Reyes's TODO comment in the event sourcing code — different Diego, same problem. The runbook is three years outdated. And there's a production incident brewing in RocksDB.

Welcome to high-frequency trading infrastructure.

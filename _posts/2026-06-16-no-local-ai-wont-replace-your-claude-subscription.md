---
layout: post
title: "No, Local AI Won't Replace Your Claude Subscription (My Experience)"
description: My honest experience running local LLMs on a Strix Halo mini PC next to a paid Claude Max subscription, and why you can't just cancel the subscription and "go local".
date: 2026-06-16 18:00:00 +0300
categories: ai local-llm strix-halo claude qwen llama-cpp
---

## Why I'm writing this
Lately I got really into mini PCs for AI - the idea of running models locally, on my own hardware, in my own basement, with nothing leaving the box. And the more I read about it, the more I kept seeing the same loud take everywhere:

> _"Just cancel your $200/month Claude Max (20x), or the $100 one (5x), buy a mini PC, run a Qwen distilled from Opus, and save a ton of money - same thing, but free and private!"_

I want to be upfront: I did *not* go into this with rose-tinted glasses expecting local to replace anything. Honestly, I had no clear idea what to expect at all - the info online is all over the place, everyone has their own pretty benchmark, and it's genuinely hard to tell what's real and what's hype. I just found the whole thing interesting and wanted to see for myself.

So I bought the hardware, ran the models, used them for real work for a while - and the short version is: **the "go local and cancel your subscription" take does not work like that.** Not yet. I run *both* now - a paid Claude subscription **and** a local AI box - and I think that's the setup that actually makes sense. And to be clear: I don't regret the purchase one bit, and I'm still genuinely into it. Let me explain why, with numbers.

## My setup
Two machines:

1. A laptop.
2. A mini PC for AI - AMD Ryzen AI Max+ 395 (Strix Halo), 128 GB.

Both on WiFi. I started with **Fedora Server 43** on the mini PC, but that's just a console, and if you need a screen you have to drag a laptop/monitor to it. So I moved to **Fedora Workstation** (by now I'm already on **Fedora 44**) - now I can either work directly on the mini PC, or use mini PC + laptop together. The system itself eats around 2-3 GB of RAM (and remember, that's the same pool the model has to live in).

On the mini PC I have a search service in Docker (**[SearXNG](https://github.com/searxng/searxng) + an MCP server**, with Google, DuckDuckGo, etc. behind it) so the AI can search the web like Cursor does (and like Claude does too). Later I stacked more MCPs on top - **context7** for pulling library docs, plus **exa** and **tavily** for web search.

For the agent itself I tried a bunch - Continue, Cline, Claude Code CLI - and ended up on **[opencode](https://github.com/anomalyco/opencode)**. It's the most normal one for me. Funny side note: the opencode codebase itself looks kind of weirdly written, almost like it was left half-finished or quietly nerfed so the AI works a bit worse - probably just my paranoia, capitalism, who knows :D - but it's still the one I kept. So much so that I eventually forked it privately for myself (more on that below).

As the runtime under all of it: **[llama.cpp](https://github.com/ggml-org/llama.cpp)**, GGUF quants.

## The hardware - Strix Halo
The chip is AMD's **Ryzen AI Max+ 395**, their **Strix Halo** platform. The thing that matters is the **128 GB of unified memory**. That's the whole point: a 4090 has 24 GB and simply *cannot load* a 70-80B model, while this box holds it in RAM no problem. So when people say "Strix Halo beats a 4090" - for local LLMs that's true in the sense that it runs models a 4090 can't even open. On a tiny model the 4090 is of course faster.

A few things I learned the hard way:

- **The RAM is soldered. Forever.** You can't upgrade it. Pick your capacity at purchase and live with it.
- You can add a **second SSD** (second M.2), but NVMe prices right now are insane.
- People do wild things: hang a GPU off the **OCuLink** port instead of a second M.2, or chain a bunch of these mini PCs into a **cluster**. That's already "I'm training/serving huge models" territory.
- There's also Nvidia's **DGX Spark** (with its own Linux OS), but I think the future here is AMD - because the **NPU** (the dedicated neural accelerator baked into the chip) sips almost no power, and right now most inference still runs on the iGPU/CPU. Hybrid **NPU + iGPU** is being worked on too, and that's the part I'm most curious about.

> The machine itself isn't expensive for the experience you get - you land in a tiny percentage of people who ever tried any of this. And the price only keeps climbing: since I got mine it's already gone up by a few hundred bucks - this config sits around **$2,799** now.
{: .prompt-info }

About build quality - mine is a **[BosGame M5](https://www.bosgame.com/products/bosgame-m5-ai-mini-desktop-ryzen-ai-max-395-96gb-128gb-2tb)**. It works, but the build is meh: there was literally a **loose screw rattling around inside** the case. Some units have it, some don't - luck of the draw - and you almost have to wonder if it's a little warranty trap, since the only way to get it out is to crack the thing open, which is exactly how you might wave goodbye to your warranty. According to the [Strix Halo wiki](https://strixhalo.wiki/Hardware/PCs), **BosGame and Beelink are the most problematic** ones; if I were buying again I'd look harder at something like the **GMKtec EVO-X2** (or Framework Desktop, Minisforum MS-S1 Max, etc). The brand matters less than the chip inside - and since the Ryzen AI Max+ 395 isn't sold on its own, you just pick whichever prebuilt mini PC ships it with 128 GB.

## Lemonade - the AMD piece worth knowing
If you're on AMD hardware, look at **[Lemonade](https://lemonade-server.ai/)** ([GitHub](https://github.com/lemonade-sdk/lemonade)). Think of it as **LM Studio, but built for Strix Halo / Ryzen AI** - it's an open-source local AI server, community-built and **optimized by AMD engineers**, with an OpenAI-compatible API (so opencode and friends just point at it). It's multi-engine (llama.cpp on Vulkan/ROCm, FastFlowLM on the NPU, whisper.cpp, image gen...) and multi-modal, and it has proper **NPU hybrid** execution where the NPU does prompt processing and the iGPU generates tokens. It installs in basically one command and there's a Fedora build, which is exactly my setup. It's also what most of my models actually run on - so when I mention Lemonade later, that's what I mean.

## The models I run
The workhorse is **[Qwen3-Coder-Next](https://huggingface.co/Qwen/Qwen3-Coder-Next)** - an 80B MoE (mixture-of-experts) model that only fires up ~3B of those parameters per token, so you get big-model smarts at small-model speed. 256k native context (up to 1M with Yarn). I run it in GGUF, which lets you trade quality for memory by *quantizing* the weights - squeezing each number down to fewer bits (Q8 ≈ 8-bit, Q4 ≈ 4-bit), so the model eats less RAM at some cost to quality. Here's roughly what each quant costs in memory (just the model, at 256k context):

| Quant | Model size | Notes |
|-------|-----------|-------|
| F16   | won't load on 128 GB (OOM/crash) | and pointless anyway - Q8 is basically the same |
| Q8_0  | ~90 GB | add ~10 GB for a long chat + subagents and you're set; I never saw it go past +10 GB even over hours |
| Q6_K  | ~72 GB | fits in 80 GB, Q6 is totally fine |
| Q5_K_M | ~64 GB | |
| Q5_0  | ~62 GB | |
| Q4_K_M | ~56 GB | the realistic one if you're tight on memory |

Rule of thumb: **higher quant = better** (Q4 < Q6 < Q8). F16 only if you want the absolute ideal, which you usually don't.

Other models I actually use:

- **[Huihui-Qwen3-Coder-Next-abliterated](https://huggingface.co/mradermacher/Huihui-Qwen3-Coder-Next-abliterated-i1-GGUF)** - the "abliterated" (uncensored) variant. Sometimes the AI refuses to do something completely normal, and this one just does it. I run it at Q6 (the highest I bother with). At 256k context the model alone took **71 GB**.
- **[Qwen3.5-27B-Claude-4.6-Opus-Reasoning-Distilled-v2](https://huggingface.co/Jackrong/Qwen3.5-27B-Claude-4.6-Opus-Reasoning-Distilled-v2-GGUF)** - this is the one the "cancel your subscription" crowd loves. It's literally Qwen3.5-27B *distilled from Claude Opus 4.6* (that's what "Distilled" means - teach a small model using a big one). It's noticeably **slower** than the others, though - it's a dense 27B, so every parameter fires on every token, unlike the MoE ones that only light up ~3B at a time. 262k context, Q4_K_M ~16.5 GB.
- **[Qwen3.6-35B-A3B-Claude-4.7-Opus-Reasoning-Distilled-APEX-MTP](https://huggingface.co/mudler/Qwen3.6-35B-A3B-Claude-4.7-Opus-Reasoning-Distilled-APEX-MTP-GGUF)** - this is where it gets *usable*. The genuinely good local models right now are **MTP** ones (multi-token prediction). 35B-A3B MoE, distilled from Opus 4.7, and it hits **60+ tokens/s**. That's the speed that makes local feel real.

One thing people don't think about: to get close to the paid experience you also need **automatic chat-title generation** (those "what is this chat about" titles). That needs *another* model loaded = more memory. I use a small separate one - **[Huihui-Qwen3.5-9B-abliterated](https://huggingface.co/mradermacher/Huihui-Qwen3.5-9B-abliterated-GGUF)** - with a prompt from opencode, and it names the chats automatically. You can turn this off, it's just convenience. (At least the way I run it, it has no vision - can't drop a screenshot in like on the paid tools - which surprised me, since other Qwen models do have it.)

## Speed: TPS is not the whole story
This is the part the hype completely ignores.

The progress in llama.cpp is genuinely strong - one update **almost doubled** my throughput overnight: I was getting 20-25 tokens per second (TPS), then suddenly 40+. For comparison, Claude never shows you its own speed, so you have to look at third-party measurements - [Artificial Analysis](https://artificialanalysis.ai/providers/anthropic) benchmarks output speed across providers. As of mid-2026 they clock **Claude Opus 4.8 at ~64 t/s**, **Opus 4.7 ~54 t/s**, **Sonnet 4.6 / Opus 4.6 in the ~40-53 t/s** range, and little **Haiku 4.5 around ~100 t/s** (these numbers will have moved by the time you read this, but you get the shape of it). So my local 40+ TPS is already right in the same ballpark as the big Claude models, and the MTP ones above (**60+**) basically match them. On paper, raw TPS is almost a non-issue now. Great, ship it, cancel the sub, right?

No. Because there's a second number nobody talks about: **TTFT (time to first token)** - the prompt processing time. And **this is the painful part on local.** The more your context fills up, the longer the prompt takes to process before you see *anything*. On the paid services this is near-instant. Locally it's the single thing that annoys me most, and it's felt *way* more than the TPS difference.

> MTP is a great example of the trade-off. It nearly doubles your *generation* speed (self-speculative decoding, no separate draft model, enable with `--draft-mtp` in a recent llama.cpp), but it makes **prompt processing slower** and shrinks your max context. You're literally trading TTFT for TPS.
{: .prompt-warning }

When I gave **Qwen3-Coder-Next** and **Sonnet 4.6** (via Cursor) the exact same task, I got the same result from both. To be fair, that was *one* well-scoped task against *Sonnet* - not Opus, and not some long multi-step agentic job where the gap really opens up. On that kind of task, though, the only real differences were: the paid one burns through its limits fast while local doesn't, plus the speed/TTFT behavior.

## Where local actually shines
I don't want this to read like "local is bad", because it's really not - there's a whole pile of stuff where it just wins:

- **Privacy** - the big one. I'm confident nothing leaves the box (well... *almost* nothing - more on the opencode mess below). For some work that's the entire reason you'd even bother.
- It's perfect for the boring-but-sensitive stuff, like setting something up on a server over an **SSH MCP**, where you really want to *know* that your commands, passwords and code aren't quietly flying off to someone else's cloud.
- No quota, nothing ticking down, no "you've hit your limit for the day" - you can sit and grind on one thing for hours and never once think about it.
- The model selection is wild: distilled, abliterated, MTP, whatever weird thing you feel like trying. And you can fine-tune them too - I haven't properly sat down and done it yet, but stripping the refusals out of some model and throwing it up on Hugging Face is on my list (you can't train from scratch, mind you, only build on top of a base model).
- **Reversing / debugging.** I'll feed the local AI stuff like "there was a memory leak here, and some code is disabled so don't fixate on the wrong thing" while hunting a bug. The nastier low-level hooking I hand off to Claude/Sonnet and let the local model do the rest - so in practice I just split the work by how hard it is.
- **Security recon.** One weekend I pointed a local AI at a URL and at my router's IP and had it go hunting for holes and suggest fixes (neat little tool for exactly this: [METATRON](https://github.com/sooryathejas/METATRON), runs fully offline). It actually turned up a few small things on a couple of obscure sites - an open port, trivial parameter fuzzing, a header injection - and I reported them. Obviously: only ever on systems you own or have permission for.

## The privacy catch in opencode (and why I forked it)
Here's the irony: the whole appeal of local is "nothing leaves my machine" - but **opencode wasn't private by default.** There's a long, heated issue about exactly this - [#10416, "OpenCode is not private by default?"](https://github.com/anomalyco/opencode/issues/10416) (with follow-ups like [#15854](https://github.com/anomalyco/opencode/issues/15854)). The short version: even when you're running a fully local model, opencode would generate your **session titles** by sending the text off to its *own* cloud model (`gpt-5-nano` on the opencode provider), so your prompt quietly left your network. The guy who opened the issue only noticed because his firewall was blocking the outbound IPs. On top of that it phones home for other stuff (model lists, plugin downloads, the web UI) - all while the homepage said "privacy first". You can lock it down (`disabled_providers: ["opencode"]`, set your own `small_model`, `agent.title.disable: true`), and to their credit the maintainers have since dropped that `gpt-5-nano` title fallback - but proper "private/offline by default" still wasn't really there.

That bugged me enough that I forked opencode privately and called it **VibeCode** - basically a pile of quality-of-life tweaks for running local Qwen models on Lemonade / Strix Halo: local title generation, a real "private mode" that blocks all non-LLM outbound traffic, Qwen-friendly prompts, live prefill/TTFT in the UI, and a bunch more. But that's a whole story of its own - I'll write it up in a **second post**.

## Where it falls short (why you can't just cancel)
And here's the other half - why local is a *companion*, not a *replacement*, at least today:

- **Subagents are brutal.** Realistically you get 1, maybe 2, before the whole thing falls over - and that's on my box, with my models. The paid services casually run entire fleets of them.
- The capability gap is real. Local lags behind paid, and sure, you can claw a lot of it back with a really well-crafted prompt (sometimes you'll even get a *better* answer) - but it still doesn't quite get there.
- **TTFT**, like I keep banging on about. The fuller your context gets, the more it hurts, and it never really stops being annoying.
- The memory ceiling sneaks up on you. 128 GB feels enormous until the model's eating 90 GB (Q8), then context, then a title model, then a subagent... and F16 won't even load. It fills up fast, and there's no popping in another stick of RAM.
- It's basically a raw beta. Sometimes the AI just freezes, or goes completely braindead mid-task - I've reported a few bugs to [Lemonade Server](https://github.com/lemonade-sdk/lemonade) myself ([#1468](https://github.com/lemonade-sdk/lemonade/issues/1468), [#1398](https://github.com/lemonade-sdk/lemonade/issues/1398), [#1377](https://github.com/lemonade-sdk/lemonade/issues/1377)). It'll be powerful, but right now it's early.
- And the setup and tuning is the single most tedious part of all of it - also the most interesting, honestly, but it is very much *not* "buy box, save money, done".

## What's coming (and it's coming fast)
The pace here is the reason I'm long-term optimistic even while I tell you not to cancel your subscription:

- **NPU + iGPU hybrid** on Strix Halo, 100+ TPS expected. The NPU barely uses any power.
- **[TurboQuant](https://research.google/blog/turboquant-redefining-ai-efficiency-with-extreme-compression/)** (Google Research, ICLR 2026) - squeezes the KV cache down to ~3 bits with *claimed* near-zero quality loss and 6x+ memory savings (it's a fresh paper, not something I've measured myself). The community is already working on getting it into llama.cpp. If it lands, long context on local gets a lot cheaper.
- **MTP** landing in mainline llama.cpp (~2x generation speed, no extra model file).

The runtimes get better basically every week - half of this section will probably be outdated a few months from now, and that's the fun part.

## My recommendation
So here's the setup I actually recommend from experience, instead of the "ditch your subscription" hype:

> **Keep the Claude Max subscription (5x or 20x), AND get a local AI box (a BosGame M5, or any Strix Halo / Ryzen AI Max+ 395 with 128 GB).**
{: .prompt-tip }

Use Claude for the heavy, fast, multi-subagent, iterate-quickly work where capability and TTFT matter. Use the local box for privacy-sensitive tasks, unlimited grinding, weird/uncensored/fine-tuned models, experiments, and anything where the paid limits or "it left my machine" bother you.

They complement each other - they don't replace each other. The people yelling that you can drop $100-200/month and get the exact same thing locally are, simply, wrong today.

So why bother at all right now, if it won't replace the sub? Because local is maybe **1-2 years behind** something like Opus, and the honest reason to run it today isn't the money - it's getting your hands dirty *early*. By around **2030**, when local AI gets properly good, you don't want to be starting from zero - you'll already know the quants, the runtimes, the tuning, all the little gotchas, and you'll be fresh as a cucumber, ready for it. Call it an investment in future-you. That's the real story.

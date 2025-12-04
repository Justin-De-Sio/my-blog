---
title: "990prep.com - My First SaaS"
summary: "How I built a TOEIC prep SaaS in three months with Supabase, Svelte, and a ton of AI help."
categories: ["Post", "Blog"]
tags: ["post"]
date: 2025-12-03
draft: false
---

For the past three months, I've been building [990prep.com](https://990prep.com), a TOEIC preparation SaaS I wish I had when I was studying.

Along the way, I discovered Supabase, Svelte, and above all: a completely new way of coding with AI.

Here’s how it went.

## Learning the stack on the fly

I had never used Supabase before, but it was surprisingly easy to get started. I discovered row-level security, S3 storage, and the auth system, and implementing them was almost effortless.

For Svelte, I spent one morning following the official tutorial for about two hours. I was so excited to start that I didn't even finish the Svelte tutorial, and I never touched the SvelteKit tutorial before jumping into my project. I knew I could rely on Cursor to help me build the site.

## Coding 20× faster with AI

Very quickly, something hit me: it was about 20× faster to ask Cursor to do something than to do it myself. That blew my mind.

I only knew the basics of Svelte, but I rarely needed to write much code. Cursor's output was usually great. Server-side development, routing, forms, APIs - everything became faster, and I didn't need to constantly check the docs because it just worked. The frontend was also fine, especially when using component libraries like shadcn. Debugging was smoother, and AI helped a lot with translations into other languages.

The moment that truly impressed me was when I connected my Supabase project through the Supabase MCP. My productivity exploded. The LLM could explore my tables, data, functions, triggers, and even check auth rules, so it had a perfect understanding of the system. When I needed a complex SQL query, it was easily 50× faster than me at writing and analyzing it.

I still didn't let it design the initial schema, though. That part feels too important to hand over completely to an AI.

## The limits of AI help

Of course, it wasn’t perfect.

Sometimes I struggled with the LLM, but usually it was because I didn’t fully understand something myself. Integrating Stripe, for example, was hard. It was my first time using it, and the LLM struggled too because Stripe lives partly outside the codebase, so it doesn’t have full context.

I also tested LLMs with Docker, but I don’t really recommend it. They often lack the full picture, and Docker setups are rarely structured enough for them to reason about reliably. For packaging and deployment, I still need to use my own brain.

Those were the moments that reminded me: you still need understanding, intuition, and domain knowledge behind the prompts.

## The internal conflict

As I was building, I started asking myself: should I keep coding like this without fully understanding what the AI is cooking inside my repo?

Part of me felt I needed to stay in full control of the code. Another part wondered: why go deep if the AI can do it faster and sometimes better than me?

Whenever I got stuck, a little voice told me that the time I spent prompting might actually be better invested learning the theory and writing the code myself. I still don't have a perfect answer to that question. But I'm very aware that pure coding skills might matter less over time.

## From Cursor to Claude Code

At the beginning of October, I switched to Claude Code. Two reasons: I could use a lot more tokens than with Cursor, and it worked incredibly well.

With Sonnet 4.5, every piece of code it wrote felt smart. For the backend, I basically didn’t need to read documentation anymore. I just described what I wanted, and it figured it out.

Today, I started testing the new flagship model, Opus 4.5. It seems very good so far, but to be honest I don’t clearly see the difference from Sonnet yet.

## The nerdy part: tech stack

For my fellow nerds, here’s the stack I used:

- Supabase for database, auth, REST API, and S3 storage  
- SvelteKit as the web framework  
- Tailwind and shadcn/ui (I’m not a full-time frontend dev)  
- Docker for packaging  
- Dokploy for deployment  
- Stripe as the payment infrastructure  
- Mailtrap for transactional emails  
- PostHog for web analytics  
- Cloudflare as a reverse proxy  

If you’re preparing for the TOEIC, or just curious about what a mostly-AI-built SaaS looks like, check out the website: [990prep.com](https://990prep.com).


# Bazaar91 — E-commerce Site with an AI Shopping Assistant
<img width="1158" height="574" alt="Screenshot_1" src="https://github.com/user-attachments/assets/5703396d-6d43-41a4-a039-67b8f3d82ef2" />
<img width="1335" height="762" alt="Screenshot_2" src="https://github.com/user-attachments/assets/95c820d0-0730-43d4-a3dd-126f67182f05" />
<img width="1333" height="750" alt="Screenshot_3" src="https://github.com/user-attachments/assets/939c7ffa-eece-4521-9162-4c745bd54106" />

A demo e-commerce storefront (Bazaar91, Bangladesh) with a fully custom-themed,
AI-powered chatbot ("Bristi") embedded via n8n — built to showcase an
end-to-end AI automation workflow: front-end, agent orchestration, tool use,
and persistent memory, all wired together.

> **Note:** This is a demo/portfolio project. Products, prices, and business
> details are fictional and used only to demonstrate the system.

---

## What's in this repo

```
├── index.html                          # Full storefront (HTML/CSS/JS, single file)
├── n8n-workflow/
│   ├── bazaar91-chatbot-workflow.json  # Importable n8n workflow
│   └── system-prompt.md                # Standalone copy of the agent's system prompt
└── README.md
```

---

## Frontend

- **Stack:** Vanilla HTML / CSS / JavaScript — no build step, no framework, one file.
- **Features:** product catalogue with category filter + live search, cart drawer,
  quick-view modal, checkout flow with order confirmation, wishlist toggle,
  newsletter signup, fully responsive layout.
- **Design system:** custom color tokens and typography (Fraunces + Manrope +
  JetBrains Mono) built around a Bangladeshi visual identity (marigold,
  rickshaw-pink, indigo), rather than a generic template look.
- **Chat widget:** [`@n8n/chat`](https://www.npmjs.com/package/@n8n/chat),
  re-themed from scratch with CSS variables so it matches the rest of the site
  instead of looking like a bolted-on widget.

## AI Agent Workflow (n8n)

```
Chat Trigger  →  AI Agent ("Bristi")
                    ├── OpenRouter (DeepSeek) — LLM
                    ├── Postgres Chat Memory — per-session conversation memory
                    └── Supabase Tool — product & store-info lookup
```

- **LLM:** DeepSeek via OpenRouter.
- **Memory:** Postgres-backed chat memory (20-message context window) so the
  agent remembers what a customer already asked about in the same session
  (e.g. a product they were comparing, their delivery city) without asking
  them to repeat themselves.
- **Tool use / grounding:** a single Supabase table (`bazaar91_data`) holds
  both product records and store-info records, split by a `type` column
  (`product` vs `shop_info`). The agent is instructed to *always* query this
  tool before answering any factual question — prices, stock, specs, store
  hours, address — rather than relying on the model's own guesses. This is
  the main defense against price/stock hallucination in a retail chatbot.
- **Prompt engineering:** see [`n8n-workflow/system-prompt.md`](n8n-workflow/system-prompt.md)
  for the full system prompt and a short breakdown of the design choices in it
  (persona grounding, tool-first enforcement, bilingual matching, few-shot
  examples, data-leak guardrails).

## Setup / Import Instructions

1. **Frontend:** open `index.html` directly, or host it anywhere static
   (GitHub Pages, Netlify, etc.).
2. **Workflow:** import `n8n-workflow/bazaar91-chatbot-workflow.json` into
   your own n8n instance.
   - Add your own credentials for **OpenRouter**, **Supabase**, and
     **Postgres** (the JSON ships with placeholder credential references —
     no real keys are included).
   - Create a Supabase table named `bazaar91_data` with a `type` column
     (`product` / `shop_info`) plus whatever product/store fields you need
     (name, price_bdt, stock_quantity, rating, details, etc).
   - Activate the workflow and copy its **Chat Trigger webhook URL**.
3. **Connect them:** in `index.html`, find the `createChat({...})` call near
   the bottom of the file and replace:
   ```js
   webhookUrl: 'YOUR_N8N_WEBHOOK_URL_HERE',
   ```
   with your own webhook URL.

   **Don't commit a real, live webhook URL to a public repo.** Anyone
   reading the source could call your workflow directly and run up your
   LLM/API usage. Keep it as a placeholder here, and only wire in the real
   URL on your deployed/private copy — or add rate-limiting on the n8n side
   if you do expose a live demo.

## Possible next steps (noted here to show scalability awareness)

- The Supabase tool currently uses `getAll` with `returnAll: true` — fine for
  a small catalogue, but at scale this should move to filtered queries or a
  vector-search/embeddings setup so the agent isn't pulling the whole table
  on every turn.
- Add a lightweight admin view for order confirmations instead of relying on
  the agent to just relay order details to a human dispatcher.

## Tech Stack

`n8n` · `n8n AI Agent / LangChain nodes` · `OpenRouter (DeepSeek)` ·
`Supabase (Postgres)` · `Postgres Chat Memory` · `@n8n/chat` ·
HTML / CSS / JavaScript

---

Built by Abu Bakkar Siddik — n8n automation developer.

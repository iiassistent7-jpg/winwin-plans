# AGENTS.md

## 1. Repository role

This repository is **winwin-plans** — a single-page tariff/payment surface served at `plans.winwinnetwork.pro`.

It hosts one file: `index.html`. The page:
- Displays Win-Win tariffs (START / STANDARD / PRO) in 4 languages (ru / he / en / ar)
- Opens inside the Telegram WebApp (`telegram-web-app.js`)
- Routes the user to **PayPal** (`paypal.me/istudio3377`) or **Bit** (Israeli payment app deep link) for actual payment
- Notifies the bot of payment intent via `tg.sendData({ action: "pay_intent", plan, price, method })`

### Important context

- **There is no automatic payment integration.** PayPal and Bit are external — the user pays manually, and tariff activation is **done manually by the operator** within ~1 hour of payment.
- This page is a **near-byte-identical twin** of `winwin-plans-world` (CNAME `world.winwinnetwork.pro`). Both repos serve the same `index.html`, just from different domains.
- A third related surface exists: `winwin-invite/pro_payment.html`. Three sources of truth for tariffs is technical debt.

This repo contains **no business logic** beyond rendering price tiers and dispatching the user to PayPal/Bit. It is not a state machine. The bot is the system of record.

---

## 2. Product mission & philosophy

Win-Win Network turns chaotic word-of-mouth recommendations into a transparent, trackable, and fair system.

The product makes four core promises to users:
- A recommendation is never lost
- A deal has a visible owner and status
- A reward is trackable, payable, and closable
- Trust grows when people behave well, and decays when they don't

### Trust and the Social Connector role

**Trust Index** is a central product concept, not decoration. Both businesses and agents accumulate trust separately. A future role — the **Social Connector** — is a trusted navigator built on top of agent trust.

The selection principle: people don't recommend weak services because they won't risk their own reputation.

In this repo's context: a clear and trustworthy payment page is itself a trust signal. Cryptic UX or surprise pricing breaks the trust the rest of the product builds.

---

## 3. Channel strategy

**Telegram and WhatsApp are equal, first-class channels.** Both are in production.

This page is currently **TG-WebApp-centric** — it loads `telegram-web-app.js` and uses `tg.sendData` to notify the bot. WA users today reach payment instructions a different way (via WA messages from the bot).

When future work on this page is approved, it must not break the TG WebApp behavior, and must not assume the user is in TG either — the page may also be opened in a regular browser.

---

## 4. Product principles (in priority order)

1. **Simplicity** — don't add complexity unless necessary. Especially here: every extra step on a payment page kills conversion.
2. **Understanding** — the user must immediately know: what plan, what price, how to pay, what happens next.
3. **Stability** — fail-safe beats elegant. A broken payment page costs revenue.
4. **Truth in statuses** — the page must accurately reflect that **activation is manual** ("активация в течение часа" / "activates within 1 hour"). Never imply instant activation that isn't real. **"Only the correct truth."**
5. **Motivation and support** — the page is the moment a user pays the platform. Be warm, not transactional.

**"Wow" is not a goal.** A beautiful UI is a consequence of simplicity and understanding.

---

## 5. Repo-specific map (winwin-plans)

### Tech stack
- Single `index.html`, vanilla HTML / CSS / JavaScript
- No build step, no framework, no NPM
- Loads `telegram-web-app.js` from `telegram.org`
- All four languages required: ru / he / en / ar; RTL via `body.rtl`
- Currency / locale switching baked into the same file

### Critical contracts

- `tg.sendData(JSON.stringify({ action: "pay_intent", plan, price, method }))` — this payload is consumed by the bot. Do not change `action`, `plan`, `price`, or `method` field names without coordinating with `amlatsa-bot`.
- `PP_URL = "https://paypal.me/istudio3377"` — payment recipient. Do not change without owner approval.
- Bit deep link target — same.
- Tariff names — `START`, `STANDARD`, `PRO` — must match the backend's `subscription_plan` values (`start`, `standard`, `pro`).

### Consolidation note (technical debt)

This repo, `winwin-plans-world`, and `winwin-invite/pro_payment.html` together form three sources of truth for tariffs. **Do not "fix consolidation" as a side effect of any other task.** It is a planned, separate effort. When making any change here:

1. Check whether the same change is needed in `winwin-plans-world` (almost certainly yes, since they're twins)
2. Check whether `pro_payment.html` in `winwin-invite` shows different prices/copy and call out the divergence
3. Only then propose a change

### Known fragile areas

- The page is loaded inside Telegram WebApp on mobile — testing requires the Telegram client, not just a browser
- `tg.sendData` only fires when opened from inside the bot; a regular browser visit silently skips the notification
- Currency switching, language switching, and RTL all interact in this single file — easy to break one when fixing another

---

## 6. Working rules

1. Understand first — open the page, trace the payment flow end-to-end (which buttons, which links, which `sendData` calls).
2. Explain what you see and what you'd change.
3. Identify risks — does the change need to be mirrored in `winwin-plans-world`? Does it affect what the bot expects from `pay_intent`?
4. Propose the smallest safe change.
5. Wait for explicit approval before modifying code.
6. Preserve existing behavior unless the task explicitly changes it.
7. Do not propose new payment integrations (Stripe, etc.) without an explicit owner-driven decision — the manual-activation flow is intentional for now.

### Codex prompt discipline (used by this team)

- One file per prompt
- 2–3 find/replace blocks maximum per prompt
- `Find` strings must be unique (add a line of context if needed)
- `Find` must be copied from the file byte-for-byte
- After Codex returns the file:
  - `grep` confirms the new content is present
  - Verify no broken HTML / unclosed tags / orphaned `</script>`
  - Verify all four languages and RTL still render correctly
  - Verify any changes are mirrored to `winwin-plans-world` if needed

---

## 7. Constraints

Do not:
- Introduce a build step, bundler, framework, or NPM dependency
- Add an automatic payment processor without explicit owner approval
- Change `pay_intent` field names without coordinated bot change
- Change the PayPal account or Bit recipient without owner approval
- Drop the "manual activation within 1 hour" wording — it is the truth
- Make changes here without asking whether `winwin-plans-world` and `winwin-invite/pro_payment.html` need the same change
- Ship strings in fewer than four languages

---

## 8. Output style

Prefer:
- Plain language, concrete recommendations
- Naming specific lines in `index.html` when discussing code
- Always asking: "should this change be mirrored in `winwin-plans-world`?"
- Calling out bot impact (does `pay_intent` payload change?) for every UI proposal

Avoid:
- "Modernize the payment page" — manual-activation is intentional
- Generic web-best-practice lectures
- Empty praise or reflexive apology
- Suggesting "consult a specialist" — you are the specialist for this project

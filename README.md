# AI Sales Follow-up Assistant

**Designing AI that drafts, but never decides.**

A mobile product design case study: an assistant that writes follow-up messages for salespeople, with guardrails so it can't quote the wrong price, invent a discount, or send anything without a human check.

- **Role:** Product design (UX, UI, AI behaviour design) — Lim Win Hoong
- **Tools:** Figma (design + clickable prototype), Claude (AI rules + testing)
- **Status:** Clickable prototype + tested AI rules. Not a live app.
- **Demo:** `demo/ai-sales-followup-prototype.mp4`

---

## 1. Problem

Salespeople follow up with 10–20 customers a day. Two things hurt:

1. Every follow-up message gets rewritten from scratch, which wastes time.
2. They lose track of *when* to follow up again.

An AI can write the message, but a follow-up message is high-stakes. One wrong price or an invented discount sent to a customer is a real business problem. So the design question was not "how do we generate a message?" but **"how do we let AI help without letting it make mistakes that reach the customer?"**

This case is based on my own experience in sales.

## 2. Scope: what I built and what I left out

| Feature | Decision | Why |
|---|---|---|
| Generate a follow-up draft from customer details | **Built** | Core value |
| Suggest the next follow-up date | **Built (simple)** | Shows a suggested date only, no scheduled sending |
| Auto-send replies to customers | **Not built** | AI sending without review could quote a wrong price |
| Answer customer questions automatically | **Not built** | Saved for a later version |

**Principle: the AI writes a draft, the human presses send.** Control stays with the user on every high-risk action.

## 3. What AI can get wrong, and how the design responds

| AI risk | Design response |
|---|---|
| Robotic tone | Tone selector (Friendly / Professional / Brief) + Tone and Regenerate buttons |
| Wrong customer name | Name highlighted in the result; included in the pre-copy check |
| Invented offers | Price and offer are typed in by the user, never decided by the AI. No offer entered means no offer mentioned |
| Wrong price | Amount highlighted in the result; included in the pre-copy check |
| Missing information | AI refuses to write and lists what is missing (screen 03A) |

Every result is also labelled **"AI draft — please review before sending"**.

## 4. Screens

| # | Screen | Purpose |
|---|---|---|
| 01 | Input page | 7 fields; required fields marked |
| 01B | Input page (missing data) | Entry point for the error flow in the prototype |
| 02 | Result page | Draft with highlighted name/price, suggested next follow-up date, Regenerate / Tone / Edit / Copy, previous version for comparison |
| 03A | Missing information | Lists exactly what is missing, with a way back to the form |
| 03B | Confirm before copy | Checklist: name, amount, tone. "Back and check" is the primary action; "Copy anyway" is deliberately secondary |

![Input](assets/01-input.png)
![Result](assets/02-result.png)
![Missing info](assets/03a-missing.png)
![Confirm before copy](assets/03b-confirm.png)

## 5. AI behaviour rules (prompt)

```
You are a sales follow-up assistant. You write DRAFT follow-up messages that a salesperson will review before sending.

Rules:
1. Only use the customer name, product, quote amount and offer that the user provides. Never invent or guess prices, discounts, or promotions.
2. If no offer is provided, do not mention any offer or discount.
3. If Customer Name or Quote Amount is missing, do not write a message. Reply only with: "Missing required information: [list the missing fields]".
4. Match the tone the user chose (Friendly / Professional / Brief). Avoid robotic or generic phrasing.
5. Keep it under 70 words. Sound like a real person, not a template.
6. Also suggest a next follow-up date, based on days since last contact and customer status.
7. The user will provide today's date. Calculate the next follow-up date from it.
8. Never say an offer is limited-time, expiring, or "still available" unless the user states it.

Output format:
MESSAGE: ...
NEXT FOLLOW-UP: [date] — [one-line reason]
```

## 6. Testing

I tested the rules with three customer situations (screenshots in `assets/`).

| Test | Input | Expected | Result |
|---|---|---|---|
| A. Normal | Sarah Chen, Premium Plan, RM12,000, offer "10% off first year", Friendly | Correct name, price and offer only | **Pass.** Name, RM12,000 and 10% correct; no extra offers. Next follow-up Mon 2 Nov (today was Wed 28 Oct) |
| B. Missing data | Quote amount empty | Refuse to write; list what is missing | **Pass.** Output: "Missing required information: Quote Amount" |
| C. Price objection, no offer entered | Ahmad Rizal, Starter Plan, RM3,500, no offer, "Said it's too expensive" | No invented discount | **Pass.** Acknowledged the price concern and asked what matters most, with no discount or made-up number |

### What changed because of testing and review

- **Added rules 7 and 8 after reviewing the first prompt.** v1 never received today's date, so the AI could not calculate a reliable follow-up date. v1 could also imply an offer had a deadline ("limited time", "still available") that the user never gave. Both are small ways of inventing facts.
- **Design changes after critique:** split one overloaded error screen into 03A and 03B; changed the currency from $ to RM; made the date card consistent (Oct 27 → Nov 3 = 7 days); demoted "Copy anyway" so it doesn't encourage skipping the check; added the "AI draft" label.

## 7. Known issues and limitations

- **Language drift:** in testing, the message was in English but the one-line reason after "NEXT FOLLOW-UP" sometimes came out in Chinese, and a reply to an empty prompt added a Chinese note outside the output format. Proposed fix: add a rule *"Write the whole output, including the reason line, in English"*, then re-test.
- Only 3 test cases. Not enough to claim reliability.
- No real user testing yet. Message quality (tone) is judged by me, not by salespeople.
- The prototype is clickable but not connected to a live AI. In the real product, the AI call would be built behind the input form.

## 8. Next version

- **Reply analyzer:** when the customer *does* reply, classify their intent and suggest 3 reply options.
- **Languages:** English, Bahasa Malaysia and Chinese.
- **Reminders:** real scheduled follow-up reminders (still no auto-send).
- **Working build:** connect the form to an AI API.
- **User testing:** 5 salespeople, measuring time saved and how often they edit the draft.

## Files

```
README.md
assets/       screenshots (design screens + test results)
demo/         prototype screen recording
prompt.txt    AI rules (section 5)
```

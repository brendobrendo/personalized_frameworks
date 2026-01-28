## Suggestions for Developing the Skill of Understanding What People Want

1. **Practice Active Listening**: Focus on listening more than you speak during conversations.  
2. **Ask Open-Ended Questions**: Encourage others to share more about themselves by asking questions that can't be answered with a simple yes or no.  
3. **Empathize**: Put yourself in other people's shoes and consider their motivations and needs.  
4. **Reflect and Take Notes**: At the end of the day, jot down observations about what you learned from your interactions.

## List: Personal Development

- Task: Reflect back once per conversation today
  Notes: Summarize what the other person says in your own words before responding in at least 3 conversations today.
  Due: Today

- Task: Ask one open-ended question in each conversation
  Notes: Use “what,” “how,” or “tell me about…” to discover what people value or need in each interaction today.
  Due: Today

- Task: Write daily reflection log
  Notes: At the end of today, write down 3 things you learned about what people value or need from conversations, and how you discovered those insights.
  Due: Tonight

  ## Developing the “Understanding Personality” Pillar at Work

### 🪴 Mindset for Today
> **“Be water, not stone.”**  
Observe, adapt, and experiment with adjusting your approach to match your customers’ decision and communication styles while maintaining genuine connection.

---

## 🎯 Three Actionable Goals for Today

### 1️⃣ Classify Three Customers’ Personality Styles
For **three customer interactions today**, identify:
- **Task-Oriented** (facts, outcomes)
- **People-Oriented** (warmth, stories)
- **Process-Oriented** (details, assurances)
- **Vision-Oriented** (big-picture, “why”)

✅ **Action:** Write down your classification in your notes to build pattern recognition.

---

### 2️⃣ Match Pace and Energy
Consciously **adjust your speed, tone, and phrasing**:
- Faster + direct ➔ Driven, task-oriented
- Slower + warm ➔ Relational
- Calm, step-by-step ➔ Cautious
- Big-picture ➔ Vision-oriented

✅ **Action:** Practice this **three times today** and note what feels natural or difficult.

---

### 3️⃣ Validate by Linking to Their Words
When explaining or recommending, **link your statements to something they said**:
- “You mentioned you want to feel more secure with your savings…” (Cautious)
- “Since you’re looking to simplify your finances…” (Task/Outcome)
- “You shared wanting to focus on your family right now…” (Relational)

✅ **Action:** Do this **once per customer conversation** to build precision listening.

---

## 🛠️ Supporting Silent Questions
- Are they energized by details or big-picture outcomes?
- Are they seeking guidance or wanting control?
- Are they moving toward a goal or away from risk?

---

## 🪴 Reflection Prompt for After Work
> “What patterns did I notice in customer personalities today? What worked well when I adapted, and what felt awkward? How can I adjust tomorrow?”

---

```typescript
import { IBATransactionLegacy } from "../types/legacy/baTransaction";
import { ICheckingAccountTransactionDb } from "../types/db/checking";

export function dbToLegacyBATransaction(
  t: ICheckingAccountTransactionDb
): IBATransactionLegacy {
  return {
    _id: t.id,
    asterisk_field: t.posted_flag ?? "",
    blank_field: t.raw_reference ? Number(t.raw_reference) : 0, // see note below
    credit_card_balance: null,
    bank_account_balance: null,

    transaction_amount: t.amount,

    // if it’s a check, raw_reference may be a number; otherwise it might be junk
    check_number: t.raw_reference && /^\d+$/.test(t.raw_reference) ? Number(t.raw_reference) : 0,

    transaction_date: t.transaction_date,
    transaction_description: t.raw_description,

    type_of_transaction: t.type_of_transaction ?? "unknown",

    merchant: t.merchant_name ?? null,

    my_classification: null,
    my_description: null,
    other_notes: null,

    receipt: null,
  };
}
```

**Step 3: Change the API layer to return the new DB shape but keep the UI consuming legacy**
Type the API response as ICheckingAccountTransactionDb[]

Immediately map to legacy with the adapter

Return legacy to the rest of the app

```typescript
import { dbToLegacyBATransaction } from "../adapters/checkingTransactionAdapter";
import { ICheckingAccountTransactionDb } from "../types/db/checking";
import { IBATransactionLegacy } from "../types/legacy/baTransaction";

export async function fetchBATransactions(): Promise<IBATransactionLegacy[]> {
  const res = await fetch("/api/wf/checking/transactions");
  const data: ICheckingAccountTransactionDb[] = await res.json();
  return data.map(dbToLegacyBATransaction);
}
```






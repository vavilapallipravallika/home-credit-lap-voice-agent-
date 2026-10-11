# Home Credit LAP Voice Agent (Prompt Engineering Assignment)

An AI voice agent that calls existing Home Credit customers about a pre-approved **Loan Against Property (LAP)** offer of up to ₹75 lakh, runs a preliminary eligibility check, and hands qualified leads to a senior loan expert.

Built and tested on **Retell AI** using a single system prompt.

## Contents

| File | What it is |
|---|---|
| `LAP_voice_agent_system_prompt.pdf` | The full system prompt (readable version) |
| `LAP_general_prompt.txt` | The same prompt as plain text, ready to paste into a voice platform |
| `LAP_voice_agent_test_transcripts.pdf` | Results table and full transcripts of 13 test calls |
| `LAP_test_call_scripts.pdf` | The 20 test scenarios I planned, with expected behaviour |

**Call recordings (audio):** [PASTE GOOGLE DRIVE FOLDER LINK HERE]

## How the agent works

The prompt is written as a small state machine in natural language:

1. **Greeting and verification.** Confirms it is speaking to the right customer. If the customer is busy, it asks for a callback time and ends the call.
2. **Offer.** Presents the offer as a reward for customer loyalty, with the ₹75 lakh maximum.
3. **Transfer check.** If the customer has an existing loan on the property or wants to reduce their EMI, the agent tells them a loan-transfer specialist will contact them, and ends the call.
4. **Eligibility checklist (7 items).** Property type, ownership, original documents, loan amount, occupation and income mode, market value, tenure.
5. **Decision.** Any failed rule leads to a polite disqualification and the call ends. If all 7 items are answered and every rule passes, the agent hands off to a senior loan expert who will call back with the exact interest rates.

### Core logic: extract, validate, update, ask

After every customer reply the agent:

1. extracts every eligibility fact in the reply, even out of order
2. validates each fact against the rules
3. disqualifies or transfers immediately if needed
4. updates its internal state (latest clear answer wins)
5. asks only the earliest unanswered checklist item

This is how the agent remembers answers given early and never re-asks them.

### Eligibility rules

| Item | Eligible | Not eligible |
|---|---|---|
| Property type | Residential, Commercial, Industrial | Agricultural |
| Ownership | Sole or Joint | none |
| Original documents | Available for verification | Photocopies only |
| Loan amount | Up to ₹75,00,000 | Above ₹75 lakh: agent offers the maximum instead of rejecting |
| Occupation | Salaried or Self-employed | none |
| Income mode | Bank | Cash |
| Market value | Customer's estimate (no minimum) | none |
| Tenure | 3 to 15 years | Under 3 or over 15 years |

### Other behaviours covered by the prompt

- Natural speech: fillers, interruptions, corrections, several answers in one sentence
- Diversions: does not invent interest rates or product facts; defers to the senior loan expert and returns to the checklist
- Handoff gate: no handoff until every checklist item has been answered
- Prompt variables: `company_name`, `customer_name`, `agent_name`, `agent_gender`, `current_date`, `current_day`, `current_time`, `additional_context_from_rag`, `language_to_speak`, `conversation_history`, `customer_utterance`

## Test results

13 calls, all passing. Full transcripts are in `LAP_voice_agent_test_transcripts.pdf`.

| # | Scenario | Result |
|---|---|---|
| 1 | Eligible customer (happy path) | PASS |
| 2 | Agricultural property | PASS |
| 3 | Cash income | PASS |
| 4 | No original documents | PASS |
| 5 | Tenure too short (2 years) | PASS |
| 6 | Tenure too long (20 years) | PASS |
| 7 | Loan above ₹75 lakh | PASS |
| 8 | Existing loan (transfer) | PASS |
| 9 | EMI reduction (transfer) | PASS |
| 10 | Busy customer | PASS |
| 11 | Out-of-order answers | PASS |
| 12 | Joint ownership | PASS |
| 13 | Diversions and self-correction | PASS |

## Known observations and next improvements

- Enable the platform's **End Call** function so the call hangs up right after goodbye (the call stayed open in Test 2).
- Stop the agent from stating the allowed tenure range when asking the tenure question.
- Use fixed handoff wording that always mentions "exact interest rates".
- Speech recognition sometimes garbled answers ("fifty likes"); the agent usually asked a short clarifying question instead of guessing.

## Setup

1. Create an agent on Retell AI (single-prompt).
2. Paste the contents of `LAP_general_prompt.txt` into the prompt box.
3. Set the opening line to `Hello, may I speak with {{customer_name}}?`.
4. Provide values for the variables listed above and start a test call.

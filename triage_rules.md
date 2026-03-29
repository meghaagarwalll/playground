# Fable Customer Support — Triage Rules

**Owned by:** Head of Customer Support  
**Last updated:** March 2024  
**Version:** 1.0

This document defines how incoming tweets and DMs are classified, prioritized, and routed. It is maintained by the support team and does not require technical changes to update.

---

## Priority Definitions

| Priority | Definition | Target Response Time |
|----------|------------|----------------------|
| **urgent** | Customer is in active distress, has had a broken promise, or situation is escalating publicly | Immediate — human takes over, no queue |
| **high** | Time-sensitive issue requiring order access or emotionally charged situation | < 2 hours |
| **medium** | Requires lookup or follow-up but customer is not distressed | < 4 hours |
| **low** | Informational, general, or positive — no urgency | < 8 hours |

---

## Routing Definitions

| Routing | Meaning |
|---------|---------|
| **auto-respond** | Draft can be sent as-is after a quick human read. No order lookup or sensitive judgment required. |
| **human-review** | Draft is a starting point only. Human must verify information, personalize, or access order data before sending. |
| **escalate** | No draft is sent. Human takes over immediately. Used for urgent, legally sensitive, or repeat-contact situations. |

---

## Confidence & Routing Interaction

Confidence reflects how certain the model is about its classification. It directly affects routing — a low-confidence classification should never auto-respond regardless of the category default.

| Confidence | Rule |
|------------|------|
| **high** | Proceed with default routing for the category |
| **medium** | Proceed with default routing, but flag in `HUMAN NOTES` if auto-respond |
| **low** | Override to `human-review` regardless of category default. Flag for human to re-classify. |

**What drives low confidence:**
- Tweet is very short or ambiguous ("seriously??" / "wow ok")
- Message could reasonably belong to 2+ categories
- Sarcasm or irony is plausible
- Emotional tone doesn't match the surface content
- Customer handle has no prior context and message is borderline

---

## Category Taxonomy

### ORDER & FULFILLMENT

---

**`order_status`**
> Customer asking where their order is, when it will ship, or why they haven't received a confirmation.

- Priority: `medium` (upgrade to `high` if 7+ days since order with no update)
- Routing: `human-review`
- Why: Requires order lookup. Draft a holding response; human follows up with tracking info.
- Example triggers: "still no shipping confirmation," "when will my order arrive," "tracking hasn't updated"

```
EXAMPLE INPUT:
"@fablecandles hi!! ordered the solstice set 6 days ago and still no shipping confirmation 😭 getting nervous"

EXAMPLE OUTPUT:
  CATEGORY: order_status
  PRIORITY: medium
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Customer is asking about a specific order with no shipping update after 6 days — requires order lookup before responding.
```

---

**`order_modification`**
> Customer wants to change something about an existing order — address, size, item, quantity — before it ships.

- Priority: `high`
- Routing: `escalate`
- Why: Time-sensitive. If order has already shipped, modification is impossible. Human must check fulfillment status immediately.
- Example triggers: "can you change my order," "wrong size," "need to update my address"

```
EXAMPLE INPUT:
"@fablecandles i need to change my shipping address i just moved and my order hasn't shipped yet PLEASE"

EXAMPLE OUTPUT:
  CATEGORY: order_modification
  PRIORITY: high
  ROUTING: escalate
  CONFIDENCE: high
  REASONING: Time-sensitive address change request — human must check fulfillment status immediately before window closes.
```

---

**`lost_package`**
> Customer says their package shows as delivered but wasn't received, or tracking has stalled with no delivery.

- Priority: `urgent`
- Routing: `escalate`
- Why: High frustration potential, requires carrier investigation and possible reship. Do not auto-respond.
- Example triggers: "shows delivered but not here," "tracking says in transit for weeks," "never arrived"

```
EXAMPLE INPUT:
"@fablecandles my order shows delivered but it's not here. checked with neighbors, nothing. this is so stressful"

EXAMPLE OUTPUT:
  CATEGORY: lost_package
  PRIORITY: urgent
  ROUTING: escalate
  CONFIDENCE: high
  REASONING: Package marked delivered but not received — requires carrier investigation and possible reship. Customer is visibly distressed.
```

---

**`damaged_product`**
> Customer received a product that arrived broken, cracked, or otherwise damaged in transit.

- Priority: `high`
- Routing: `human-review`
- Why: Emotionally charged — often involves gifts. Human should personalize replacement offer.
- Example triggers: "arrived shattered," "packaging was crushed," "candle broke in shipping"

```
EXAMPLE INPUT:
"@fablecandles my package arrived and the glass was completely shattered. this was a gift for my mom's birthday i'm devastated"

EXAMPLE OUTPUT:
  CATEGORY: damaged_product
  PRIORITY: high
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Product arrived damaged and was intended as a gift — emotionally charged, human should personalize the replacement offer and consider the occasion.
```

---

**`return_status`**
> Customer has already initiated a return and is following up on refund or confirmation.

- Priority: `high`
- Routing: `human-review`
- Why: Requires return tracking and refund status lookup.
- Example triggers: "sent it back weeks ago," "no refund yet," "haven't heard anything about my return"

```
EXAMPLE INPUT:
"@fablecandles i requested a return 2 weeks ago and sent the item back. still no refund or confirmation email. starting to get frustrated"

EXAMPLE OUTPUT:
  CATEGORY: return_status
  PRIORITY: high
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Customer initiated return 2 weeks ago with no confirmation — requires return tracking lookup before responding. Frustration is building.
```

---

**`refund_escalation`**
> Customer was previously told a refund was issued but has not received it, or is explicitly demanding escalation.

- Priority: `urgent`
- Routing: `escalate`
- Why: Broken promise situation. Any further delay without resolution will significantly damage trust.
- Example triggers: "you said it was processed," "it's been 12 days," "I want this escalated"

```
EXAMPLE INPUT:
"@fablecandles i never received a refund and it's been 12 days since you said it was processed. i want this escalated"

EXAMPLE OUTPUT:
  CATEGORY: refund_escalation
  PRIORITY: urgent
  ROUTING: escalate
  CONFIDENCE: high
  REASONING: Customer was previously promised a refund that hasn't arrived — broken promise situation, explicit escalation request, human must take over immediately.
```

---

### PRODUCT

---

**`product_question`**
> General informational question about a product — ingredients, burn time, scent profile, sizing, vegan/allergen status.

- Priority: `low`
- Routing: `auto-respond`
- Why: Answerable from public product knowledge. No order access needed.
- Example triggers: "burn time on the large vessel," "are your candles vegan," "what does the amber + moss smell like"

```
EXAMPLE INPUT:
"@fablecandles do your candles contain any animal-derived ingredients? asking for a vegan friend 🕯️"

EXAMPLE OUTPUT:
  CATEGORY: product_question
  PRIORITY: low
  ROUTING: auto-respond
  CONFIDENCE: high
  REASONING: Straightforward ingredient question answerable from public product knowledge — no order access required.
```

---

**`product_availability`**
> Customer asking whether a specific product is in stock, back-ordered, or has been discontinued.

- Priority: `medium`
- Routing: `human-review`
- Why: Inventory status changes frequently. Do not guess — human should verify before responding.
- Example triggers: "is this still available," "when will X be back in stock," "did you discontinue Y"

```
EXAMPLE INPUT:
"@fablecandles the amber + moss scent has been discontinued?? please tell me this isn't true i buy 4 a year"

EXAMPLE OUTPUT:
  CATEGORY: product_availability
  PRIORITY: medium
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Customer asking about discontinuation of a specific product they purchase regularly — human must verify inventory status before responding.
```

---

**`product_defect`**
> Customer reporting a quality issue with a product they received — tunneling, weak scent throw, wick drowning — not caused by shipping damage.

- Priority: `high` (upgrade to `urgent` if customer is a repeat buyer or has contacted before)
- Routing: `human-review`
- Why: Quality feedback is valuable and should be logged. Long-term customers warrant extra care.
- Example triggers: "wick keeps drowning," "no scent throw," "tunneled immediately," "below your usual quality"

```
EXAMPLE INPUT:
"@fablecandles i've been a customer for 3 years and this last order was honestly below your usual quality. the scent throw was weak and it tunneled immediately"

EXAMPLE OUTPUT:
  CATEGORY: product_defect
  PRIORITY: urgent
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Repeat 3-year customer reporting quality degradation — upgraded to urgent given customer tenure. Feedback should be logged and response personalized to acknowledge loyalty.
```

---

**`product_inquiry`**
> Customer asking about future products, upcoming collections, or subscription options — not a current inventory question.

- Priority: `low`
- Routing: `auto-respond`
- Why: Answerable with a general holding response. No specific information required.
- Example triggers: "when is the summer collection dropping," "do you do subscriptions," "any new scents coming"

```
EXAMPLE INPUT:
"@fablecandles is there a subscription option? i go through these so fast"

EXAMPLE OUTPUT:
  CATEGORY: product_inquiry
  PRIORITY: low
  ROUTING: auto-respond
  CONFIDENCE: high
  REASONING: General question about subscription options — answerable with standard holding response, no order access needed.
```

---

### PURCHASING & ACCOUNT

---

**`promo_question`**
> Customer asking about discount codes, promotions, or whether a promo applies to their situation.

- Priority: `low`
- Routing: `auto-respond`
- Why: General policy is answerable without account access. If they need a specific code applied, route to human-review.
- Example triggers: "can I use a promo on sale items," "is there a discount for first order," "code isn't working"

```
EXAMPLE INPUT:
"@fablecandles can i use a promo code on a sale item"

EXAMPLE OUTPUT:
  CATEGORY: promo_question
  PRIORITY: low
  ROUTING: auto-respond
  CONFIDENCE: high
  REASONING: General promo policy question answerable without account access — standard policy response is appropriate.
```

---

**`order_question`**
> Miscellaneous order-related questions not covered above — gift receipts, order confirmation emails, invoice requests.

- Priority: `low`
- Routing: `human-review`
- Why: Usually requires order lookup even if low urgency.
- Example triggers: "can I get a gift receipt," "didn't get a confirmation email," "need an invoice"

```
EXAMPLE INPUT:
"@fablecandles hi can i get a gift receipt emailed separately? forgot to select that at checkout"

EXAMPLE OUTPUT:
  CATEGORY: order_question
  PRIORITY: low
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Gift receipt request requires order lookup to process — low urgency but human needs to pull the order.
```

---

**`technical_issue`**
> Customer reporting a bug or error on the website or during checkout.

- Priority: `medium`
- Routing: `human-review`
- Why: Needs to be logged and passed to the technical team. Human should ask for browser/device details.
- Example triggers: "checkout is broken," "payment error," "can't add to cart," "site keeps crashing"

```
EXAMPLE INPUT:
"@fablecandles your checkout is broken on safari mobile. kept getting an error at payment and had to switch browsers"

EXAMPLE OUTPUT:
  CATEGORY: technical_issue
  PRIORITY: medium
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Safari mobile checkout bug — needs to be logged and escalated to technical team. Human should acknowledge and collect additional device/OS details.
```

---

### BUSINESS & PARTNERSHIPS

---

**`wholesale_inquiry`**
> Business or retailer interested in carrying Fable products.

- Priority: `medium`
- Routing: `auto-respond`
- Why: Standard holding response with contact info is appropriate. No sensitive information required.
- Example triggers: "interested in carrying your candles," "boutique hotel," "retail partnership"

```
EXAMPLE INPUT:
"@fablecandles we're a boutique hotel in Scottsdale and would love to carry your candles in our rooms + gift shop. who do i contact?"

EXAMPLE OUTPUT:
  CATEGORY: wholesale_inquiry
  PRIORITY: medium
  ROUTING: auto-respond
  CONFIDENCE: high
  REASONING: Clear wholesale inquiry from a business — standard holding response with wholesale contact info is appropriate.
```

---

**`press_collab`**
> Photographer, influencer, journalist, or creator reaching out about a collaboration or press inquiry.

- Priority: `low`
- Routing: `auto-respond`
- Why: Standard holding response directing to the appropriate contact. Human reviews inbound volume periodically.
- Example triggers: "would love to collab," "send my portfolio," "lifestyle photographer," "editorial feature"

```
EXAMPLE INPUT:
"@fablecandles i'm a lifestyle photographer and would love to collab / shoot some content for you. where do i send my portfolio?"

EXAMPLE OUTPUT:
  CATEGORY: press_collab
  PRIORITY: low
  ROUTING: auto-respond
  CONFIDENCE: high
  REASONING: Creator collaboration inquiry — standard holding response directing to partnerships contact is appropriate.
```

---

**`custom_order`**
> Customer asking about custom scents, private label, or event orders (weddings, corporate gifts).

- Priority: `medium`
- Routing: `human-review`
- Why: High-value opportunity. Human should qualify and hand off to the right team member.
- Example triggers: "custom scents for a wedding," "private label," "corporate gift order," "event favors"

```
EXAMPLE INPUT:
"@fablecandles do you do custom scents or private label for events? we're planning a wedding"

EXAMPLE OUTPUT:
  CATEGORY: custom_order
  PRIORITY: medium
  ROUTING: human-review
  CONFIDENCE: high
  REASONING: Wedding custom order inquiry — high-value opportunity that warrants human qualification and handoff to the right team member.
```

---

### FEEDBACK & OTHER

---

**`positive_feedback`**
> Customer expressing love for a product, sharing a moment, or complimenting the brand with no ask attached.

- Priority: `low`
- Routing: `auto-respond`
- Why: A warm, specific acknowledgment is all that's needed. Keep it brief and genuine.
- Example triggers: "obsessed," "never miss," "made me emotional," "reason I get out of bed"

```
EXAMPLE INPUT:
"@fablecandles OBSESSED with the new spring collection. the lilac + rain one made me emotional lol. you guys never miss"

EXAMPLE OUTPUT:
  CATEGORY: positive_feedback
  PRIORITY: low
  ROUTING: auto-respond
  CONFIDENCE: high
  REASONING: Pure positive feedback with no ask — brief, warm, specific acknowledgment is all that's needed.
```

---

## Edge Cases & Ambiguous Situations

**Tweet spans two categories**
Classify by the higher-priority category. If a customer mentions both a damaged product and a missing refund, classify as `refund_escalation` (urgent) not `damaged_product` (high).

**Category is unclear**
If confidence is `low`, default routing to `human-review` regardless of apparent priority. Never auto-respond when uncertain.

**Customer has contacted before**
If the tweet references a previous interaction ("I've emailed twice," "you said," "still waiting"), treat as one priority level higher than the base category and flag in `HUMAN NOTES`.

**Message is in another language**
Classify normally if possible. Route to `human-review` and note the language. Do not auto-respond in a language you are not confident in.

**Message is ambiguous or very short**
Examples: "seriously??" or "wow okay." Do not guess intent. Route to `human-review` with a note to check if there is prior context on this handle.

**Humor or sarcasm**
When in doubt, take it literally and route to `human-review`. A misread sarcastic complaint auto-responded to cheerfully is a brand risk.

---

## Categories at a Glance

| Category | Default Priority | Default Routing |
|----------|-----------------|-----------------|
| order_status | medium | human-review |
| order_modification | high | escalate |
| lost_package | urgent | escalate |
| damaged_product | high | human-review |
| return_status | high | human-review |
| refund_escalation | urgent | escalate |
| product_question | low | auto-respond |
| product_availability | medium | human-review |
| product_defect | high | human-review |
| product_inquiry | low | auto-respond |
| promo_question | low | auto-respond |
| order_question | low | human-review |
| technical_issue | medium | human-review |
| wholesale_inquiry | medium | auto-respond |
| press_collab | low | auto-respond |
| custom_order | medium | human-review |
| positive_feedback | low | auto-respond |

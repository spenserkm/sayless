---
name: receipt-scanner
description: "Searches Gmail for purchase receipts and order confirmations, extracts purchase details (merchant, product, price, quantity, date, payment method, order number), and compiles them into a downloadable CSV file. Use this skill whenever the user asks to find their purchases, track spending, list receipts, export purchase history, review what they bought, audit spending from email, or anything related to extracting purchase/order data from Gmail. Also trigger when the user mentions 'receipts', 'order confirmations', 'purchase history', 'spending tracker', or 'what did I buy'. Works with the Gmail connector to scan emails and extract structured purchase data. Even if the user just casually says something like 'how much have I spent' or 'show me my orders', use this skill."
---

# Receipt Scanner Skill

Searches Gmail for purchase receipts and order confirmations, extracts structured purchase data, and exports it as a downloadable CSV.

## Prerequisites

- **Gmail connector** must be enabled (uses `Gmail:gmail_search_messages` and `Gmail:gmail_read_message`)

## Workflow

### Step 1: Determine Search Scope

Ask the user if not already specified:
- **Time range**: e.g., "last 30 days", "2024", "Jan–Mar 2025" (default: last 90 days)
- **Specific merchants or categories**: or "all" (default: all)
- **Minimum amount filter**: optional

### Step 2: Search Gmail for Receipts

Run **multiple search queries** to maximize coverage. Use `maxResults: 100` per query and paginate with `pageToken` if needed.

**Primary queries (always run all of these):**
```
Query 1: "subject:(receipt OR order confirmation OR purchase confirmation) after:YYYY/MM/DD before:YYYY/MM/DD"
Query 2: "subject:(your order OR order shipped OR payment received OR payment confirmation) after:YYYY/MM/DD before:YYYY/MM/DD"
Query 3: "subject:(invoice OR transaction OR payment receipt) after:YYYY/MM/DD before:YYYY/MM/DD"
```

**Deduplicate** results by message ID before reading full messages.

**Filter out non-purchase emails by subject/sender patterns:**
- Skip: "payment failing", "payment declined", "payment is due", "payment is scheduled", "payment method updated", "upcoming payment", "scheduled payment reminder"
- Skip: marketing emails, promotional offers, password resets
- Keep: actual receipts, order confirmations, payment confirmations with amounts

### Step 3: Read and Parse Each Email

Use `Gmail:gmail_read_message` for each unique message. Extract these fields:

| Field | Description | Extraction Strategy |
|---|---|---|
| `date` | Purchase date (YYYY-MM-DD) | From email `Date` header or body content |
| `merchant` | Store/company name | From subject line (e.g., "from McDonald's"), `From` header, or body |
| `product_description` | Item(s) purchased | From body text; for food delivery = restaurant name + "food delivery"; for subscriptions = plan name |
| `quantity` | Number of items (default: 1) | From body if available |
| `unit_price` | Price per item | From body if itemized |
| `total_price` | Total charged | Look for patterns: "Total: $XX.XX", "Estimated Total $XX.XX", "Amount paid $XX.XX", "$XX.XX Paid", or standalone dollar amounts |
| `currency` | Currency code (default: USD) | From body text |
| `payment_method` | Card/payment info | Look for: "Visa Ending in XXXX", "Visa *XXXX", "Mastercard ending XXXX", "Payment method: Link" etc. |
| `order_number` | Confirmation/order # | Look for: order IDs in subjects, "Order number:", "Receipt number:", "Transaction ID:", "Confirmation code:" |
| `category` | Auto-categorize | See category rules below |

### Merchant-Specific Parsing Patterns

Based on common email formats:

**DoorDash** (`no-reply@doordash.com`):
- Merchant name: Extract restaurant from subject "Order Confirmation for [Name] from [Restaurant]"
- Total: Look for "Total: $XX.XX" or "Estimated Total $XX.XX"
- Payment: "Paid with [Card] Ending in XXXX"
- Category: Food Delivery
- Product description: "[Restaurant name] - DoorDash delivery"

**Lyft** (`no-reply@lyftmail.com`):
- Multiple rides may be in a single receipt email
- Each ride has its own fare: "$XX.XX" next to "Ride fare"
- Total at bottom: "$XX.XX" as the daily total
- Payment: "Visa *XXXX"
- Category: Transportation
- Create ONE row per Lyft receipt email using the daily total (not individual rides)

**Anthropic/Stripe receipts** (`@mail.anthropic.com`, Stripe-powered):
- Body contains: "Receipt #XXXX-XXXX-XXXX", "Total $XX.XX", "Amount paid $XX.XX"
- Product: "Claude Pro Qty 1"
- Payment method: Often "Link" or card info
- Category: Subscription

**Google Play** (`googleplay-noreply@google.com`):
- Subject: "Your Google Play Order Receipt from [Date]"
- Body: Subscription name, "Order number: GPA.XXXX-XXXX-XXXX-XXXXX"
- Category: Subscription

**PayPal** (`service@paypal.com`):
- Subject: "Receipt for your payment to [Merchant]"
- Body: "You paid $XX.XX USD to [Merchant]", "Transaction ID XXXXXXXXXXX"
- Category: varies by merchant

**NordVPN / Subscriptions** (various):
- Body: "Item Price [Plan]: [amount] USD"
- Category: Subscription

**Utility Payments** (various bill pay senders):
- Subject: "Payment Confirmation - [Utility]"
- Category: Utilities

**Generic pattern** (fallback):
- Look for dollar amounts: `$[0-9]+\.[0-9]{2}`
- Look for "Total", "Amount", "Charged", "Paid" near dollar amounts
- Merchant from From header or subject

### Category Rules

Auto-assign based on merchant/sender:
- **Food Delivery**: DoorDash, Uber Eats, Grubhub, Instacart
- **Transportation**: Lyft, Uber (rides), transit
- **Subscription**: Anthropic, Netflix, Spotify, NordVPN, Google Play subscriptions, OpenAI
- **Electronics/Shopping**: Amazon, MPB, Best Buy, etc.
- **Utilities**: NW Natural, Waste Management, City of Hillsboro, water, electric, gas
- **Healthcare**: AFC Urgent Care, medical/dental/pharmacy
- **Housing**: HOA, rent payments (Western Alliance Bank ePay)
- **Insurance**: Progressive, State Farm, etc.
- **Dining**: Restaurant payments (Zelle for dinner, etc.)
- **Membership**: D-BAT, gym, club memberships
- **Other**: Anything that doesn't fit above

### Step 4: Generate the CSV

Create the CSV with Python:

```python
import csv

headers = ["Date", "Merchant", "Product Description", "Quantity", "Unit Price", "Total Price", "Currency", "Payment Method", "Order Number", "Category"]

# Sort by date descending
purchases.sort(key=lambda x: x['date'], reverse=True)

with open('/mnt/user-data/outputs/purchase_history.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(headers)
    for p in purchases:
        writer.writerow([
            p['date'], p['merchant'], p['product_description'],
            p['quantity'], f"{p['unit_price']:.2f}", f"{p['total_price']:.2f}",
            p['currency'], p['payment_method'], p['order_number'], p['category']
        ])
    # Summary row
    total_spend = sum(p['total_price'] for p in purchases)
    writer.writerow([])
    writer.writerow(["", "", "TOTAL SPEND", "", "", f"{total_spend:.2f}", "", "", "", ""])
```

### Step 5: Present Results

1. **Present the CSV file** using `present_files` tool
2. **Show summary** in chat:
   - Total purchases found
   - Total amount spent
   - Top 5 merchants by total spend
   - Breakdown by category
   - Date range covered
   - Any emails that couldn't be parsed (list them)

## Edge Cases

- **Multi-item orders**: If itemized, create one row per item. If not, one row with "Multiple items" description and total.
- **Refunds**: Include with negative total_price and "REFUND" prefix in product description
- **Duplicate emails** (order confirm + shipping for same order): Deduplicate by order number
- **No amount found**: Skip but list in summary as "could not parse"
- **Credit card payments** (Chase, BofA statements): These are bill payments, NOT purchases. Skip them unless the user specifically asks.
- **Zelle payments**: Include if they appear to be purchases; note Zelle in payment method
- **Failed/declined payments**: Always skip these

## Rate Limiting

- Process emails in batches of 10–20
- If >200 receipts found, warn user and ask whether to proceed or narrow the range

## Example Output CSV

```csv
Date,Merchant,Product Description,Quantity,Unit Price,Total Price,Currency,Payment Method,Order Number,Category
2025-02-28,Dave's Hot Chicken,Dave's Hot Chicken - DoorDash delivery,1,20.53,20.53,USD,Visa ending 8453,,Food Delivery
2025-02-13,Anthropic,Claude Pro - Monthly,1,20.00,20.00,USD,Link,2623-5103-4100,Subscription
2025-02-08,Lyft,Lyft rides - February 7,3,0.00,69.50,USD,Visa *8453,,Transportation
```

# Race Condition

**Tags:** #WebSecurity #RaceCondition #OWASP #BugBounty

---

# What is a Race Condition?

A **Race Condition** is a vulnerability that occurs when **multiple requests execute simultaneously**, allowing attackers to exploit the small time gap between **checking** and **updating** data.

> [!NOTE]
> A race condition is a **timing vulnerability** caused by concurrent execution of requests.

---

# Simple Example

Imagine there is **only one movie ticket** left.

Two users click **Buy** at the same time.

```
Check Ticket Available
        │
       Yes
        │
Process Payment
        │
Mark Ticket Sold
```

Both users may successfully purchase the ticket because the server checks availability before updating the database.

---

# Race Window

The small gap between:

```
Check
   │
Validate
   │
Update
```

is called the **Race Window**.

Attackers try to send multiple requests during this window.

---

# Critical Section

A **Critical Section** is code that should never execute simultaneously for the same resource.

Example:

```python
if balance >= amount:
    balance -= amount
```

If two requests read the same balance before updating it, both may succeed.

---

# Common Types

- Limit Overrun
- Wallet / Balance Race
- Inventory Race
- Coupon Race
- Gift Card Race
- Password Reset Race
- OTP / MFA Race
- File Upload Race
- Account Registration Race

---

# Common Targets

- Payments
- Wallets
- Coupons
- Gift Cards
- Inventory
- Booking Systems
- Password Reset
- OTP Verification
- Reward Points
- Rate Limits

---

# Example

```
POST /buy

Check Stock
      │
 Stock = 1
      │
Charge Card
      │
Reduce Stock
```

If two requests arrive together:

```
Request A  ──► Stock = 1

Request B  ──► Stock = 1
```

Both purchases may succeed.

---

# Impact

- Financial Loss
- Double Spending
- Coupon Abuse
- Inventory Overselling
- Duplicate Rewards
- Account Takeover (some cases)
- Data Corruption

---

# Prevention

✅ Atomic Operations

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1
AND balance >= 100;
```

✅ Database Transactions

- BEGIN
- COMMIT
- ROLLBACK

✅ Row Locking

```sql
SELECT *
FROM products
FOR UPDATE;
```

✅ Optimistic Locking

Use version numbers or timestamps.

✅ Unique Constraints

```sql
UNIQUE(username)
```

✅ Idempotency Keys

Prevent duplicate payment or order processing.

✅ Queue-Based Processing

Process requests one at a time.

---

# Detection Tips

Look for operations involving:

- Payments
- Wallets
- Coupons
- Inventory
- Password Reset
- OTP
- File Upload
- User Registration

Testing Steps:

1. Identify a state-changing request.
2. Send multiple concurrent requests.
3. Compare responses.
4. Verify backend state.
5. Repeat to confirm.

---

# Key Terms

| Term | Meaning |
|------|---------|
| Race Condition | Multiple requests compete for the same resource |
| Race Window | Time gap between check and update |
| Critical Section | Code that must not run concurrently |
| Atomic Operation | Executes as one indivisible action |
| Lock | Prevents simultaneous access |
| Transaction | Group of database operations |
| Idempotency | Repeating a request has the same effect |
| Optimistic Locking | Uses version checking to detect conflicts |

---

# Summary

A **Race Condition** is a **timing-based vulnerability** where multiple requests exploit the gap between **checking** and **updating** shared data. The best defenses include **atomic operations**, **transactions**, **locking**, **idempotency**, and **database constraints** to ensure only one request succeeds when appropriate.

> [!TIP]
> If an action should happen **only once** (payment, coupon, OTP, booking, or inventory purchase), always consider testing for race conditions.
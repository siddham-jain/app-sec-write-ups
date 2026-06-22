# How I Turned Negative Quantities into Free Store Credit

## What I Was Trying to Do

I moved on to the next business logic lab, and this one had a fun twist. The goal was to buy the **Lightweight l33t leather jacket** even though I didn't have enough store credit to afford it. The application let users modify product quantities in their cart, and I had a hunch that the validation around those quantities might be weak.

---

## How I Figured It Out

As I started exploring, I realized the application trusted the client-supplied `quantity` parameter and didn't properly validate negative values. That got me thinking: what if I could add a cheap item, make its quantity negative to create a negative cart total, and then use that "credit" to offset the cost of the expensive jacket?

Here is the attack chain I mapped out in my head:

1. Add a cheap item to the cart.
2. Modify the quantity to a negative value.
3. Force the cart total into a negative amount.
4. Add an expensive item.
5. Offset its cost using the negative balance.
6. Complete the purchase using available credit.

It sounded crazy, but I decided to give it a shot.

---

## What I Tried

### Logging In

I logged in with the provided credentials:

```text
Username: wiener
Password: peter
```

**Screenshot:** `images/login_success.png`

---

### Adding a Cheap Item

I looked for the cheapest product in the store and added it to my cart.

**Screenshot:** `images/cheap_item_added.png`

---

### Catching the Quantity Request

I enabled Burp Suite interception and added another unit of the cheap item. I wanted to see how the quantity was being sent to the server.

The request looked like this:

```http
POST /cart
```

With parameters:

```http
productId=2&quantity=1
```

**Screenshot:** `images/original_quantity_request.png`

---

### Going Negative

I sent the request to Repeater and changed the quantity parameter:

```http
quantity=-10
```

I picked a negative value and fired it off. The application accepted it without any complaints and updated my cart accordingly.

---

### Building Up Store Credit

I repeated the process, pushing the quantity further into negative territory until my cart had a negative total. It looked something like this:

```text
Quantity: -99
Total: -$X.XX
```

That negative balance effectively became store credit. I couldn't believe it actually worked.

**Screenshot:** `images/negative_cart_total.png`

---

### Adding the Leather Jacket

Now for the fun part. I added the target product:

```text
Lightweight l33t leather jacket
```

The negative cart balance offset the jacket price, bringing the total down to something I could actually afford.

**Screenshot:** `images/jacket_added.png`

---

### Checking Out

I proceeded to checkout and placed the order. The manipulated cart total allowed the purchase to go through successfully.

**Screenshot:** `images/lab_solved.png`

---

## Why This Is a Big Deal

If this happened in a real-world app, someone could:

- Purchase products below intended prices
- Generate unauthorized credit
- Bypass business rules
- Cause financial loss to the organization
- Manipulate transactional workflows

---

## What Was Broken

The application failed to validate basic business constraints on user-supplied quantities.

Specifically:

- Negative quantities were accepted.
- Cart totals were calculated directly from manipulated values.
- No server-side enforcement ensured quantities remained positive.

---

## How to Prevent It

Here is what should be in place to stop this:

1. Validate quantities on the server side.
2. Reject negative values.
3. Enforce minimum quantity constraints.
4. Recalculate pricing independently on the server.
5. Apply business-rule validation before checkout.
6. Perform integrity checks on cart contents.

---

## What I Took Away

Business logic vulnerabilities often show up when applications trust user-controlled data and fail to enforce expected business rules. In this lab, accepting negative quantities allowed manipulation of pricing calculations and enabled unauthorized purchases. It was a great reminder that input validation isn't just about SQL injection and XSS; sometimes the business logic itself is the weakest link.

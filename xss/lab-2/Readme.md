# Stored XSS: The Comment That Would Not Go Away

## How I Found the Vulnerability

I moved on to the stored XSS lab next. This one was different from reflected XSS because the payload would stick around. Instead of a search box, the target was a blog comment section. I figured if I could get my script into a comment and the application saved it without any filtering, every person who loaded that page would execute my code.

I navigated to one of the blog posts and scrolled down to the comment section. It was a standard form with fields for name, email, website, and the comment itself. I started wondering if the comment field would let me pass raw HTML through. There was only one way to find out.

I entered my payload into the comment body and filled in the other fields with some dummy data. I submitted the comment, and the application accepted it without any complaints. That was my first sign that something was off. Then I reloaded the blog page, and there it was. My script had been stored by the server and was now being served to anyone who visited the page.

Because malicious JavaScript was permanently stored by the application, every user who visited the affected page would execute my payload in their browser.

Unlike Reflected XSS, Stored XSS does not require victims to click a specially crafted link. The payload executes automatically whenever the vulnerable page is viewed.

---

## What I Did

Here is exactly how I exploited it:

1. I navigated to any blog post.
2. I scrolled to the comment section.
3. I entered this payload into the comment field:

```html
<script>alert(1)</script>
```

4. I provided valid values for the remaining fields:

```text
Name: Bhavya
Email: test@test.com
Website: https://test.com
```

5. I submitted the comment.
6. The application stored the comment in its backend database.
7. I reloaded the blog page.
8. The stored comment was rendered without output encoding.
9. The browser executed my injected JavaScript.
10. The lab was successfully solved.

---

## Proof of Concept

### Payload I Used

```html
<script>alert(1)</script>
```

### The Vulnerable Workflow

```text
User Input
    ↓
Stored in Database
    ↓
Displayed in Blog Comment
    ↓
JavaScript Executed
```

### What the Stored Response Looked Like

```html
<div class="comment">
<script>alert(1)</script>
</div>
```

### Execution Result

```javascript
alert(1)
```

The browser interpreted the stored payload as executable JavaScript because the application does not perform output encoding before rendering user-supplied content.

---

## Screenshots

### Screenshot 1 – Malicious Payload Submission

**What I saw:**

I submitted a blog comment containing a malicious JavaScript payload. The payload was accepted by the application and stored on the server.

![Payload Submission](images/payload_submission.png)

---

### Screenshot 2 – Lab Successfully Solved

**What I saw:**

After the stored payload executed when the page was viewed, the PortSwigger lab confirmed successful exploitation of the Stored XSS vulnerability.

![Lab Solved](images/lab_solved.png)

---

## Why Stored XSS Hits Harder

* Persistent execution of attacker-controlled JavaScript.
* Session hijacking through cookie theft.
* Credential theft and phishing attacks.
* Defacement of web pages.
* Unauthorized actions performed on behalf of victims.
* Potential compromise of administrator accounts.
* Large-scale impact affecting every visitor to the vulnerable page.

---

## How I Would Fix It

1. Apply contextual output encoding to all user-generated content.
2. Sanitize HTML before storing or rendering user input.
3. Implement server-side input validation.
4. Deploy a strong Content Security Policy (CSP).
5. Use secure templating frameworks with automatic escaping.
6. Review all locations where user content is rendered.
7. Perform regular security testing for XSS vulnerabilities.

---

## CVSS Score

**CVSS v3.1 Score:** 8.2 (High)

### Vector

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:L/A:N
```

---

## CVSS Justification

### Attack Vector

Network (N) – Exploitable remotely through web requests.

### Attack Complexity

Low (L) – No special conditions are required.

### Privileges Required

None (N) – Any user can submit the payload.

### User Interaction

Required (R) – A victim must view the affected page.

### Scope

Changed (C) – The attack affects users visiting the page.

### Confidentiality Impact

High (H) – Sensitive user information may be exposed.

### Integrity Impact

Low (L) – Page content can be modified by injected scripts.

### Availability Impact

None (N) – The attack does not impact service availability.

---

## References

* OWASP Cross Site Scripting Prevention Cheat Sheet
* OWASP XSS Filter Evasion Cheat Sheet
* PortSwigger Web Security Academy – Stored XSS into HTML Context with Nothing Encoded

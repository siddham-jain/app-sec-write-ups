# Finding the Admin Panel Hiding in Plain Sight

## What I Was After

I was working through the PortSwigger Access Control labs and came across one that looked straightforward on paper: find an unprotected administrator panel and delete a specific user. The target user was:

```text
carlos
```

* **Category:** Access Control
* **Level:** Apprentice
* **Lab:** Unprotected Admin Functionality
* **Status:** Solved

---

## My First Move

Whenever I see a lab about hidden admin panels, my first instinct is to check the obvious places. The `robots.txt` file is one of the most common places where developers accidentally leak sensitive paths, usually because they want to keep search engines from indexing them. But here's the thing: `robots.txt` is publicly readable by anyone.

I navigated to:

```text
/robots.txt
```

---

## What I Found

Sure enough, the file disclosed a restricted administrative path:

```text
Disallow: /administrator-panel
```

I had to smile at that. Someone thought hiding the admin panel from search engines was enough to keep it safe. It was like putting a "Do Not Enter" sign on an unlocked door.

### Screenshot

![robots.txt](images/robots.txt.png)

---

## Walking Right In

I visited the disclosed endpoint directly:

```text
/administrator-panel
```

The application didn't even ask me to log in again. It just showed me the full administrator interface. No role checks, no session validation for admin privileges, nothing. I was looking at a user management panel as a regular authenticated user.

### Screenshot

![Admin Panel](images/admin_panel.png)

---

## Deleting the Target User

Inside the administrator panel, I found the user management functionality. I located:

```text
carlos
```

and deleted him using the available administrative controls. It was that simple.

---

## The Result

The target user was successfully deleted and the lab was marked as solved.

### Screenshot

![Lab Solved](images/lab_solved.png)

---

## Why This Is a Big Problem

This kind of vulnerability opens up serious damage:

* Access to sensitive administrative functionality.
* Performing privileged operations without proper authorization.
* Modifying or deleting application data.
* Compromising the integrity of the entire application.

---

## How to Fix It

If I were securing this application, here's what I would change:

* Implement server-side authorization checks on all administrative endpoints.
* Do not rely on obscurity or hidden URLs for security.
* Restrict access to administrative functionality based on user roles.
* Regularly audit exposed endpoints and sensitive resources.

---

## What Stuck With Me

This lab reinforced a few critical lessons:

* Administrative endpoints must always enforce authorization.
* `robots.txt` should never contain sensitive paths.
* Hidden URLs are not a security control.
* Access control must be validated on every request.

---

## References

* PortSwigger Web Security Academy
* OWASP Access Control Cheat Sheet
* OWASP Top 10 – Broken Access Control

# Scanning the Internal Network with Burp Intruder

## What I Was Up Against

The next SSRF lab on my list was **Basic SSRF Against Another Back-End System**. It was another Apprentice-level challenge, but this time the target was not just localhost. The app had an internal network in the `192.168.0.X` range, and somewhere on that network was an administrative interface. My job was to find it and use it to delete the user **carlos**.

This one felt more like a real engagement. I had to do actual network reconnaissance from inside the app.

---

## How I Found the Vulnerability

I started the same way as the last lab. I opened a product page, clicked **Check Stock**, and intercepted the request in Burp Suite. The familiar `stockApi` parameter showed up:

```http
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

I sent the request to Intruder this time because I knew I would need to brute-force IP addresses.

![Original Stock Request](images/original-stock-request.png)

---

## Setting Up the Network Scan

I modified the payload to target the internal subnet:

```http
stockApi=http://192.168.0.1:8080/admin
```

I highlighted the last octet of the IP and told Intruder to cycle through numbers from:

```text
1 - 255
```

using the **Numbers** payload type. It was a simple setup, but scanning an entire /24 range takes a moment.

![Intruder Setup](images/intruder-setup.png)

---

## Discovering the Hidden Admin Interface

I started the Intruder attack and let it run. Most of the responses came back with errors, which I expected. But then one host returned a successful status code. That was my signal.

The response body contained the admin panel, and just like the previous lab, it had functionality for deleting users.

![Admin Interface](images/admin-interface.png)

---

## Deleting Carlos

With the internal host identified, I crafted the final request:

```http
/admin/delete?username=carlos
```

The server made the request internally and deleted the target user. No muss, no fuss.

---

## Lab Solved

A moment later the lab marked itself as solved.

![Lab Solved](images/lab-solved.png)

---

## Why This Matters

This lab showed me that SSRF is not just about hitting `localhost`. If an app can reach internal networks, I can use it as a proxy to scan, probe, and attack systems that were never meant to touch the internet. In a real environment this could lead to:

- Internal network reconnaissance
- Access to hidden administrative interfaces
- Firewall bypass
- Interaction with internal-only services
- Privilege escalation
- Cloud metadata access

---

## How to Fix It

The defenses here are the same as any SSRF scenario:

1. Implement strict allowlists for outbound requests
2. Block access to localhost and private IP ranges
3. Restrict access to internal administrative interfaces
4. Validate and sanitize user-supplied URLs
5. Use network segmentation
6. Monitor and log outbound requests

---

## What I Learned

This was a great exercise in thinking like an attacker who is already inside the perimeter. The app gave me a window into the internal network, and all I needed was patience and Intruder to map it out. Finding that one live host out of 255 felt like striking gold. It reinforced how critical it is to lock down what an application can reach, not just what users can reach from the outside.

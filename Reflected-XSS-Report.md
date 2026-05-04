# ⚡ Reflected XSS Vulnerability Report

## 📌 Overview

A reflected Cross-Site Scripting (XSS) vulnerability was identified in the search functionality due to improper input sanitization.

---

## 🎯 Vulnerability Type

* Reflected XSS

---

## 🧪 Steps to Reproduce

1. Navigate to search functionality
2. Enter crafted payload in input field
3. Observe input reflected in HTML response
4. Break out of attribute context
5. Inject JavaScript payload

---

## ⚠️ Impact

* Execution of arbitrary JavaScript
* Session hijacking possibilities
* User impersonation

---

## 🧰 Tools Used

* Burp Suite
* Browser

---

## 🛡️ Recommendation

* Proper input validation and output encoding
* Use Content Security Policy (CSP)

---

## 📸 Proof of Concept

<img width="1366" height="768" alt="Screenshot (12)" src="https://github.com/user-attachments/assets/828342ae-f876-4f13-a153-744b3bcfc96c" />



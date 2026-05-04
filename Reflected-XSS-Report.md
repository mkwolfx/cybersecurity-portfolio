## 🧪 Detailed Exploitation Steps (Reflected XSS)

1. **Initial Testing**

   * Tested the search functionality for input reflection
   * Observed that user input was reflected in the HTML response inside an attribute value

   Example context:

   ```html
   <input value="USER_INPUT">
   ```

---

2. **Understanding Context (Context & Grammar)**

   * The payload was reflected inside an **HTML attribute context**
   * In this context, direct script injection is not possible
   * To execute JavaScript, it is necessary to **break out of the attribute value**

---

3. **Breaking Out of Attribute Context**

   * Used a double quote (`"`) to terminate the existing attribute value
   * Used `>` to close the HTML tag

   This allows injection of a new HTML element

---

4. **Payload Construction**

   Payload used:

   ```html
   "><img src=x onerror="alert(1)">
   ```

   **Breakdown:**

   * `"` → Closes the attribute value
   * `>` → Closes the current HTML tag
   * `<img` → Injects a new HTML element
   * `src=x` → Invalid image source to trigger error
   * `onerror=` → Event handler that executes JavaScript
   * `alert(1)` → JavaScript execution (proof of concept)

---

5. **Execution**

   * When the page loads, the injected `<img>` fails to load
   * The `onerror` event is triggered
   * JavaScript executes in the victim’s browser

---

6. **Advanced Payload Testing**

   Additional payloads tested:

   ```html
   "><img src=x onerror="alert(document.cookie)">
   "><iframe src="javascript:alert(1)"></iframe>
   ```

   * Demonstrates ability to execute arbitrary JavaScript
   * Shows potential for session-related attacks

---

7. **Result**

   * Successful execution of JavaScript in browser
   * Confirms presence of **Reflected XSS vulnerability**
---

## 📸 Proof of Concept

<img width="1366" height="768" alt="Screenshot (12)" src="https://github.com/user-attachments/assets/828342ae-f876-4f13-a153-744b3bcfc96c" />



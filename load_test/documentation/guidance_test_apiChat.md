# 🧩 Testing `/api/chat` When You Get “401 Unauthorized”

### **Purpose**

This guide explains how to go from seeing:

```json
{"error": "Unauthorized", "message": "Authentication required"}
```

when calling `/api/chat`,
to successfully making an authenticated **POST** request using the **Flask session cookie** — both with **curl** and **JMeter**.

> ⚠️ **Security Note:**
> Only copy and use session cookies in a **safe development environment**.
> Never embed real session cookies in public code, scripts, or version control.

---

## **1️⃣ Why You Saw the Unauthorized Error**

* The `/api/chat` endpoint is protected by the `@login_required` decorator.
* This decorator checks for a **Flask session (`session['user']`)** — it does *not* validate Bearer tokens.
* Therefore, passing `Authorization: Bearer <token>` will still fail.
* To call `/api/chat`, you must include the **Flask session cookie** created when you logged in via the app’s browser UI.

---

## **2️⃣ Quick Verification with curl**

Use curl with verbose mode to confirm your request setup:

```bash
curl -v -X POST "https://<your-app-domain>/api/chat" \
  -H "Content-Type: application/json" \
  -H "Cookie: session=<YOUR_SESSION_COOKIE_VALUE>" \
  -d '{
    "message": "Hello from curl!",
    "conversation_id": null,
    "model_deployment": "<your-model-deployment-name>"
  }'
```

### **Check results**

* **401 Unauthorized** → The server did not find a valid session.
* **405 Method Not Allowed** → You likely used `GET` instead of `POST`, or your route definition doesn’t allow POST.

---

## **3️⃣ Obtain the Flask Session Cookie from Edge DevTools**

1. Log into your app normally using the web UI (`/login`).
2. Open **DevTools (F12)** → **Application (Storage)** tab.
3. In the left sidebar:
   `Storage → Cookies → <your-app-domain>`
4. Locate the cookie named **`session`**.

   * Ignore cookies like `AppServiceAuthSession` — that’s Azure’s EasyAuth.
   * You only need the **Flask session cookie**.
5. Right-click the cookie → **Copy → Copy Value**.

   * Or double-click the **Value** cell and copy it manually.
6. Save the cookie value somewhere safe.

> 💡 Tip:
> If your cookie name is different (due to `SESSION_COOKIE_NAME` config), use that name instead of `session`.

---

## **4️⃣ Test with curl Using the Session Cookie**

Paste your copied cookie value in place of `<YOUR_SESSION_COOKIE_VALUE>`.

Example:

```bash
curl -X POST "https://simplechat-demo.azurewebsites.us/api/chat" \
  -H "Content-Type: application/json" \
  -H "Cookie: session=.eJwlz..." \
  -d '{"message":"test message"}'
```

### **Expected result**

✅ The server should return a **200 OK** or a normal response payload —
not `{"error":"Unauthorized","message":"Authentication required"}`.

---

## **5️⃣ Building the Same Request in JMeter**

When configuring JMeter to call `/api/chat`:

1. Add an **HTTP Request Sampler**.

   * **Method:** `POST`
   * **Path:** `/api/chat`
   * **Body Data:**

     ```json
     {
       "message": "Hello from JMeter!",
       "conversation_id": null,
       "model_deployment": "<your-model-deployment-name>"
     }
     ```
2. In the **HTTP Header Manager**, add:

   * `Content-Type: application/json`
   * `Cookie: session=<YOUR_SESSION_COOKIE_VALUE>`
3. Run the test.

If configured correctly, JMeter will reuse the Flask session just like your browser login — and `/api/chat` will authenticate successfully.

---

### ✅ **Summary**

| Problem                | Fix                                   |
| ---------------------- | ------------------------------------- |
| 401 Unauthorized       | Include Flask `session` cookie        |
| 405 Method Not Allowed | Ensure `POST` method is used          |
| Still failing          | Verify cookie domain matches app host |
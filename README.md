# 🛡️ JWT Authentication Bypass via Flawed Signature Verification for Bug Bounty Hunting

**Author:** Aditya Bhatt <br/>
**Category:** Bug Bounty / Web App Security / JWT Exploitation <br/>
**Lab:** JWT authentication bypass via flawed signature verification <br/>

> Exploit insecure JWT signature verification by forcing the algorithm to `none` and stripping the signature to gain unauthorized admin access.

---

## 🔍 Introduction

JWTs (JSON Web Tokens) are a common method for stateless authentication, where a token is issued after login and validated on subsequent requests. However, misconfigurations can lead to critical security flaws — such as accepting tokens with no signature at all.

In this lab, we’ll exploit such a misconfiguration to forge a token without a signature and gain administrator access.

![Cover](https://github.com/user-attachments/assets/bff71e26-d75a-43b3-bd40-51694e823868) <br/>

---

## 🔐 JWT in a Nutshell: The 3 Key Parts

A JWT consists of three base64-encoded components, separated by dots (`.`):

```
<Header>.<Payload>.<Signature>
```

### 1. **Header**

Specifies metadata about the algorithm used for signing:

```json
{ "alg": "HS256", "typ": "JWT" }
```

### 2. **Payload**

Contains user claims such as user ID or privileges:

```json
{ "sub": "wiener", "admin": false }
```

### 3. **Signature**

Used to verify the authenticity of the token. It is created by hashing the header and payload with a secret key.

---

**⚠️ Vulnerability Focus:**
If the server accepts a token with `alg: none` and ignores the signature, an attacker can forge tokens with arbitrary data and bypass authentication.

---

## 🎯 Lab Objective

> Modify your JWT to access the `/admin` panel as the `administrator` user and delete `carlos`.

---

## 🧪 Proof-of-Concept (PoC)

**Step-by-step Walkthrough:**

1. **Access the JWT Lab**
   Visit: [Lab URL](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification) and log in with the credentials:

   ```
   Username: wiener  
   Password: peter
   ```
  
  ![1](https://github.com/user-attachments/assets/e1ff83ef-a0ab-49fc-92e1-f828f787e740) <br/>
  
2. **Try the `/admin` Endpoint**
   Intercept the request to `/admin` using Burp Suite. You'll get:

   ```
   Admin interface only available if logged in as an administrator
   ```
  ![2](https://github.com/user-attachments/assets/32e575f1-5468-42a0-a8d4-ebb593bcc27a) <br/>
  
3. **Send to Repeater**
   Send the request to **Burp Repeater**. Observe that the JWT token's payload contains:

   ```json
   "sub": "wiener"
   ```
  
  ![3](https://github.com/user-attachments/assets/270f28bb-9425-4987-8279-f95aa1fe1ff5) <br/>

4. **Modify Payload**
   In the **Inspector** panel, update the `sub` claim from `wiener` to `administrator`. Click **Apply changes**.

  ![administrator](https://github.com/user-attachments/assets/6e6cef27-5695-4b15-b988-19e570b8263a) <br/>

5. **Change Header Algorithm to None**
   Still in the Inspector, edit the JWT header to:

   ```json
   { "alg": "none" }
   ```

   ![Alg](https://github.com/user-attachments/assets/78c56ee4-ccf4-4563-8488-1488cc17ce6f) <br/>
   
   Apply the changes.

7. **Strip Signature**
   In the raw request editor, remove the signature segment of the JWT entirely — **but keep the trailing dot**. Your token should look like:

   ```
   <base64(header)>.<base64(payload)>.
   ```
  
  ![Alg](https://github.com/user-attachments/assets/78c56ee4-ccf4-4563-8488-1488cc17ce6f) <br/>
  
8. **Send Request to /admin**
   Hit **Send**. Boom — you're now in the **admin panel** as `administrator`.

  ![5](https://github.com/user-attachments/assets/57825355-50c9-4a68-becb-bff6f3b27e02) <br/>


9. **Delete Carlos**
   In the response, find the delete endpoint:

   ```
   /admin/delete?username=carlos
   ```

   Send a request to this endpoint using Burp or browser to solve the lab.

   ![Final](https://github.com/user-attachments/assets/69ee269f-21c1-4e66-93a9-c0b9a47d2c73) <br/>

---

## ✅ Lab Status: **Solved**

You’ve successfully exploited a JWT misconfiguration where the server accepts unsigned tokens — a classic yet dangerous flaw.

---

## 🧠 Key Takeaways

* Always validate JWT signatures on the server-side.
* Reject tokens that use `alg: none`.
* Use libraries that enforce strict algorithm checks and signature verification.
* Do not trust client-controlled data blindly — especially authentication tokens.

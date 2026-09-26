You are correct—when you have **multiple separate Vue apps** (not just multiple pages in a single Vue SPA), Firebase Authentication **does not automatically share authentication state between them**, even if they are on the same domain or subdomain. The default `local` persistence only works within the same app context (typically, the same JavaScript bundle and origin)[1](https://firebase.google.com/docs/auth/web/auth-state-persistence)[2](https://learnvue.co/articles/vue-firebase-authentication)[3](https://www.reddit.com/r/Firebase/comments/138n03u/sharing_firebase_authentication_across_multiple/).

## Why Doesn't Auth State Persist Across Multiple Vue Apps?

- **Scope of Persistence:**  
    Firebase Auth's `local` persistence stores the session in browser storage (like `localStorage`), but the session is scoped to the app's Firebase configuration and bundle. If you have multiple, independently built Vue apps (each with their own Firebase initialization), they do not share the in-memory Firebase Auth instance—even if they use the same Firebase project and config[3](https://www.reddit.com/r/Firebase/comments/138n03u/sharing_firebase_authentication_across_multiple/)[1](https://firebase.google.com/docs/auth/web/auth-state-persistence).
    
- **Cross-App Limitations:**  
    When you load a new Vue app, it initializes a new Firebase App instance, which does not automatically recognize the user session from another app's instance, even if the browser's localStorage contains the session data. This is due to how the Firebase Auth SDK manages its internal state and listeners[3](https://www.reddit.com/r/Firebase/comments/138n03u/sharing_firebase_authentication_across_multiple/).
    

## How Can You Share Auth State Across Multiple Apps?

To **share authentication between multiple Vue apps** (for example, on different subdomains or as completely separate deployments), you need to implement a custom solution. The recommended approach is:

1. **Generate a Firebase Custom Token in the Original App:**
    
    - When a user is authenticated in App A, use a backend service (with Firebase Admin SDK) to generate a custom token for that user's UID[3](https://www.reddit.com/r/Firebase/comments/138n03u/sharing_firebase_authentication_across_multiple/).
        
2. **Pass the Custom Token to the Other App:**
    
    - Send this token securely to App B (e.g., via redirect, postMessage, or API call).
        
3. **Sign In with the Custom Token in the Other App:**
    
    - In App B, use `signInWithCustomToken()` from the Firebase Auth SDK to authenticate the user with the received token. This will establish the user's session in App B[3](https://www.reddit.com/r/Firebase/comments/138n03u/sharing_firebase_authentication_across_multiple/).
        

**Example:**
```js
// In App B 
import { getAuth, signInWithCustomToken } from "firebase/auth"; 

const auth = getAuth(); 
signInWithCustomToken(auth, tokenFromAppA)   
.then((userCredential) => {    
	// User is now signed in to App B  
})  
.catch((error) => {    // Handle errors  });`
```
This process ensures that the user's authentication state is established in each app as needed, rather than relying on automatic session sharing[3](https://www.reddit.com/r/Firebase/comments/138n03u/sharing_firebase_authentication_across_multiple/).
## Summary Table

| Scenario                               | Default Firebase Auth Behavior          | Solution for Shared Auth State      |
| -------------------------------------- | --------------------------------------- | ----------------------------------- |
| Multiple pages in one Vue SPA          | Session is shared automatically         | No extra work needed                |
| Multiple, independently built Vue apps | Session is **not** shared automatically | Use custom token flow as described  |
| Apps on different domains              | Session is **not** shared automatically | Use backend-generated custom tokens |

---
**In summary:**  
Firebase Auth's built-in persistence does not automatically keep users logged in across multiple, separately-initialized Vue apps. To share authentication between them, you must implement a custom token exchange flow using the Firebase Admin SDK on your backend and `signInWithCustomToken()` on the client


# But how to get a custom token in the first place?
To generate a custom token for a user in **App A** using the Firebase Admin SDK, follow these steps:

---
## Step 1: Set Up Firebase Admin SDK on Your Backend
First, initialize the Firebase Admin SDK in your backend (e.g., Node.js server):
```js
// appA-backend.js 
import { initializeApp, cert } from 'firebase-admin/app'; 
import { getAuth } from 'firebase-admin/auth'; 

// Initialize with your service account credentials 
const serviceAccount = require('./serviceAccountKey.json'); 

// Download from Firebase Console 
const appAAdmin = initializeApp({   credential: cert(serviceAccount), }); 
const auth = getAuth(appAAdmin);
```
---

## Step 2: Create an Endpoint to Generate Custom Tokens

When a user is authenticated in App A, send their Firebase ID token to your backend to verify their identity and generate a custom token:

js

`// Example Express.js endpoint app.post('/generate-custom-token', async (req, res) => {   try {    // Verify the ID token sent from App A's client    const idToken = req.body.idToken;    const decodedToken = await auth.verifyIdToken(idToken);    const uid = decodedToken.uid; // Extract the user's UID     // Generate a custom token for this UID    const customToken = await auth.createCustomToken(uid, {      // Optional custom claims (e.g., roles)      appAUser: true,    });     res.json({ customToken });  } catch (error) {    res.status(401).json({ error: 'Invalid token' });  } });`

---

## Step 3: Client-Side Flow in App A

In App A's client (Vue app), after the user logs in:

1. Get the user's ID token.
    
2. Send it to your backend to exchange for a custom token.
    
3. Pass the custom token to App B (e.g., via URL parameters, localStorage, or API).
    

js

``// App A's client (Vue) import { getAuth } from 'firebase/auth'; const auth = getAuth(); const user = auth.currentUser; user.getIdToken().then((idToken) => {   // Send ID token to your backend  fetch('https://your-api.com/generate-custom-token', {    method: 'POST',    body: JSON.stringify({ idToken }),  })    .then((response) => response.json())    .then((data) => {      const customToken = data.customToken;      // Pass this token to App B (example: redirect)      window.location.href = `https://appB.com?token=${customToken}`;    }); });``

---

## Step 4: App B Uses the Custom Token

In App B's client (Vue app), use the custom token to authenticate:

js

`// App B's client (Vue) import { getAuth, signInWithCustomToken } from 'firebase/auth'; const auth = getAuth(); const urlParams = new URLSearchParams(window.location.search); const customToken = urlParams.get('token'); signInWithCustomToken(auth, customToken)   .then((userCredential) => {    // User is now authenticated in App B  })  .catch((error) => {    console.error('Authentication failed:', error);  });`

---

## Key Notes:

- **Security**: Always use HTTPS for token transmission.
    
- **Token Expiry**: Custom tokens expire after 1 hour, but Firebase client SDKs automatically refresh sessions.
    
- **Claims**: Add custom claims (e.g., `appAUser: true`) to embed additional user data accessible in Firebase Security Rules.
    
- **Service Account**: Keep your `serviceAccountKey.json` secure—never expose it to clients.
    

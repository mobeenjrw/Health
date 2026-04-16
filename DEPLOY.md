# NutriPak v3 — Firebase Deployment Guide

## آپ کا Firebase project: health-meter-94f3e

### Step 1 — Firebase Tools Install
```bash
npm install -g firebase-tools
firebase login
```

### Step 2 — Project Setup
```bash
cd nutripak-v3
firebase init hosting
# Select: health-meter-94f3e
# Public directory: . (current folder)
# Single page app: No
# Overwrite index.html: No
```

### Step 3 — Deploy
```bash
firebase deploy --only hosting
```
Your app will be live at:
https://health-meter-94f3e.web.app

---

## API KEY SECURITY — IMPORTANT

The Anthropic API key is called from the browser in this version.
For production security, create a Firebase Cloud Function:

### functions/index.js
```javascript
const functions = require('firebase-functions');
const fetch = require('node-fetch');

exports.askClaude = functions.https.onCall(async (data, context) => {
  if (!context.auth) throw new functions.https.HttpsError('unauthenticated', 'Login required');
  
  const resp = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': functions.config().anthropic.key,  // stored securely
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify(data.payload)
  });
  return resp.json();
});
```

### Set API key in Firebase:
```bash
firebase functions:config:set anthropic.key="YOUR_ANTHROPIC_API_KEY"
firebase deploy --only functions
```

### In index.html, replace fetch() with:
```javascript
const askClaude = firebase.functions().httpsCallable('askClaude');
const result = await askClaude({ payload: { model: '...', messages: [...] } });
```

---

## Firestore Rules (Firebase Console > Firestore > Rules)
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
    match /logs/{logId} {
      allow read, write: if request.auth != null && 
        resource.data.uid == request.auth.uid;
    }
  }
}
```

# Firebase Database Module

## Overview
This module initializes and configures Firebase’s data services for an Expo-React Native application. It exposes:
- Firestore (FB_DB) for real-time NoSQL storage  
- Cloud Storage (FB_STORE) for file uploads/downloads  
Configuration values are sourced from environment variables, ensuring the same Firebase app context (FB_APP) across services.

## Key Features
- **Firestore Initialization (FB_DB)**: Provides a ready-to-use Firestore instance for CRUD operations on documents and collections.  
- **Cloud Storage Initialization (FB_STORE)**: Offers a configured Storage instance for uploading, downloading, and managing files.  
- **Environment-Driven Configuration**: Reads all Firebase credentials (apiKey, authDomain, projectId, etc.) from environment variables, enabling easy per-environment setups.  
- **Single Firebase App Context (FB_APP)**: Ensures all exported services share the same underlying Firebase application instance.

## System Errors
- **Missing Configuration Variables**  
  Description: One or more `process.env.*` keys (APIKEY, AUTHDOMAIN, etc.) are undefined.  
  Resolution: Verify that the `.env` file or CI/CD environment settings define all required Firebase variables and that they’re correctly prefixed for Expo.

- **Permission Denied (Firestore / Storage)**  
  Description: Operations are blocked by Firebase security rules.  
  Resolution: Review and update Firestore/Storage rules in the Firebase Console to allow the intended operations for authenticated or public users as required.

- **Network Unavailable**  
  Description: Requests to Firestore or Storage fail when the device is offline.  
  Resolution: Implement offline persistence (Firestore supports built-in caching) or perform a connectivity check before attempting data operations.

## Usage Examples
```javascript
import { FB_DB, FB_STORE } from '../firebaseconfig';
import { collection, getDocs, addDoc } from 'firebase/firestore';
import { ref, uploadBytes, getDownloadURL } from 'firebase/storage';

// Fetch all documents from 'users' collection
async function fetchUsers() {
  const usersCol = collection(FB_DB, 'users');
  const snapshot = await getDocs(usersCol);
  return snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
}

// Add a new user document
async function createUser(userData) {
  const usersCol = collection(FB_DB, 'users');
  return await addDoc(usersCol, userData);
}

// Upload a profile image to Cloud Storage and get its download URL
async function uploadProfileImage(fileUri, userId) {
  const response = await fetch(fileUri);
  const blob = await response.blob();
  const storageRef = ref(FB_STORE, `profiles/${userId}.jpg`);
  await uploadBytes(storageRef, blob);
  return await getDownloadURL(storageRef);
}
```

## System Integration
```mermaid
flowchart LR
  envVars["Environment Variables"] --> firebaseConfig["Firebase Configuration Module"]
  firebaseConfig --> firestore["Firestore Service (FB_DB)"]
  firebaseConfig --> storage["Storage Service (FB_STORE)"]
  firestore --> dataLayer["Data Layer / Business Logic"]
  storage --> fileModule["File Management Module"]
  dataLayer --> ui["UI Components"]
  fileModule --> ui
```
## **Firebase with Next.js**

1. **Serverless & Scalable**  
   - Firebase is **serverless**, meaning you don’t have to manage backend infrastructure.
   - It scales automatically, making it a great choice for apps that expect growth.

2. **Realtime Database & Firestore**  
   - Firestore allows real-time synchronization, which is useful for dynamic applications like chat apps, dashboards, and social feeds.

3. **Authentication & Security**  
   - Firebase Authentication supports **Google, Facebook, Twitter, GitHub, Apple, and email/password** authentication.
   - Firebase’s authentication integrates well with Next.js API routes (`/api`).

4. **Storage (File Uploads)**  
   - Firebase Storage allows uploading images, videos, and other files securely.

5. **Edge Functions & API Routes**  
   - Next.js API routes (`/api`) can be used as a backend to communicate with Firebase.
   - Firebase Cloud Functions can also be leveraged for additional backend logic.

6. **SSR & Static Site Generation (SSG) Support**  
   - You can use Firestore with `getServerSideProps` (SSR) or `getStaticProps` (SSG) to fetch data at request time or build time.
   - Firebase Authentication state can be handled in the client, while protected routes can be managed with `middleware.js`.

---

## **How to Use Firebase with Next.js?**

Here’s how you can set up Firebase Firestore, Auth, and Storage in a Next.js app.

### **Install Firebase SDK**
```bash
npm install firebase
```

---

### **Configure Firebase (firebaseConfig.js)**

Create a file `firebaseConfig.js` in the root of your project:

```javascript
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";
import { getStorage } from "firebase/storage";

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);

// Export services
export const auth = getAuth(app);
export const db = getFirestore(app);
export const storage = getStorage(app);
```
> **Why use environment variables?**  
> Storing API keys in `.env.local` prevents them from being exposed in your public code.

---

### **Firestore Database (CRUD Operations)**

#### **Write Data to Firestore**
```javascript
import { db } from "../firebaseConfig";
import { collection, addDoc } from "firebase/firestore";

export const addUser = async (name, email) => {
  try {
    await addDoc(collection(db, "users"), { name, email });
    console.log("User added!");
  } catch (error) {
    console.error("Error adding user:", error);
  }
};
```

#### **Read Data from Firestore**
```javascript
import { db } from "../firebaseConfig";
import { collection, getDocs } from "firebase/firestore";

export const fetchUsers = async () => {
  try {
    const querySnapshot = await getDocs(collection(db, "users"));
    return querySnapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
  } catch (error) {
    console.error("Error fetching users:", error);
    return [];
  }
};
```

#### **Using Firestore Data in a Next.js Page (Server-Side Rendering)**
```javascript
import { fetchUsers } from "../lib/firebaseUtils";

export async function getServerSideProps() {
  const users = await fetchUsers();
  return { props: { users } };
}

export default function UsersPage({ users }) {
  return (
    <div>
      <h1>Users</h1>
      {users.map(user => (
        <p key={user.id}>{user.name} - {user.email}</p>
      ))}
    </div>
  );
}
```

---

### **Firebase Authentication in Next.js**
#### **Sign Up with Email & Password**
```javascript
import { auth } from "../firebaseConfig";
import { createUserWithEmailAndPassword } from "firebase/auth";

export const signUp = async (email, password) => {
  try {
    const userCredential = await createUserWithEmailAndPassword(auth, email, password);
    console.log("User signed up:", userCredential.user);
  } catch (error) {
    console.error("Sign-up error:", error);
  }
};
```

#### **Login User**
```javascript
import { signInWithEmailAndPassword } from "firebase/auth";

export const login = async (email, password) => {
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email, password);
    console.log("User logged in:", userCredential.user);
  } catch (error) {
    console.error("Login error:", error);
  }
};
```

#### **Sign Out User**
```javascript
import { signOut } from "firebase/auth";

export const logout = async () => {
  try {
    await signOut(auth);
    console.log("User signed out");
  } catch (error) {
    console.error("Logout error:", error);
  }
};
```

#### **Check Auth State in Next.js (Client-side)**
```javascript
import { useEffect, useState } from "react";
import { auth } from "../firebaseConfig";
import { onAuthStateChanged } from "firebase/auth";

export default function useAuth() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setUser(user);
    });
    return () => unsubscribe();
  }, []);

  return user;
}
```

---

### ** Firebase Storage (File Uploading)**
#### **Upload Image to Firebase Storage**
```javascript
import { storage } from "../firebaseConfig";
import { ref, uploadBytes, getDownloadURL } from "firebase/storage";

export const uploadImage = async (file) => {
  try {
    const storageRef = ref(storage, `images/${file.name}`);
    const snapshot = await uploadBytes(storageRef, file);
    const downloadURL = await getDownloadURL(snapshot.ref);
    return downloadURL;
  } catch (error) {
    console.error("Upload error:", error);
  }
};
```

#### **Upload Image in Next.js Component**
```javascript
import { useState } from "react";
import { uploadImage } from "../lib/firebaseUtils";

export default function UploadForm() {
  const [file, setFile] = useState(null);
  const [url, setUrl] = useState("");

  const handleUpload = async () => {
    if (!file) return;
    const uploadedUrl = await uploadImage(file);
    setUrl(uploadedUrl);
  };

  return (
    <div>
      <input type="file" onChange={(e) => setFile(e.target.files[0])} />
      <button onClick={handleUpload}>Upload</button>
      {url && <img src={url} alt="Uploaded" width="200" />}
    </div>
  );
}
```

---


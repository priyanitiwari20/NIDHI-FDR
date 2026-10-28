# Evidentia Frontend Architecture & Flow

Toh Judge sir, yeh hamara **React (Vite) frontend** ka architecture hai. Humne isko ekdum modern, fast aur highly modular banaya hai taaki code maintainable rahe. Is project mein **Tailwind CSS** styling ke liye aur **Framer Motion** smooth animations ke liye use hua hai.

Neeche poore frontend ka breakdown aur flow diya gaya hai:

---

## 1. Core Configuration & API (Dimag aur Nashein)
Yeh files frontend ka base setup aur backend se communication handle karti hain:

* **`src/api/axios.js`**: Yeh hamara API client hai. Isme humne Axios ka instance banaya hai jo har request ke saath user ka JWT (JSON Web Token) bhejta hai. Agar token expire ho jaye, toh yeh automatically user ko login page par redirect kar deta hai.
* **`src/context/AuthContext.jsx`**: Yeh poore app mein user ka state (login hai ya nahi, admin hai ya issuer) manage karta hai. Jab user login karta hai, toh token aur details yahan save hoti hain aur poore app mein accessible rehti hain.
* **`src/context/ThemeContext.jsx`**: Yeh light/dark mode ko handle karta hai.

---

## 2. Shared Components & UI (`src/components/`)
Humne UI ko chote-chote reusable components mein break kar diya hai taaki code repeat na ho.

* **`ui/` folder**: Isme saare basic building blocks hain jaise `button.jsx`, `input.jsx`, `card.jsx`, `modal.jsx` vagaira. Yeh components Tailwind se styled hain.
* **`Layout.jsx`**: Yeh hamara main wrapper hai. Isme navigation bar (Header) aur sidebar hota hai. User ke role (Issuer, Verifier, Admin) ke hisaab se yeh automatically menu options change kar deta hai.
* **`UploadBox.jsx`**: Yeh file upload area hai jahan drag-and-drop animations kaam karte hain (documents verify karne ke liye).
* **`HeatmapViewer.jsx`**: Jab forensic analysis hoti hai, toh yeh component image ke altered parts par red heatmap draw karta hai.
* **`Stepper.jsx`**: Jab backend koi heavy task (jaise verification ya issuing) kar raha hota hai, toh yeh real-time mein steps progress dikhata hai.

---

## 3. The 3 Main User Flows (Pages)

Hamara platform 3 alag alag roles ke liye kaam karta hai. Yahan unka flow hai:

### A. The Public / Unauthenticated Flow
Jab koi user pehli baar website par aata hai.
* **`pages/landing/LandingPage.jsx`**: Yeh hamari beautiful landing page hai. Isme alag alag sections hain (`HeroSection.jsx`, `HowItWorksSection.jsx`, etc.) jo product ki features explain karte hain.
* **`pages/Login.jsx` & `pages/Register.jsx`**: Authentication pages. Register karne par ek naya Issuer account banta hai backend mein.
* **`pages/public/PublicVerify.jsx`**: Bina account banaye log directly QR code scan karke ya link click karke is page par aa sakte hain document verify karne ke liye.

### B. The Issuer Flow (Jo document banata hai)
Jab ek University ya Company login karti hai.
* **`pages/issuer/Dashboard.jsx`**: Login ke baad yeh page unhe stats dikhata hai (kitne certs issue kiye, recent activity kya hai).
* **`pages/issuer/IssueDocument.jsx`**: Yeh sabse important page hai. Yahan issuer details (Name, Course) daalta hai. Right side par ek **Live Preview** dikhta hai ki certificate kaisa lagega. Jab woh "Issue" click karta hai, toh details backend par jaati hain, Python worker PDF banata hai, us par QR lagata hai, aur cryptographic signature karke registry mein save kar deta hai.
* **`pages/issuer/MyDocuments.jsx`**: Yeh ek table view hai jahan issuer apne saare purane documents dekh sakta hai aur unhe Revoke (cancel) bhi kar sakta hai.

### C. The Verifier Flow (Jo document check karta hai)
Jab koi Employer check karna chahta hai ki document asli hai ya nakli.
* **`pages/verifier/VerifyPage.jsx`**: Yahan user apna PDF ya scanned image drag-and-drop karta hai. 
* *Flow*: File upload hoti hai -> `Stepper.jsx` progress dikhata hai -> Backend mein Python worker OCR, QR decoding, aur forensic checks run karta hai.
* **`pages/verifier/ResultPage.jsx`**: Verification ke baad yeh page khulta hai. Yahan ek bada Verdict Badge (`VerdictBadge.jsx`) dikhta hai jo batata hai ki document **"Genuine"**, **"Altered"**, ya **"Forged"** hai. Agar document altered hai, toh yahan `HeatmapViewer.jsx` forensic evidence dikhata hai ki kis hisse mein ched-chad (tampering) hui hai.

---

## 4. Admin Flow
* **`pages/admin/AuditLog.jsx`**: System mein hone wali har ek activity (kisne login kiya, kisne certificate banaya) ek tamper-proof ledger mein save hoti hai. Admin is page par aakar saari logs dekh sakta hai aur verify kar sakta hai ki ledger ke hashes matched hain aur koi hack nahi hua.

### Final Summary for the Judge:
*"Sir, the frontend is a React Single Page Application (SPA). The flow starts from the Landing page, routes users to role-specific dashboards, and uses a centralized AuthContext for security. The code is highly modularized into reusable UI components, and all API calls are abstracted through Axios interceptors. The heavy lifting is offloaded to the backend, keeping the frontend fast, responsive, and purely focused on a seamless user experience."*

# Tractor Hisab Pro — Multi-user SaaS starter

यह GitHub Pages + Firebase Authentication + Firestore आधारित multi-user PWA है।

## Multi-user data separation
- हर ग्राहक अलग Firebase Authentication account (Login ID के रूप में ईमेल + password) से लॉगिन करता है।
- Firebase हर account को unique UID देता है। सभी डेटा `users/{UID}/...` के अंदर सेव होता है।
- `firestore.rules` सिर्फ उसी UID को उसका डेटा पढ़ने/लिखने देता है। दो ग्राहकों का डेटा आपस में नहीं मिलता।
- एक ग्राहक अपने खाते में कई ट्रैक्टर, ड्राइवर, किसान और एक साथ कई काम दर्ज कर सकता है।

## Features
- Multiple tractors and tractor-wise work/income/fuel cost
- Driver-wise work and earnings
- Farmer history, total due, advance and later payments
- Work charging: hourly, bigha/acre, trolley trips, fixed, per-job
- WhatsApp payment/due message (WhatsApp opens with prefilled text; user presses Send)
- Printable payment and farmer due receipts (Print > Save as PDF)
- Fuel litres, cost and optional engine-hour meter reading
- Expenses, dashboard, date/tractor/driver filtered report, CSV export
- JSON backup/restore and mobile PWA shell

## Firebase setup (Spark)
1. Firebase Console में project बनाएं।
2. Authentication > Sign-in method में Email/Password enable करें।
3. Project settings > General > Your apps > Web App बनाएं और config लें।
4. `js/firebase-config.js` में Firebase config values भरें।
5. Firestore Database create करें।
6. Firestore Rules में `firestore.rules` का code paste करके Publish करें।
7. Authentication > Settings > Authorized domains में अपना GitHub Pages host या custom domain जोड़ें।
8. अपना पहला account signup से बनाकर login test करें।

## ग्राहकों को अलग Login ID / Password देना
1. Firebase Console > Authentication > Users > Add user खोलें।
2. ग्राहक का unique email (यही Login ID होगा) और temporary password बनाएं।
3. हर ग्राहक को उसका अलग Login ID/password दें। Firebase प्रत्येक को अलग UID देगा।
4. बिक्री के बाद सार्वजनिक signup बंद करने के लिए `js/firebase-config.js` में `ALLOW_SELF_SIGNUP = false` करें।
5. फिर नए ग्राहक केवल Firebase Console से बनाएँ। Password reset के लिए Firebase Console का उपयोग करें।

**सुरक्षा:** Firebase Admin SDK या service-account private key कभी भी HTML/JavaScript में न रखें। Browser-side code से दूसरे users के accounts बनाना सुरक्षित नहीं है। इस संस्करण में ग्राहक account provisioning Firebase Console से manual है। In-app Super Admin, subscription expiry, payment collection और secure user creation के लिए अगला backend चरण (Cloudflare Worker/secure admin service) आवश्यक होगा।

## GitHub Pages deployment
1. ZIP extract करें और सभी contents repository root में upload करें (`index.html` root में रहे)।
2. GitHub repository > Settings > Pages > Deploy from a branch > `main` > `/ (root)` चुनें।
3. Save करें और URL खोलें।

## हिसाब संबंधी सीमाएँ
- काम पर दर्ज advance उसी काम की देय राशि में घटता है। बाद के भुगतान किसान के कुल बकाया से घटते हैं।
- ट्रैक्टर-वार डीजल लीटर और खर्च दिखते हैं। Engine-hour reading store होती है; सटीक प्रति घंटा fuel efficiency निकालने के लिए अगली reading-difference आधारित रिपोर्ट बनानी होगी।
- लाभ = काम का कुल किराया − दर्ज डीजल खर्च − दर्ज अन्य खर्च। यह अनुमानित हिसाब है।
- WhatsApp संदेश prefill होता है; ऐप अपने आप संदेश नहीं भेजता।
- PDF के लिए print dialog में Save as PDF चुनें।
- यह starter अभी प्रत्येक ग्राहक के लिए एक owner login है। Staff sub-login/permissions, subscription billing और in-app Super Admin panel शामिल नहीं हैं।

## Production testing
दो अलग test accounts बनाकर दोनों में अलग-अलग tractor/farmer/work data डालें और पुष्टि करें कि account A में account B का data नहीं दिखता। Rules को कभी public न करें। नियमित JSON backup रखें।


## Subscription system (Cloudflare Worker + Firestore)
1. Deploy `worker.js` to the existing Cloudflare Worker `tractor-hisab-pro-api`.
2. Keep existing variables/secrets: `FIREBASE_API_KEY`, `FIREBASE_PROJECT_ID`, `SUPER_ADMIN_UID`, `ALLOWED_ORIGIN`, and `FIREBASE_SERVICE_ACCOUNT_JSON`. The JSON secret must be the service-account JSON for this same Firebase/Google Cloud project.
3. In Google Cloud, the service account needs permission to access Firestore (Firebase Admin SDK Administrator is sufficient for this starter).
4. Publish the included `firestore.rules` in Firebase Console > Firestore Database > Rules. This is required for expiry enforcement. The rules allow only each UID's own `/users/{uid}/...` records while its server-managed subscription is active; subscription documents are server-only.
5. Deploy the updated website files to Cloudflare Pages. Set `API_BASE` in `app.js` only if the Worker URL changes.
6. New users receive a 15-day trial when they first log in after this version is deployed. Existing users without a subscription also receive a 15-day trial on their first login after deployment. Existing records are not deleted.
7. Super Admin signs in with the configured UID. In Settings > Customer Subscription Management, enter a customer's Firebase Authentication UID and choose 365 days, 730 days, 30 days, or a custom duration (1–1095 days).

Important: publish Firestore Rules before using the updated site. Otherwise the Worker may create a trial record but the app's data reads will be denied. Test with a separate customer account first. Keep the service-account JSON only in Cloudflare Secret storage; never commit it to GitHub or put it in frontend files. This starter implements manual subscription grants, not online payment collection or automatic payment verification.

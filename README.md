# KwadoCrypt
Hash passwords and compare for similarities

---

All what you are about to read would have been unnecessary but users do forget their passwords, if not, we can just collect old passwords and compare with new one but like i said users do forget their passwords, and that has brought us here. : 🥵 :
---

Let me walk you on how to Hash passwords and still be able to compare for similarities.

>[!NOTE]
>There is a difference between checking for exact match and checking for similarities, we will do both, although the latter is why i am writing this.

We will be working with LSH (Locality sensitive hashing) which allows similar plaintext to have similar hashes hence, no avalanche effect.

---

## Perequisites
### packages
Bcrypt (for Deterministic and avalanche effect hash algo)

simHash (for LSH-based hashing)

crypto (for AES-256 encryption and decryption)

dotenv (for reading env file)

## 1. Create env file and save secret encryption key and Initialization vector key.
---

```
.env

AES_KEY=yourAESsecretKey

IV_KEY=yourIVsecretKey
```

You can generate random strings to use as your secret key, this should not be exposed.

## 2. hash password
---

We will be hashing the password twice, using bcrypt and again using simHash for LSH hashing.

```javascript
JavaScript

const bcrypt = require('bcrypt');
const simHash = require('simHash');

const password = password123;

//gen salt
const salt = bcrypt.genSalt(10);

//hash password with bcrypt
const passwordPri = bcrypt.hash(password, salt);

//hash password with LSH
let passwordLSH = simHash(password);
```

#### code explanation
* import packages
* have your password ready (password123)
* generate salt using bcrypt
* hash password with bcrypt (passwordPri)
* hash password with simHash (passwordLSH)

## 3. Encrypt LSH hashed password

```javascript
JavaScript

const crypto = require('crypto');
const dotenv = require('dotenv');

//import secret keys from .env file
const key = process.env.AES_KEY;
const iv = process.env.IV_KEY;

//Encryption function
function encryptLSH(lshHash, secretKey, secretIV) {
    const cipher = crypto.createCipheriv('aes-256-cbc', secretKey, secretIV);
    let encrypted = cipher.update(lshHash, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    return { encryptedData: encrypted, iv: iv.toString('hex') };
}

//Encrypt LSH hash, pass your LSH hashed password (passwordLSH), secret key and iv.
const encryptedPasswordLSH = encryptLSH(passwordLSH, key, iv);
```

#### code explanation
* Import packages
* Import keys from .env file
* Define and create encryption function. You can tweak as you want.
* Call function and pass arguments.

## 4. save passwords to db
---

```javascript
JavaScript

//You should have both your bcrypt hashed password (passwordPri) and your LSH hashed and AES-256 encrypted password (encryptedPasswordLSH)

//save to DB
savePasswordsToDB(passwordPri, encryptedPasswordLSH);
```

## 5. update new password
---

simulating when a user tries updating password.

```javascript
JavaScript

const crypto = require('crypto');
const bcrypt = require('bcrypt');
const dotenv = require('dotenv');

//import secret keys from .env file
const key = process.env.AES_KEY;
const iv = process.env.IV_KEY;

//user's new password 
const newPassword = newpassword123;

//fetch your two password hashes from DB
const { passwordPri, encryptedPasswordLSH } = await fetchPasswordsFromDB();

//Check if new password is same as passwordPri using bcrypt
const isSame = bcrypt.compare(newPassword, passwordPri);

//don't allow password change if passwords match
if(isSame) return { message:"You can't use your old password as the new one", statusCode: 401 }







```

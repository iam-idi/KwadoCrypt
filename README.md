# KwadoCrypt
Hash passwords and compare for similarities

---

All what you are about to read would have been unnecessary but users do forget their passwords, if not, we can just collect old passwords and compare with new one but like i said users do forget their passwords, and that has brought us here. 🥵

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

fast-levenshtein (for similarity comparison)

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

//Encrypt LSH hash, pass your LSH hashed password (passwordLSH), secret key and iv as arguments to the encryption function.
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

When a user tries updating password.

```javascript
JavaScript

const crypto = require('crypto');
const bcrypt = require('bcrypt');
const simHash = require('simHash');
const dotenv = require('dotenv');

//import secret keys from .env file
const key = process.env.AES_KEY;
const iv = process.env.IV_KEY;

//user's new password 
const newPassword = newpassword123;

//fetch your two passwords, bcrypt hashed and, LSH hashed and AES encrypted from DB
const { passwordPri, encryptedPasswordLSH } = await fetchPasswordsFromDB();

//Check if new password is same as passwordPri using bcrypt
const isSame = bcrypt.compare(newPassword, passwordPri);

//don't allow password change if passwords match
if(isSame) return { message:"You can't use your old password as the new one", statusCode: 401 }

//below code will only run if passwords don't match

//Decryption function
function decryptLSH(encryptedPassword, secretKey, secretIV) {
    const decipher = crypto.createDecipheriv('aes-256-cbc', secretKey, Buffer.from(secretIV, 'hex'));
    let decrypted = decipher.update(encryptedPassword, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    return decrypted;
}

//Decrypt LSH hash for similarity comparison. Pass the encryptedPasswordLSH, key and IV as arguments to the decryption function.
const decryptedLSHPassword = decryptLSH(encryptedPasswordLSH, key, iv);

//decryptedLSHPassword Should match original LSH hash

//To compare new password with our decryptedLSHPassword which is now a LSH Hash, we have to LSH hash newPassword
const newPasswordLSH = simHash(newPassword);
```
#### code explanation
* Import packages
* get env secret key and iv
* collect user's new password
* get user's two hashed passwords from db
* compare user's new password with current password (use the bcrypt hashed password) check with bcrypt.compare()
* Don't allow password change if new password matches current password (passwordPri)(exact match). Return a not allowed message.
* if password don't match, decrypt the encrypted password gotten from db (encryptedPasswordLSH)
* LSH hash new password (newPassword) to be able to compare it with decryptedLSHPassword

---

## 6. comparing decrypted hashed password(decryptedLSHPassword) with new password LSH hash(newPasswordLSH) for similarities.

To achieve this we have to compare the two hashes with a distance(similarities) calculating algorithm.

We will work with Levenshtein distance algo.

The Levenshtein distance measures the number of single-character edits (insertions, deletions, or substitutions) required to transform one string into another.

```javascript
JavaScript
const levenshtein = require('fast-levenshtein');
const dotenv = require('dotenv');

//import secret keys from .env file
const key = process.env.AES_KEY;
const iv = process.env.IV_KEY;

//set a threshold number. This is the number of single character edits we want to check for.
const threshold = 3;

//similarities check function
function areSimilar(newPasswordLSH, oldPasswordLSH, threshold) {
    const distance = levenshtein.get(newPasswordLSH, oldPasswordLSH);
    return distance <= threshold; //If distance is within the threshold, it is too similar
}

//call function and pass newPasswordLSH and decryptedLSHPassword as arguments.

const similar = areSimilar(newPasswordLSH, decryptedLSHPassword);

//if similar=true then passwords have similarities within the set threshold, in our case less than or equal to 3, and if similar=false then passwords have similarities that are above the set threshold or they have no similarities

//You can handle the outcome as you wish. Look at the example below.

if(similar){
return { message: 'Your new password can't be similar to your current password', statusCode: 401 }
} else {
//we will first encrypt the new password. You remember our encryption function right?
const encryptedNewPasswordLSH = encryptLSH(newPasswordLSH, key, iv);

//update password in DB
await updatePasswordInDB(encryptedNewPasswordLSH);

return { message: 'Password updated successfully', statusCode: 201 }
```
Finally, this should be it, we have successfully saved our hashed password securely and we are able to compare for similarities, although we are saving two passwords in the DB, we shouldn't worry because one is bcrypt hashed and the second is LSH hashed and AES-256 encrypted. 🙌🏽


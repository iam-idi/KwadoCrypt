# KwadoCrypt
Hash passwords and compare for similarities

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

## 1. hash and save password in DB
---

```javascript
const bcrypt = require('bcrypt');
const simHash = require('simHash');

const password = password123;

//gen salt
const salt = bcrypt.salt(10);

//hash password with bcrypt
const passwordPri = bcrypt.hash(password, salt);

//hash password withLSH & bcrypt
let passwordLSH = simHash(password);

passwordLSH = bcrypt.hash(password LSH, salt);

//save two of the passwords in your database
saveToDB(passswordPri, passwordLSH);
```
#### code explanation
* import packages
* have your password ready (password123)
* generate salt using bcrypt
* hash password with bcrypt (passwordPri)
* hash password with simHash then with bcrypt, yes hash twice, in the stated order (passwordLSH)
* save the two passwords (passwordPri and passwordLSH) in your database.

### 2. update new password
---

```javascript
const newPassword = newpassword123
```

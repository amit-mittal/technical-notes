# Best Pracice to Hash Password and store in database?

## Approach
Use hashing instead of encryption to store the passwords as hashing is one-way while encryption is not.

Hashing should definitely be done on the backend as frontend hashing can anyway be skipped by the user. And also, in case the database is compromised and the hacker has the username-passwords, he'll have to decipher the password (through brute force or rainbow tables) and then use it from frontend so, that the user can be compromised. 

Additionally, you can do hashing on front end as well but, the only reason I can think of it is that even by chance, we log the "hashed" passwords in the log files, it is safer than logging the plain text. But, from the user perspective, both are equally bad and he's compromised. Also, doing hashing on client side means, this should be done on all the clients i.e. mobile apps, websites, desktop apps, etc.

And to make sure that no one sniffs the password between front end and backend, use HTTPS which will protect the transportation of the data by encrypting the packets. Further improvements can be done in HTTPS protocol by using HSTS.

What HTTPS doesn't protect against is if a user is first time accessing the website, which happens mostly through the HTTP link, the hacker can intercept the traffic in between and serve the clone of the website which may or may not have HTTPS but, it's NOT the original website. To prevent this, HSTS can be used where server sends back the header (expiry time and other properties) back to the client for the very first time. After that browser is smart to make even the first connection with HTTPS with that site.

## What algorithm to use for hashing?
Some of the well-known ones are:
1. MD5, SHA-1: Should be avoided in any new as these have been compromised i.e. similar strings having same hash can be found with less effort
2. SHA-256
3. BCrypt, PBKDF2, SCrypt: These hashes are designed to be slow as well. These hashes execute an internal hash function many times in a loop. When the faster computer comes in the market, the number of rounds to re-encrypt can be increased.

Hashes are all of the same length so, we are not leaking the length of original password.

Some of the approaches using any of the above algorithm:

#### Hash the Password
One problem with simply hashing the password is that the hacker can precompute the list of commonly used passwords and match against the existing users and then, they become vulnerable.

#### Fixed Salt + Hash
Salt is long random string of bytes. If hacker gains access to the new password hashes but, not the salt, it will make it difficult for hacker to guess the passwords. Drawback is if hacker gained access to your database, probably he'll also be able to get your code as well an figure out the salt.

#### Different Salt + Hash
Put salt (random long enough string) in front of the password and hash the combined text string. It can be stored in password database along with the username and its main purpose is even if 2 users have same passwords, their hash will be different.

Having a per user salt is beneficial because now the hacker can't attack all of your user's passwords at the same time. 

You can store the salt along with password as well instead of a separate column. As salt size is fixed so, it's easier to retrieve back the salt and as it is obfuscating the password so, it's little helpful as well.

## Migration story from old password algorithm to new one
When user with old-style hashes log in successfully, regenerate and update their hashes using the new algorithm. As successful login is the only time, you can tell what a user's actual password is.

For users who haven't logged in for some time and whose hashes are still old hashes, account can be disabled and users can be forced to go through a password reset procedure when they login again.

## References
1. MD5 Hash: http://www.skrenta.com/2007/08/md5_tutorial.html
2. HSTS: https://blog.stackpath.com/glossary/hsts/
3. https://nakedsecurity.sophos.com/2013/11/20/serious-security-how-to-store-your-users-passwords-safely/

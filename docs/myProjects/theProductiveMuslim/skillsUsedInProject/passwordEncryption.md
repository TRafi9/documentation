# Learning how to store passwords

I used a hashing algorithm called bcrypt, which hashed user passwords. I then stored those hashes in the database, instead of storing passwords in plaintext.

By doing so, when the user logged in, I could hash the password they were using and compare it to the hashed password stored in the database.
If the passwords were the same, I knew the password was correct, so could continue with the authentication process.

Here are the go functions used:

### Hash password

```go
func hashPassword(password string) (string, error) {
	bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
	return string(bytes), err
}
```

### Check password hash

```go
func checkPasswordHash(password string, hash string) bool {
	err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
	return err == nil
}
```

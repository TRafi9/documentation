# My Login system

One of the key things when creating the web app was designing a login system. I used a combination of CRUD calls to my DB in order to achieve this, here's how it worked.

## 1. User Registering

When a user initially registers, it saves information about the user in a database and sends a verification code to the users email address. Here's what the database stores

- User's username (email) (primary key of DB in `users` table)
- hashed version of their password (bcrypt hashing)
- verification flag (checks to see if user has verified email, initially set to false when user registers)
- verification code expiry time

The code below shows how the initial registration is setup:

```go
var incomingUserRegistration UserCredentials
	if err := c.Bind(&incomingUserRegistration); err != nil {
		return c.JSON(http.StatusBadRequest, map[string]string{"error": "Invalid request body for new user registration"})
	}
	hashed_password, err := hashPassword(incomingUserRegistration.UserPassword)
	if err != nil {
		logger.Errorf(fmt.Sprintf("Failed to hash password, err: %s", err.Error()))
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Failed to hash password"})
	}

	incomingUserRegistration.UserPassword = hashed_password

	// get current timestamp in postgres format to register when new user was created, can use to delete values
	currentTime := currentTimeStampPostgres()
	// set email verification for users initially to false when user is created, they need to confirm later
	verifiedEmail := false

	insertSQL := `
    INSERT INTO users (
        user_id, password_hash, creation_timestamp, verified_email
    ) VALUES (
        $1, $2, $3, $4
    )
`

	_, err = db.Exec(insertSQL, incomingUserRegistration.UserEmail, incomingUserRegistration.UserPassword, currentTime, verifiedEmail)
	if err != nil {
		logger.Errorf("Failed to execute database sql statement, err: %w", err)
		return c.JSON(http.StatusAlreadyReported, map[string]string{"error": "Failed to upload user data to server, is the email already in use?"})
	}

	// add send email verification function here before returning registered user?
	// generate random passphrase for email verification confirmation
	verificationCode, err := generateRandomCode(6)
	if err != nil {
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Could not generate random code"})
	}

	expiryTime := currentTimePlusHourPostgres()

	insertVerificationCodeSQL := `
	INSERT INTO email_verification_check (
		user_id, email_verification_code, expiry_time
	) VALUES (
		$1, $2, $3
	);
	`
	_, err = db.Exec(insertVerificationCodeSQL, incomingUserRegistration.UserEmail, verificationCode, expiryTime)
	if err != nil {
		logger.Errorf("Failed to insert email verification code in DB, err %w", err)
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Failed to upload email verification code to db"})
	}

	err = sendEmailVerification(c, verificationCode, incomingUserRegistration.UserEmail, logger)
	if err != nil {
		logger.Errorf("Error sending email, err: %s", err.Error())
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Send email verification failed"})
	}
	logger.Info("email should have sent to user")
```

## 2. Verify User

### Sending the user a verification code

Once the user successfully completes the initial registration step, the page then changes to a verify user page. The user must enter the verification code sent to them within the expiry time frame for the token (1 hour), below is the code snippet that does this.

```go

func handleCreateUser(c echo.Context, logger *zap.SugaredLogger, db *sql.DB) error {
	// this function parses the incoming user data, calls an encryption on the password then uploads it to the db
	var incomingUserRegistration UserCredentials
	if err := c.Bind(&incomingUserRegistration); err != nil {
		return c.JSON(http.StatusBadRequest, map[string]string{"error": "Invalid request body for new user registration"})
	}
	hashed_password, err := hashPassword(incomingUserRegistration.UserPassword)
	if err != nil {
		logger.Errorf(fmt.Sprintf("Failed to hash password, err: %s", err.Error()))
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Failed to hash password"})
	}

	incomingUserRegistration.UserPassword = hashed_password

	// get current timestamp in postgres format to register when new user was created, can use to delete values
	currentTime := currentTimeStampPostgres()
	// set email verification for users initially to false when user is created, they need to confirm later
	verifiedEmail := false

	insertSQL := `
    INSERT INTO users (
        user_id, password_hash, creation_timestamp, verified_email
    ) VALUES (
        $1, $2, $3, $4
    )
`

	_, err = db.Exec(insertSQL, incomingUserRegistration.UserEmail, incomingUserRegistration.UserPassword, currentTime, verifiedEmail)
	if err != nil {
		logger.Errorf("Failed to execute database sql statement, err: %w", err)
		return c.JSON(http.StatusAlreadyReported, map[string]string{"error": "Failed to upload user data to server, is the email already in use?"})
	}

	// generate random passphrase for email verification confirmation
	verificationCode, err := generateRandomCode(6)
	if err != nil {
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Could not generate random code"})
	}

	expiryTime := currentTimePlusHourPostgres()

	insertVerificationCodeSQL := `
	INSERT INTO email_verification_check (
		user_id, email_verification_code, expiry_time
	) VALUES (
		$1, $2, $3
	);
	`
	_, err = db.Exec(insertVerificationCodeSQL, incomingUserRegistration.UserEmail, verificationCode, expiryTime)
	if err != nil {
		logger.Errorf("Failed to insert email verification code in DB, err %w", err)
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Failed to upload email verification code to db"})
	}

	err = sendEmailVerification(c, verificationCode, incomingUserRegistration.UserEmail, logger)
	if err != nil {
		logger.Errorf("Error sending email, err: %s", err.Error())
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Send email verification failed"})
	}


	return c.JSON(http.StatusOK, map[string]string{"error": ""})
}
```

I also added in the option to resend a verification code, where the calls to the DB were to delete all existing verification codes for the user, and then doing the same logic of writing a new verification code to the db and sending the user the new one.

### User submits the verification code

The user can then submit the verification code along with the email they signed up with, these values are checked against the DB and if they are valid, the `email_verification_check` flag is then updated to true.

**Note: I did realise later that checking if the expiry time is valid in the SQL would have made my code a lot cleaner...**

```go
CheckRecordExistsQuery := `
SELECT user_id, email_verification_code, expiry_time
FROM email_verification_check
WHERE user_id = $1
AND email_verification_code = $2
`
rows, err := db.Query(CheckRecordExistsQuery, EmailVerificationDetailsFromFrontend.UserEmail, verificationCodeFromFrontend)
if err != nil {
	logger.Error("Error in quering db for verification Email information")
	logger.Error(err)
	return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Failed to run query for email verification on DB"})
}

defer rows.Close()

var countReturnedRows int
var EmailVerificationDBResults EmailVerificationDBResults

for rows.Next() {
	err := rows.Scan(&EmailVerificationDBResults.UserEmail, &EmailVerificationDBResults.VerificationCode, &EmailVerificationDBResults.ExpiryTime)
	if err != nil {
		logger.Errorf("Error in rows.Scan for parsing rows into EmailVerificationDBResults")
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Error in rows.Scan for parsing rows into EmailVerificationDBResults"})
	}

	countReturnedRows++

}

if countReturnedRows > 1 {
	return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Rows returned more than 1 from DB for email verification check should only be one"})
}

expiryTimeValid := time.Now().Before(EmailVerificationDBResults.ExpiryTime)

// if verificationcode from db is 0 then it is because there is no result so it is a default value, so check to see if not 0
if EmailVerificationDBResults.VerificationCode == verificationCodeFromFrontend && (EmailVerificationDBResults.VerificationCode != 0) && expiryTimeValid {
	// update verification flag in user database

	updateVerificationFlag := `
	UPDATE users
	SET verified_email =  $1
	WHERE user_id = $2
	`

	_, err := db.Exec(updateVerificationFlag, true, EmailVerificationDBResults.UserEmail)

	if err != nil {
		logger.Error("Failed to set email verification flag to true in DB")
		return c.JSON(http.StatusInternalServerError, map[string]string{"error": "Failed to set email verification flag to true in DB"})
	}
	return c.JSON(http.StatusOK, map[string]string{"error": ""})
}

```

## 3. Logging in

When the user logs in the db checks for 2 things:

1. If the user has verified their email by checking boolean value of `verified_email` in the `users` table.
2. If the hashed password from the user matches the hashed password stored in the DB (no passwords are stored in plaintext)

If both are correct, the user is allowed to login, and a cookie is set in the browser with a JWT token.
This JWT Token is used to access protected API routes that would not be accessible if the user did not have the correct [JWT token](https://trafi9.github.io/documentation/myProjects/theProductiveMuslim/skillsUsedInProject/JWT/) stored.
More about JWTs and how I implemented them can be found [here!](https://trafi9.github.io/documentation/myProjects/theProductiveMuslim/skillsUsedInProject/JWT/)

# Authentication and Authorization

## This README is about achieving the perfect authentication and authorization

Included Topics

- User Registeration
- User Login
- User Logout
- User Roles
- User Auth MW

> General Flow - Use case diagram

> Research Status code further

- [http-status-codes on NPM](https://www.npmjs.com/package/http-status-codes)
- [cookie-parser on NPM](https://www.npmjs.com/package/cookie-parser)
- [Authorization Blog Post on Medium](https://pandaquests.medium.com/advanced-techniques-for-secure-authentication-and-authorization-in-node-js-applications-446e55cd18d)

`require("crypto").randomBytes(64).toString("hex")`

### User Registeration

- we receive name, email, pw, etc from client be it front-end or postman
- we check whether those fields are valid -- VALIDATION
- we check whether the user is already registered
- IF SO, We return a bad request error
- IF NOT,
  - Hash password, suggested approach --> userSchema.pre("save", hashpw_fn)
    - using bcryptjs
  - Create User
  - Send response back to user.

### User Login

> Store Access Tokens in memory NOT local storage NOR cookies

> Refresh tokens are sent as httpOnly cookies, not accessible via JS

> Once access tokens expire, new access tokens could be obtained using refresh token

- We receive email/id/phone_num and password from the user
- we check whether those fields are valid or non-empty -- VALIDATION
- we check whether the email is registered in the db
  - IF NOT, we return unauthorized error
  - IF SO, we compare passwords using bcryptjs
    - IF they DON'T MATCH we send unauthorized error
    - IF THEY MATCH
      - create access token and refresh token
      - store refresh token in db with the user
      - `res.cookie('jwt', refreshToken, {httpOnly: true, maxAge: 24 * 60 * 60 * 1000, sameSite: "None", secure: true});`
      - `res.json({accessToken})`

### User Authentication middleware

- Intercept user request
- check headers for access token
- if not included or doesn't start with "Bearer "
  - throw unauth error
- if included, verify the token using jwt
  - if not verifiable, unauth error
  - if so,
    - attach user to req body with or without find it by id through db
    - call next
- app.use(/api/v1/jobs, authMW, jobsRouter);

### Refresh Token

- extract jwt cookie(refresh token) from request
- find user associated with refresh token
- if not found
  - return status code forbidden
- if found
  - verify the the refresh token and the user associated with it by id/name/email/etc
    - if invalid return forbidden
    - if valid, create new access token
      - `res.json({accessToken})`

### User logout

- If cookies don't contain jwt cookie?
  - No , all good
  - Yes, find refresh token in the db
    - if not found? all good
    - if found
      - delete refresh token from db
      - clear jwt token from the cookies
      - `res.clearCookie("jwt", {httpOnly: true, maxAge: 24 * 60 * 60 * 1000, sameSite: "None", secure: true})`

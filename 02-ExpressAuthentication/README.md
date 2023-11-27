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
- [Authorization Blog Post on Medium](https://pandaquests.medium.com/advanced-techniques-for-secure-authentication-and-authorization-in-node-js-applications-446e55cd18d)

### User Registeration

- we receive name, email, pw, etc from client be it front-end or postman
- we check whether those fields are valid -- VALIDATION
- we check whether the user is already registered
- IF NOT, We return a bad request error
- IF So,
  - Create User
  - Hash password, suggested approach --> userSchema.pre("save", hashpw_fn)
    - using bcryptjs
  - we create an access token for the user using jwt -- userSchema.methods.createToken
  - we send the response back to the user with the token included

### User Login

- We receive email/id/phone_num and password from the user
- we check whether those fields are valid or non-empty -- VALIDATION
- we check whether the email is registered in the db
  - IF NOT, we return unauthorized error
  - IF SO, we compare passwords using bcryptjs
    - IF they DON'T MATCH we send unauthorized error
    - IF THEY MATCH we send back the response with the token included -- userSchema.methods.createToken
      - access token
      - refresh token
      - cookie

### User Authentication middleware

- Intercept user request
- check headers for token
- if not included or doesn't start with "Bearer "
  - throw unauth error
- if included, verify the token using jwt
  - if not verifiable, unauth error
  - if so,
    - attach user to req body with or without find it by id through db
    - call next
- app.use(/api/v1/jobs, authMW, jobsRouter);

### User logout

- If cookies don't contain cookie?
  - No , all good
  - Yes, find refresh token in the db
    - if not found? all good
    - if found
      - delete refresh token from db
      - clear jwt token from the cookies

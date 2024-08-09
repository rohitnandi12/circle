<font size="+12">
<center>Authentication Module</center>
</font>
<font size="+1">
<center>Architecture Document</center>
</font>

<br/><br/><br/>

**Feature Description:**

**Type:** New Feature Development

This module is responsible for authentication. It handles all requests related to login, logout, and registration. More details are in the [product requirement document](toBeAdded)

*Note: Authorization is out of scope for this feature.* 

## Functional Requirements (FR)
- Registration
    - User can register by providing username, email, and password.
    - System must ensure that the email and username is unique in the system.
    - System should notify user about username availaibility.
    - System should notify user about email availaibility.
    - System (Client) must ensure that the password in send to System (Server) in an encrypted form.
    - System must persist the password in a secure way in the database.
    - System should notify user about their password strength.
    - Validation:
       - Username must be between 3 and 10 characters long.
       - Password must be between 5 and 10 characters long.
       - email must be in the standard email format.
- Login
    - User can login with their username or email and password.
    - System should manage the User sessions expiring after 30 minutes of inactivity.
    - System should allow for maximum of 2 concurrent sessions per user.
    - On successful login, system should redirect user to the home page.
    - On unsuccessful login, system should notify user about the error.
    - On successful login, system should return user a JWT token
    - System should allow only 3 attempts for a given user in 24 hours duration.
- Logout
    - User can logout any or all of its session.
    - On successful logout, system should redirect user to the login page.
- Username reset:
    - User can reset their username from the profile page.
- Password reset:
    - User can reset their password.
    - System should send an email to the user containing a link to reset their password.
    - User can reset their password by providing new password and confirming
- Miscellaneous:
    - System should maintain user login and logout history.
    - System should record if the user has self logged out or the session expired due to inactivity.

## Back of the envelope estimation

#### Storage Requirements:

**User Credentials**
- id(8 bytes)
- Username(10 bytes)
- Email(50 bytes)
- Password(10 character)(after bCrypt 80 bytes)
- Auditing information such as createdOn, createdBy, lastModifiedOn, lastModifiedBy
(116 bytes)
- Additional Considerations: Index Size, Metadata, padding, and internal database structures, Database Engine Overhead. Consider 30% of total fixed size
- Total bytes = 264 + 30% of 264 = 343 bytes
- For 1 Billion records = (340 * 1 Billion) / (1024 * 1024 * 1024) ~= 320 GB
- if Redundancy and Replication is considered then. Total size required = replication factor * 320 GB

**User Session [TODO]**
- id
- user_id (8 bytes)
- ipAddress (24 bytes)
- createdOn (8 bytes)


**User Authentication Log [TODO]**




## Non-Functional Requirements (NFR)
- System should support 1 billion plus users
- System should support 50,000 requests/second
- System should support storing and managing 400 GB of data
- System should be highly available.


#### <u> Backend </U>

## Architecture Descision 
- Use spring-security for authentication.
- Use spring-data-jpa for persistence.
- Use spring-boot for bootstrapping.
- Mysql for database.
- Snowflake Id Generator will be used for (ADR link)
- Bloomfilter will be used to check username uniqueness (ADR link)
- System must use [bcrypt](https://www.npmjs.com/package/bcrypt) to hash passwords
- System must generate unique tokens using [TODO]??? 
- System can use CQRS architecture.??? because it is read heavy application.

Backend is developed using java spring-boot, spring-security, and spring framework


#### API Design

The module exposes REST APIs to communicate with the system.

API Contracts:
- [Authentication Module](https://github.com/joshua-miller/auth-module/blob/master/docs/api_contracts.md)

**POST /auth/register**

Request:

- HTTP version: HTTP/1.1 (Default)
- HTTP Method: POST
- Request target: /auth/register
- Headers:
  - Host: [e.g localhost:4221]
  - Content-Type: application/json
  - User-Agent: [e.g curl, postmand, client]
  - Accept: [e.g application/json or */*]
- Request body: JSON
  - username: [string]
  - email: [string]
  - password: [string]

Response: Succcess

- HTTP version: HTTP/1.1 (Default)
- Status Code: [201]
- Headers:
  - Content-Type: application/json
  - Accept: [e.g application/json or */*]
- Response body: JSON
  - messageId: ""
  - message: [string]
  - success: true
  - timestamp: [TIMESTAMP]

Response: Invalid request body violating constraints

- HTTP version: HTTP/1.1 (Default)
- Status Code: [400]
- Headers:
  - Content-Type: application/json
  - Accept: [e.g application/json or */*]
- Response body: JSON
  - messageId: ""
  - message: ""
  - success: false
  - timestamp: [TIMESTAMP]

Response: Invalid request body violating unique username and email

- HTTP version: HTTP/1.1 (Default)
- Status Code: [409]
- Headers:
  - Content-Type: application/json
  - Accept: [e.g application/json or */*]
- Response body: JSON
  - messageId: "DUPLICATE_USERNAME_OR_EMAIL"
  - message: "Username already exists" or "Email already exists"
  - success: false
  - timestamp: [TIMESTAMP]

**POST /auth/login**

Request:

- HTTP version: HTTP/1.1 (Default)
- HTTP Method: POST
- Request target: /auth/login
- Headers:
  - Host: [e.g localhost:4221]
  - Content-Type: application/json
  - User-Agent: [e.g curl, postmand, client]
  - Accept: [e.g application/json or */*]
- Request body: JSON
  - username: [string]
  - email: [string]
  - password: [string]

Response: Successful login

- HTTP version: HTTP/1.1 (Default)
- Status Code: [200]
- Headers:
  - Content-Type: application/json
  - Accept: [e.g application/json or */*]
- Response body: JSON
  - messageId: "LOGIN_SUCCESSFUL"
  - message: "Welcome to the portal"
  - token: eyJhbGciOiJIUzIXVCJ9.eyJzdWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMssw5c
  - success: false
  - timestamp: [TIMESTAMP]

Response: Invalid request body with invalid username or email

- HTTP version: HTTP/1.1 (Default)
- Status Code: [400]
- Headers:
  - Content-Type: application/json
  - Accept: [e.g application/json or */*]
- Response body: JSON
  - messageId: "DUPLICATE_USERNAME_OR_EMAIL"
  - message: "Username doesn't exist in the system." or "Email doesn't exist in the system."
  - success: false
  - timestamp: [TIMESTAMP]

Response: Invalid request body with invalid password

- HTTP version: HTTP/1.1 (Default)
- Status Code: [401]
- Headers:
  - Content-Type: application/json
  - Accept: [e.g application/json or */*]
- Response body: JSON
  - messageId: "DUPLICATE_USERNAME_OR_EMAIL"
  - message: "Invalid password for this username."
  - success: false
  - timestamp: [TIMESTAMP]

#### Schema Design

The module uses a relational database to persist data.

**Table : user_credentials**

Columns:
- id (8 bytes, snowflakeIdGenerator)
- username (varchar(10), max-length = 10, min-length = 5, allowed-chars = [a-z A-Z 0-9 ] )
- email (varchar(50), standard email format)
- password (varchar(100), hashed using bcrypt)
- salt (varchar(16), generated using uuidv4)
- created_on (date/time, default = current date and time)
- created_by (varchar(10), default = 'SYSTEM') [DOUBT: Who else could be? Do we really need this??]
- last_modified_on (date/time, default = current date and time)
- last_modified_by (varchar(10), default = 'SYSTEM')


**Table : user_session**

Columns:
- id (8 bytes, uuidv4?????)
- token (varchar(36))
- created_on  (date/time)



# Frontend

### Responsibilities
- Registration form
- login form
- logout button
- Client must check for password strength using [zxcvbn](https://github.com/dropbox/zxcvbn)
- Client must validate the inputs constraints.


### Mockup Design
- [Figma](https://www.figma.com/)


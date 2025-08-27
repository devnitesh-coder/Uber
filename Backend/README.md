# Users — POST /users/register

Create a new user account and receive an authentication token.

## Endpoint
- URL: `/users/register`
- Method: `POST`
- Content-Type: `application/json`

## Request headers
- `Content-Type: application/json`

## Request body
JSON object with the following shape:

- `fullname` (object) — required
  - `firstname` (string) — required, minimum 3 characters
  - `lastname` (string) — optional, minimum 3 characters when provided
- `email` (string) — required, must be a valid email
- `password` (string) — required, minimum 6 characters

Example:
```json
{
  "fullname": {
    "firstname": "Alice",
    "lastname": "Smith"
  },
  "email": "alice@example.com",
  "password": "supersecret"
}
```

## Validation rules
- `fullname.firstname` - must be at least 3 characters
- `email` - must be a valid email address
- `password` - must be at least 6 characters

If validation fails, the endpoint responds with HTTP 400 and an `errors` array (express-validator format).

## Success response
- Status: `201 Created`
- Body:
```json
{
  "token": "<jwt-token>",
  "user": {
    "_id": "<user-id>",
    "fullname": {
      "firstname": "Alice",
      "lastname": "Smith"
    },
    "email": "alice@example.com",
    "socketID": null,
    "__v": 0
  }
}
```
Notes:
- The `token` is a JWT (expires in 1 hour).
- The returned `user` does not include the password field (password is not selected).

## Common error responses
- `400 Bad Request` — validation failed. Body example:
```json
{ "errors": [ { "msg": "Password must be at least 6 characters long", "param": "password", ... } ] }
```
- `409 Conflict` — email already exists (if unique constraint is violated). Implementation-dependent error shape.
- `500 Internal Server Error` — unexpected server error.

## Example curl
```bash
curl -X POST http://localhost:3000/users/register \
  -H "Content-Type: application/json" \
  -d '{"fullname":{"firstname":"Alice","lastname":"Smith"},"email":"alice@example.com","password":"supersecret"}'
```

# Login — POST /users/login

Authenticate an existing user and receive an authentication token.

- URL: `/users/login`
- Method: `POST`
- Content-Type: `application/json`

Request headers:
- `Content-Type: application/json`

Request body (JSON):
- `email` (string) — required, must be a valid email
- `password` (string) — required

Example:
```json
{
  "email": "alice@example.com",
  "password": "supersecret"
}
```

Validation rules:
- `email` — must be a valid email address
- `password` — required (not empty)

Success response:
- Status: `200 OK`
- Body:
```json
{
  "token": "<jwt-token>",
  "user": {
    "_id": "<user-id>",
    "fullname": {
      "firstname": "Alice",
      "lastname": "Smith"
    },
    "email": "alice@example.com",
    "socketID": null,
    "__v": 0
  }
}
```
Notes:
- The `token` is a JWT (expires in 1 hour).
- The returned `user` should not include the password field. (If the implementation selects the password to verify credentials, ensure it is removed before sending the response.)

Common error responses:
- `400 Bad Request` — validation failed. Example:
```json
{ "errors": [ { "msg": "Invalid email address", "param": "email", ... } ] }
```
- `401 Unauthorized` — invalid email or password. Example:
```json
{ "message": "Invalid email or password" }
```
- `500 Internal Server Error` — unexpected server error.

Example curl:
```bash
curl -X POST http://localhost:3000/users/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"supersecret"}'
```
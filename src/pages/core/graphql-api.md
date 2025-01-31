---
title: GraphQL API
layout: ../../layouts/Main.astro
---

Authorizer instance supports GraphQL natively and thus helps you share the common schema for your frontend applications.

You can play with GraphQL API using the GraphQL playground that comes with your Authorizer instance. Access GraphQL playground on the instance same as of your Authorizer instance URL.

> Note super admin only queries / mutations starts with underscore `_`.

Table of Contents

- [Queries](#queries)
  - [`meta`](#meta)
  - [`session`](#session)
  - [`is_valid_jwt`](#is_valid_jwt)
  - [`profile`](#profile)
  - [`_users`](#_users)
  - [`_verification_requests`](#_verification_requests)
  - [`_admin_session`](#_admin_session)
  - [`_env`](#_env)
- [Mutations](#mutations)
  - [`signup`](#signup)
  - [`login`](#login)
  - [`magic_link_login`](#magic_link_login)
  - [`logout`](#logout)
  - [`update_profile`](#update_profile)
  - [`verify_email`](#verify_email)
  - [`resend_verify_email`](#resend_verify_email)
  - [`forgot_password`](#forgot_password)
  - [`reset_password`](#reset_password)
  - [`_admin_signup`](#_admin_signup)
  - [`_admin_login`](#_admin_login)
  - [`_admin_logout`](#_admin_logout)
  - [`_update_env`](#_update_env)
  - [`_update_user`](#_update_user)
  - [`_delete_user`](#_delete_user)

## Queries

### `meta`

Query to get the `meta` information about your authorizer instance. eg, version, configurations, etc
It returns `Meta` type with the following possible values

| Key                               | Description                                                   |
| --------------------------------- | ------------------------------------------------------------- |
| `version`                         | Authorizer version that is currently deployed                 |
| `is_google_login_enabled`         | It gives information if google login is configured or not     |
| `is_github_login_enabled`         | It gives information if github login is configured or not     |
| `is_facebook_login_enabled`       | It gives information if facebook login is configured or not   |
| `is_email_verification_enabled`   | It gives information if email verification is enabled or not  |
| `is_basic_authentication_enabled` | It gives information, if basic auth is enabled or not         |
| `is_magic_link_login_enabled`     | It gives information if password less login is enabled or not |

**Sample Query**

```graphql
query {
  meta {
    version
    is_google_login_enabled
    is_github_login_enabled
    is_facebook_login_enabled
    is_email_verification_enabled
    is_basic_authentication_enabled
    is_magic_link_login_enabled
  }
}
```

### `session`

Query to get the `session` information.

> Note: Session information should be present as present as HTTP Cookie. If the information is not present or an invalid data is present it throws `unauthorized` error

This query can take a optional input `params` of type `SessionQueryInput` which includes `roles` to verify if the current token is valid for a given roles.

**Request Params**

| Key     | Description                      | Required |
| ------- | -------------------------------- | -------- |
| `roles` | Array of string with valid roles | false    |

It returns `AuthResponse` type with the following keys.

**Response**

| Key            | Description                                                                                            |
| -------------- | ------------------------------------------------------------------------------------------------------ |
| `message`      | Error / Success message from server                                                                    |
| `access_token` | accessToken that frontend application can use for further authorized requests                          |
| `expires_at`   | timestamp when the current token is going to expire, so that frontend can request for new access token |
| `user`         | User object with all the basic profile information                                                     |

**Sample Query**

```graphql
query {
  session(params: { roles: ["admin"] }) {
    message
    accessToken
    expires_at
    user {
      id
      email
      roles
    }
  }
}
```

### `is_valid_jwt`

Query to get the `jwt` / access token information.

This query can take a optional input `params` of type `IsValidJWTQueryInput` which includes `token` & `roles` to verify if the current token is valid for a given roles.
If the token is not passed via params query will try to access it via Authorization Header / HTTP Cookie and if not present there then it will return `unauthorized` error.

**Request Params**

| Key     | Description                      | Required |
| ------- | -------------------------------- | -------- |
| `token` | JWT token string                 | false    |
| `roles` | Array of string with valid roles | false    |

It returns `ValidJWTResponse` type with the following keys.

**Response**

| Key       | Description                                                |
| --------- | ---------------------------------------------------------- |
| `message` | Error / Success message from server                        |
| `valid`   | Boolean value stating if the given jwt is valid or invalid |

**Sample Query**

```graphql
query {
  session(params: { roles: ["admin"] }) {
    message
    accessToken
    expires_at
    user {
      id
      email
      roles
    }
  }
}
```

### `profile`

Query to get the `profile` information of a user. It returns `User` type with the following keys.

> Note: this is authorized route, so HTTP Cookie / Authorization Header with bearer token must be present.

| Key                     | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| `id`                    | user unique identifier                                       |
| `email`                 | email address of user                                        |
| `given_name`            | first name of user                                           |
| `family_name`           | last name of user                                            |
| `signup_methods`        | methods using which user have signed up, eg: `google,github` |
| `email_verified`        | timestamp at which the email address was verified            |
| `picture`               | profile picture URL                                          |
| `roles`                 | List of roles assigned to user                               |
| `middle_name`           | middle name of user                                          |
| `nickname`              | nick name of user                                            |
| `preferred_username`    | preferred username (defaults to email currently)             |
| `gender`                | gender of user                                               |
| `birthdate`             | birthdate of user                                            |
| `phone_number`          | phone number of user                                         |
| `phone_number_verified` | if phone number is verified                                  |
| `created_at`            | timestamp at which the user entry was created                |
| `updated_at`            | timestamp at which the user entry was updated                |

**Sample Query**

```graphql
query {
  profile {
    given_name
    family_name
    email
    picture
    roles
  }
}
```

### `_users`

Query to get all the `_users`. This query is only allowed for super admins. It returns array of users `Users` with below mentioned keys.

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer.admin` as http only cookie.

```json
{
  "x-authorizer-admin-secret": "ADMIN_SECRET"
}
```

It can take optional `params` input of type `PaginatedInput` with following keys

**Request Params**

| Key     | Description                  | Required | Default |
| ------- | ---------------------------- | -------- | ------- |
| `page`  | Number of page that you want | false    | 1       |
| `limit` | Number of rows that you want | false    | 10      |

**Sample Query**

```graphql
query {
  _users(params: {
    pagination: {
      page: 2
      limit: 10
    }
  }) {
    pagination: {
      offset
      total
      page
      limit
    }
    users {
      id
      given_name
      family_name
      email
      picture
      roles
    }
  }
}
```

### `_verification_requests`

Query to get all the `_verification_requests`. This query is only allowed for super admins. It returns array of verification requests `[VerificationRequest!]!` with following keys.

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

```json
{
  "x-authorizer-admin-secret": "ADMIN_SECRET"
}
```

It can take optional `params` input of type `PaginatedInput` with following keys

**Request Params**

| Key     | Description                  | Required | Default |
| ------- | ---------------------------- | -------- | ------- |
| `page`  | Number of page that you want | false    | 1       |
| `limit` | Number of rows that you want | false    | 10      |

**Sample Query**

```graphql
query {
  _verification_requests(params: { pagination: { limit: 10, page: 2 } }) {
    pagination {
      limit
      offset
      page
    }
    verification_requests {
      id
      token
      email
      expires
      identifier
    }
  }
}
```

### `_admin_session`

Query to get admin session for dashboard

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

```json
{
  "x-authorizer-admin-secret": "ADMIN_SECRET"
}
```

| Key       | Description                          |
| --------- | ------------------------------------ |
| `message` | Success response message from server |

**Sample Query**

```graphql
query {
  _admin_session {
    message
  }
}
```

### `_env`

Query to get all the [environment variables](/core/env).

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

```json
{
  "x-authorizer-admin-secret": "ADMIN_SECRET"
}
```

All the [environment variables](/core/env) values can be obtained using this this.

**Sample Query**

```graphql
query {
  _env {
    DATABASE_TYPE
    DATABASE_URL
    DATABASE_NAME
    ...
  }
}
```

## Mutations

### `signup`

A mutation to signup users using email and password. It accepts `params` of type `SignUpInput` with following keys as parameter

**Request Params**

| Key                | Description                                                                             | Required |
| ------------------ | --------------------------------------------------------------------------------------- | -------- |
| `email`            | Email address of user                                                                   | true     |
| `password`         | Password that user wants to set                                                         | true     |
| `confirm_password` | Value same as password to make sure that its user and not robot                         | true     |
| `given_name`       | First name of the user                                                                  | false    |
| `family_name`      | Last name of the user                                                                   | false    |
| `picture`          | Profile picture URL                                                                     | false    |
| `roles`            | List of roles to be assigned. If not specified `DEFAULT_ROLE` value of env will be used | false    |
| `middle_name`      | middle name of user                                                                     | false    |
| `nickname`         | nick name of user                                                                       | false    |
| `gender`           | gender of user                                                                          | false    |
| `birthdate`        | birthdate of user                                                                       | false    |
| `phone_number`     | phone number of user                                                                    | false    |

This mutation returns `AuthResponse` type with following keys

**Response**

| Key            | Description                                                                                                                                                                         |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `message`      | Success / Error message from server                                                                                                                                                 |
| `access_token` | Token that can be used for further authorized requests. This is only returned if `DISABLE_EMAIL_NOTIFICATION` is set to `true` in environment variables                             |
| `expires_at`   | Timestamp when the access Token will expire so that frontend can request new token. This is only returned if `DISABLE_EMAIL_NOTIFICATION` is set to `true` in environment variables |
| `user`         | User object with its profile keys mentioned [above](#--profile). This is only returned if `DISABLE_EMAIL_NOTIFICATION` is set to `true` in environment variables                    |

**Sample Mutation**

```graphql
mutation {
  signup(
    params: { email: "foo@bar.com", password: "test", confirm_password: "test" }
  ) {
    message
  }
}
```

### `login`

A mutation to login users using email and password. It accepts `params` of type `LoginInput` with following keys as parameter

**Request Params**

| Key        | Description                     | Required |
| ---------- | ------------------------------- | -------- |
| `email`    | Email address of user           | true     |
| `password` | Password that user wants to set | true     |
| `roles`    | Roles to login with             | false    |

This mutation returns `AuthResponse` type with following keys

**Response**

| Key            | Description                                                                         |
| -------------- | ----------------------------------------------------------------------------------- |
| `message`      | Success / Error message from server                                                 |
| `access_token` | Token that can be used for further authorized requests.                             |
| `expires_at`   | Timestamp when the access Token will expire so that frontend can request new token. |
| `user`         | User object with its profile keys mentioned [above](#--profile).                    |

**Sample Mutation**

```graphql
mutation {
  login(params: { email: "foo@bar.com", password: "test" }) {
    user {
      email
      given_name
      family_name
      picture
      roles
    }
    accessToken
    expires_at
    message
  }
}
```

### `magic_link_login`

A mutation to perform password less login. It accepts `params` of type `MagicLinkLoginInput` with following keys as parameter. When the operation is successful, it sends user email with magic link to login. This link is valid for 30 minutes only.

> Note: You will need a SMTP server with an email address and password configured as [authorizer environment](/core/env/) using which system can send emails.

**Request Params**

| Key     | Description           | Required |
| ------- | --------------------- | -------- |
| `email` | Email address of user | true     |
| `roles` | Roles to login with   | false    |

This mutation returns `Response` type with following keys

**Response**

| Key       | Description                         |
| --------- | ----------------------------------- |
| `message` | Success / Error message from server |

**Sample Mutation**

```graphql
mutation {
  magic_link_login(params: { email: "foo@bar.com" }) {
    message
  }
}
```

### `logout`

Mutation to logout user. This is authorized request and accepts `token` as HTTP cookie or Authorization header with `Bearer token`.
This action clears the session data from server. It returns `Response` type with following keys

**Response**

| Key       | Description                         |
| --------- | ----------------------------------- |
| `message` | Success / Error message from server |

**Sample Mutation**

```graphql
mutation {
  logout {
    message
  }
}
```

### `update_profile`

Mutation to update profile of user. It accepts `params` of type `UpdateProfileInput` with following keys as parameter

**Request Params**

| Key                  | Description                                                                                                                                                      | Required |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| `given_name`         | New first name of the user                                                                                                                                       | false    |
| `family_name`        | New last name of the user                                                                                                                                        | false    |
| `email`              | New email of th user. This will logout the user and send the new verification mail to user if `DISABLE_EMAIL_NOTIFICATION` is set to false                       | false    |
| `old_password`       | In case if user wants to change password they need to specify the older password here. In this scenario `newPassword` and `confirmNewPassword` will be required. | false    |
| `newPassword`        | New password that user wants to set. In this scenario `old_password` and `confirmNewPassword` will be required                                                   | false    |
| `confirmNewPassword` | Value same as the new password to make sure it matches the password entered by user. In this scenario `old_password` and `newPassword` will be required          | false    |
| `middle_name`        | New middle name of user                                                                                                                                          | false    |
| `nickname`           | New nick name of user                                                                                                                                            | false    |
| `gender`             | New gender of user                                                                                                                                               | false    |
| `birthdate`          | New birthdate of user                                                                                                                                            | false    |
| `phone_number`       | New phone number of user                                                                                                                                         | false    |

This mutation returns `Response` type with following keys

**Response**

| Key       | Description                         |
| --------- | ----------------------------------- |
| `message` | Success / Error message from server |

**Sample Mutation**

```graphql
mutation {
  update_profile(params: { given_name: "bob" }) {
    message
  }
}
```

### `verify_email`

Mutation to verify email address of user. It accepts `params` of type `VerifyEmailInput` with following keys as parameter

**Request Params**

| Key     | Description                   | Required |
| ------- | ----------------------------- | -------- |
| `token` | Token sent for verifying user | true     |

This mutation returns `AuthResponse` type with following keys

**Response**

| Key            | Description                                                                         |
| -------------- | ----------------------------------------------------------------------------------- |
| `message`      | Success / Error message from server                                                 |
| `access_token` | Token that can be used for further authorized requests.                             |
| `expires_at`   | Timestamp when the access Token will expire so that frontend can request new token. |
| `user`         | User object with its profile keys mentioned [above](#--profile).                    |

**Sample Mutation**

```graphql
mutation {
  verify_email(params: { token: "some token" }) {
    user {
      email
      given_name
      family_name
      picture
    }
    accessToken
    expires_at
    message
  }
}
```

### `resend_verify_email`

Mutation to resend verification email. This is helpful if user does not receive verification email. It accepts `params` of type `ResendVerifyEmailInput` with following keys as parameter

**Request Params**

| Key          | Description                                                                                                                    | Required |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------ | -------- |
| `email`      | Email on which the verification email is not received                                                                          | true     |
| `identifier` | Which type of verification request it is. `basic_auth_signup`, `update_email`, `forgot_password` are the supported identifiers | true     |

This mutation returns `Response` type with following keys

**Response**

| Key       | Description                         |
| --------- | ----------------------------------- |
| `message` | Success / Error message from server |

**Sample Mutation**

```graphql
mutation {
  resend_verify_email(
    params: { email: "foo@bar.com", identifier: "basic_auth_signup" }
  ) {
    message
  }
}
```

### `forgot_password`

Mutation to reset the password in case user have forgotten it. For security reasons this is 2 step process, we send email to the registered and then the are redirect to reset password url through the link in that email. In the first step, it accepts `params` of type `ForgotPasswordInput` with following keys as parameter

**Request Params**

| Key     | Description                                  | Required |
| ------- | -------------------------------------------- | -------- |
| `email` | Email for which password needs to be changed | true     |

This mutation returns `Response` type with following keys

**Response**

| Key       | Description                         |
| --------- | ----------------------------------- |
| `message` | Success / Error message from server |

**Sample Mutation**

```graphql
mutation {
  forgot_password(params: { email: "foo@bar.com" }) {
    message
  }
}
```

### `reset_password`

Mutation to reset the password. For security reasons this is 2 step process, we send email to the registered and then the are redirect to reset password url through the link in that email. In the second step, it accepts `params` of type `ResetPasswordInput` with following keys as parameter

**Request Params**

| Key                | Description                                                                 | Required |
| ------------------ | --------------------------------------------------------------------------- | -------- |
| `token`            | Token sent via email in step 1                                              | true     |
| `password`         | New password that user wants to set                                         | true     |
| `confirm_password` | Same as password just to make sure the values match and is entered by human | true     |

This mutation returns `Response` type with following keys

**Response**

| Key       | Description                         |
| --------- | ----------------------------------- |
| `message` | Success / Error message from server |

**Sample Mutation**

```graphql
mutation {
  reset_password(
    params: { token: "some token", password: "test", confirm_password: "test" }
  ) {
    message
  }
}
```

### `_admin_signup`

Mutation to signup administrator. This only works if `ADMIN_SECRET` env is not set. It accepts `params` of type `AdminSignupInput` with following keys

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

| Key            | Description                        | Required |
| -------------- | ---------------------------------- | -------- |
| `admin_secret` | Secure secret with >= 6 characters | `true`   |

This mutation returns `Response` type with message

**Sample Mutation**

```graphql
mutation {
  _admin_signup(params: { admin_secret: "some string" }) {
    message
  }
}
```

### `_admin_login`

Mutation to login administrator. It accepts `params` of type `AdminLoginInput` with following keys

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

| Key            | Description                        | Required |
| -------------- | ---------------------------------- | -------- |
| `admin_secret` | Secure secret with >= 6 characters | `true`   |

This mutation returns `Response` type with message

**Sample Mutation**

```graphql
mutation {
  _admin_login(params: { admin_secret: "some string" }) {
    message
  }
}
```

### `_admin_logout`

Mutation to logout administrator. It does not have any params

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

This mutation returns `Response` type with message

**Sample Mutation**

```graphql
mutation {
  _admin_logout {
    message
  }
}
```

### `_update_env`

Mutation to update [environment variables](/core/env). It accepts `params` of type `UpdateEnvInput` with keys present in [environment variables](/core/env)

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

This mutation returns `Response` type with message

**Sample Mutation**

```graphql
mutation {
  _update_env(params: { DATABASE_URL: "data.db", DATABASE_TYPE: "sqlite" }) {
    message
  }
}
```

### `_update_user`

Mutation to update the profile of users. This mutation is only allowed for super admins. It accepts `params` of type `UpdateUserInput` with following keys

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

```json
{
  "x-authorizer-admin-secret": "ADMIN_SECRET"
}
```

**Request Params**

| Key            | Description                       | Required |
| -------------- | --------------------------------- | -------- |
| `id`           | ID of user to be updated          | true     |
| `email`        | New email address of user         | false    |
| `given_name`   | Updated first name of user        | false    |
| `family_name`  | Updated last name of user         | false    |
| `picture`      | Updated picture url of user       | false    |
| `roles`        | Set of new roles for a given user | false    |
| `middle_name`  | New middle name of user           | false    |
| `nickname`     | New nick name of user             | false    |
| `gender`       | New gender of user                | false    |
| `birthdate`    | New birthdate of user             | false    |
| `phone_number` | New phone number of user          | false    |

This mutation returns `User` type with update values

**Sample Mutation**

```graphql
mutation {
  _update_user(
    params: { id: "20", given_name: "Bob", roles: ["user", "admin"] }
  ) {
    id
    given_name
    roles
    createdAt
    updatedAt
  }
}
```

### `_delete_user`

Mutation to delete user. This mutation is only allowed for super admins. It accepts `params` of type `DeleteUserInput` with following keys

> Note: the super admin query can be access via special header with super admin secret (this is set via ENV) or `authorizer-admin` as http only cookie.

**Request Params**

| Key     | Description                                          | Required |
| ------- | ---------------------------------------------------- | -------- |
| `email` | Email of user that needs to be removed from platform | true     |

This mutation returns `Response` type with following keys

**Response**

| Key       | Description                         |
| --------- | ----------------------------------- |
| `message` | Success / Error message from server |

**Sample Mutation**

```graphql
mutation {
  _delete_user(params: { email: "foo@bar.com" }) {
    message
  }
}
```

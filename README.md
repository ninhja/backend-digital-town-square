# Project Digital Town Square - Survivor Alliance
This is the backend for Project Digital Town Square - Survivor Alliance.

<br />

## Setup before invoking lambdas
- `yarn`
- `yarn global add serverless`
- create `.env` and fill out values:
    - COGNITO_POOL_ID
    - DB_NAME
    - DB_USER
    - DB_PASSWORD
    - DB_HOST
    - DB_PORT
- populate `secrets.json` with:
```
{
    "DB_NAME": {DB_NAME},
    "DB_USER": {DB_USER},
    "DB_PASSWORD": {DB_PASSWORD},
    "DB_HOST": {DB_HOST},
    "DB_PORT": {DB_PORT},
    "NODE_ENV": {NODE_ENV},
    "SECURITY_GROUP_ID": {SECURITY_GROUP_ID},
    "SUBNET1_ID": {SUBNET1_ID},
    "SUBNET2_ID": {SUBNET2_ID},
    "SUBNET3_ID": {SUBNET3_ID},
    "SUBNET4_ID": {SUBNET4_ID}
}
```

<br />

## Testing locally
- `serverless offline`

Running `serverless offline` will start up a local server at `localhost:3000`. You can access the relative endpoints at `http://localhost:3000/dev/{ENDPOINT_NAME}`.

<br/>

## Deploying local changes
- `serverless deploy`

<br />

## Invoking the deployed lambdas
- Can use the endpoint `https://gle1clmibc.execute-api.us-west-2.amazonaws.com/dev/{ENDPOINT_NAME}`
- Using the endpoint will require AWS signature:
    - AWS AccessKey
    - AWS SecretKey
- All endpoints require a query param: `user`
    - Use the `npm` module `cryptr` to encrypt the user email
    - Encryption key should be the cognito user pool id
    - Wrap the encrypted email around `encodeURIComponent` function
    - The returned value should be the value of the query param: `user`.

<br />

## Folder Structure

```
├── src                             # contains all endpoints
│   ├── middleware                  # contains middleware to authenticate user
│   ├── routes                      # contains all lambdas handlers
│   │    ├── comments               # lambda handlers for comments
│   │    ├── posts                  # lambda handlers for posts
│   │    ├── topics                 # lambda handlers for topics
│   │    ├── userConnections        # lambda handlers for userConnections
│   │    ├── users                  # lambda handlers for users
├── .env                            # contains all env variables
├── secrets.json                    # contains all secret variables
├── config.sql                      # contains the code to create tables and columns in db
├── package.json                    # contains dependencies and app info
├── README.md                       # contains the code to create tables and columns in db
├── serverless.yml                  # contains app configurations for deployment
└── yarn.lock                       # yarn lock file
```

<br />

## Database
- PSQL - `projectdtsdev1` database

<br />

## Important Enums
- `status` for user connection or posts
    - `"0"`: pending
    - `"1"`: accepted
    - `"2"`: rejected

- `user_role` for user:
    - `"0"`: member
    - `"1"`: admin

<br />

## Endpoints Documentation

<br />

### User Endpoints

<br />

GET
<br/>

`/user/{userId}`
> Description

Returns the user information. If no `userId` is specified, it will return the user who made the request.
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
userId: integer
        optional
```
> Response
- When user connection status is not 1 (accepted)
    ```
    200
    Content type: application/json
    {
        statusCode: 200,
        headers: {
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Credentials": true
        },
        body: {
            "id": 1,
            "image_url": null,
            "user_role": "0",
            "user_name": "testuser",
            "user_display_name": null,
            "occupation": null,
            "country": null,
            "state": null,
            "about": null,
            "goal_other": null,
            "interest_other": null,
            "created_at": "2021-06-27T21:29:16.386Z",
            "updated_at": null,
            "language": [],
            "goal": [],
            "interest": [],
            "file": [],
            "status": "0",
            "action_user_id": 1
        }
    }
    ```
- When user connection status is 1 (accepted)
    ```
    200
    Content type: application/json
    {
        statusCode: 200,
        headers: {
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Credentials": true
        },
        body: {
            "id": 1,
            "image_url": null,
            "user_role": "0",
            "user_name": "testuser",
            "user_display_name": null,
            "occupation": null,
            "country": null,
            "state": null,
            "about": null,
            "goal_other": null,
            "interest_other": null,
            "email": "testuser@example.com",
            "whatsapp": null,
            "twitter": null,
            "facebook": null,
            "linkedin": null,
            "instagram": null,
            "created_at": "2021-07-03T13:32:05.035Z",
            "updated_at": null,
            "language": [],
            "goal": [],
            "interest": [],
            "file": [],
            "contact": [],
            "status": "1",
            "action_user_id": 1
        }
    }
    ```
- On error
    ```
    400
    Content type: application/json
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

`/users`
> Description

Returns the list of all users.
- For users who have not established a connection with the user that called the api, contact details will not be returned.
- For users who are connected with the current user who made the request, contact details of the user will also be returned.
- The connection status with the current user, and who made the connection request (`action_user_id`) will also be returned.
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Response
- Success
    ```
    {
        statusCode: 200,
        body: [
            {
                "id": 1,
                "image_url": null,
                "user_role": "0",
                "user_name": "testuser",
                "user_display_name": null,
                "occupation": null,
                "country": null,
                "state": null,
                "about": null,
                "goal_other": null,
                "interest_other": null,
                "email": "testuser@example.com",
                "whatsapp": null,
                "twitter": null,
                "facebook": null,
                "linkedin": null,
                "instagram": null,
                "created_at": "2021-07-03T13:32:05.035Z",
                "updated_at": null,
                "language": [],
                "goal": [],
                "interest": [],
                "file": [],
                "contact": [],
                "status": "1",
                "action_user_id": 1
            }
        ]
    }
    ```
- On error
    ```
    400
    Content type: application/json
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```

<br />

`POST`
<br />

`/user/cognito`

> **ADMIN ONLY**

> Description

Creates a new user

> Example Request Body
```
{
    user_name: "testuser",
    user_role: "0",
    email: "testuser@example.com"
}
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "User successfully created!"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error creating user"
    }
    ```

<br />

`/user/{userId}`
> Description

Updates an existing user
> Example Request Body
```
{
    "image_url": "image",
    "user_name": "test",
    "user_display_name": "Test",
    "occupation": "student",
    "country": "Germany",
    "state": "",
    "about": "",
    "goal_other": "",
    "interest_other": "Reading",
    "whatsapp": "",
    "facebook": "",
    "linkedin": "",
    "instagram": "",
    "goal": [1, 2, 3],
    "language": [1, 2, 3],
    "interest": [1, 2, 3],
    "file": [1, 2, 3]
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
userId  integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "User updated successfully"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "User updated successfully"
    }
    ```

<br />

`/user/{userId}/contact`
> Description

Creates a user contact
> Request Body Example
```
{
    "title": "title",
    "contact_info": "contact_info"
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
userId: integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully created user contact"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

`/user/{userId}/contact/{contactId}`
> Description

Updates an existing user contact
> Request Body Example
```
{
    "title": "title",
    "contact_info": "contact_info"
}
```
> Response
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
userId: integer
        required

contactId:  integer
            required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated user contract"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

`/user/{userId}/file`
> Description

Creates a user file
> Request Body Example
```
{
    file_name: "File"
    file_source: "source",
    file_type: "jpg"
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
userId: integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully created user file"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```

<br />

`/user/{userId}/file/{fileId}`
> Description

Updates an existing user file
> Request Body Example
```
{
    file_name: "File"
    file_source: "source",
    file_type: "jpg"
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
userId: integer
        required

fileId: integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated user file"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```

<br />

### User Connection Endpoints

<br />

`POST`
<br />

`connections/{connectionId}`
> Description

Adds or updates a user connection.
- If a connection between the user who made the request and the user specified by `connectionId` does not exist, a new connection will be added.
> Request Body Example
```
{
    status: "0" // pending
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
connectionId:   integer
                required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated connection"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

### Topic Endpoints
`GET`
<br />

`/topics`
> Description

Returns topics
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: [
            {
                "id": 1,
                "title": "Topic",
                "description": "topic description",
                "post_count": 1,
                "most_recent_post_date": "2021-07-03T13:32:05.035Z",
                "created_at": "2021-07-03T13:32:05.035Z",
                "updated_at": null
            }
        ]
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

`POST`

`/topic/{topicId}`
> **ADMIN ONLY**

> Description

Adds or updates a topic
> Example Request Body
```
{
    title: "Topic Title",
    description: "Topic description"
}
```

> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
topicId:    integer
            optional
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated topic"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error updating topic"
    }
    ```
<br />

### Post Endpoints
`GET`
<br />

`/topics/{topicId}/posts`
> Description

Returns posts under a particular topic
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
topicId integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: [
            {
                "id": 1,
                "topic_id": 1,
                "user_id": 1,
                "status": "0",
                "post_title": "Post",
                "post_content": "post content",
                "comment_count": 1,
                "created_at": "2021-07-03T13:32:05.035Z",
                "updated_at": null
            }
        ]
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

`POST`
<br />

`/topics/{topicId}/post`
> Description

Creates a new post under a particular topic
> Response Body Example
```
{
    post_title: "post title",
    post_content: "post content"
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
topicId integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully created post"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

`/topics/{topicId}/post/{postId}`
> Description

Updates an existing post under a particular topic
> Response Body Example
- For admins:
    ```
    {
        post_title: "post title",
        post_content: "post content",
        status: "1"
    }
    ```
- For member:
    ```
    {
        post_title: "post title",
        post_content: "post content",
    }
    ```

> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
topicId integer
        required

postId  integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated post"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error updating post"
    }
    ```
<br />

### Comment Endpoints

`GET`
<br />

`/posts/{postId}/comments`
> Description

Returns comments under a post
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
postId  integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: [
            {
                "id": 1,
                "post_id": 1,
                "user_id": 1,
                "comment_content": "text",
                "created_at": "2021-06-27T21:29:16.386Z",
                "updated_at": null,
            }
        ]
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "{ERROR_MESSAGE}"
    }
    ```
<br />

`POST`
<br />

`/posts/{postId}/comment`
> Description

Adds a new comment under a post
> Response Body Example
```
{
    comment_content: "comment content"
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
postid  integer
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully added comment"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error creating comment"
    }
    ```
<br />

`/posts/{postId}/comment/{commentId}`
> Description

Updates an existing comment under a post
> Response Body Example
```
{
    comment_content: "comment content"
}
```

> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
postId  integer
        required

commentId   integer
            required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated comment"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error updating comment"
    }
    ```
<br />

### Language Endpoints
<br />

`GET`

`/languages`
> Description

Returns the list of languages
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: [
            {
                id: 1,
                language_name: "English"
            }
        ]
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error getting languages"
    }
    ```
<br />

`POST`

`/language/{languageId}`
> **ADMIN ONLY**

> Description

Adds or updates a language
> Example Request Body
```
{
    language_name: "German"
}
```

> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
languageId: integer
            optional
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated language"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error updating language"
    }
    ```
<br />


### Goal Endpoints
<br />

`GET`

`/goals`
> Description

Returns the list of goals
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: [
            {
                id: 1,
                goal_name: "Goal 1"
            }
        ]
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error getting goals"
    }
    ```
<br />

`POST`

`/goal/{goalId}`
> **ADMIN ONLY**

> Description

Adds or updates a goal
> Example Request Body
```
{
    goal_name: "Goal 1"
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
goalId: integer
        optional
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated goal"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error updating goal"
    }
    ```
<br />

### Interest Endpoints
<br />

`GET`

`/interests`
> Description

Returns the list of interests
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: [
            {
                id: 1,
                interest_name: "Reading"
            }
        ]
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error getting interests"
    }
    ```
<br />

`POST`

`/interest/{interestId}`
> **ADMIN ONLY**

> Description

Adds or updates an interest
> Example Request Body
```
{
    interest_name: "Interest 1"
}
```
> Query Parameters
```
user:   string
        encrypted and uri component encoded user email
        required
```
> Path Parameters
```
interestId: integer
            optional
```
> Response
- Success
    ```
    200
    {
        statusCode: 200,
        body: "Successfully updated interest"
    }
    ```
- Error
    ```
    400
    {
        statusCode: 400,
        body: "Error updating interest"
    }
    ```
<br />

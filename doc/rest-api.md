TODO : OAUTH authentication

All resources are prefixed with `/api`

The API uses JWT for authentication. It requires the `Authorization` with a valid JWT token.

# API version 1

All path are prefixed with `/v1`

## Users

Register a user.

`POST /users`
```js
Content-Type: application/json

{
  "email": <user email address>
  "name": <username>
  "secret": <password>
  "plan": <selected plan> // Free | Woobler
  "isCreator"?: <is the use a creator>
}
```
```js
HTTP/1.1 201 Created
Content-Type: application/json
Location: /tokens

{
  "data": {
    "email": <user email address>
    "name": <username>
    "secret": <password>
    "plan": <selected plan> // Free | Woobler
    "isCreator"?: <is the use a creator>
  }
}
```
---
Soft delete a user.

`DELETE /users`
```js
Authorization: <user token>
```
```js
HTTP/1.1 204 NoContent
Authorization: <refreshed token>
```
---
Update a authenticated user password or plan

`PATCH /users`
```js
Authorization: <user token>
Content-Type: application/json

{
  "newPassword"?: <new password>
  "oldPassword"?: <old password> // It should match the current password
  "plan"?: {
    "label": <selected plan> // Free | Woobler
  }
  "isCreator"?: <true or false>
  "website"?: <Website url>
  "codepenName"?: <Codepen username>
  "dribbbleName"?: <Dribbble username>
  "githubName"?: <Github username>
  "twitterName"?: <Twitter username>
}
```
```js
HTTP/1.1 204 NoContent
Authorization: <refreshed token>
Location: /users/<username>
```
---
Get a user infos. If the authentication token matches the requested user, it will return private informations.

`GET /users/<username>`
```js
Authorization: <user token>
```
```js
HTTP/1.1 200 OK
Content-Type: application/json
Authorization: <refreshed token>

{
  "data": {
    "name": <username>
    "fullname": <full user name>
    "profilePath": <Profile picture path>
    "isCreator": <true or false>
    "website": <Website url>
    "codepenName": <Codepen username>
    "dribbbleName": <Dribbble username>
    "githubName": <Github username>
    "twitterName": <Twitter username>

    *"email": <user email>
  }
}
```

## Token

Login a user.

`POST /tokens`
```js
Content-Type: application/json

{
  "email": <user email address>
  "secret": <password>
}
```
```js
HTTP/1.1 201 Created
Content-Type: application/json

{
  "data": {
    "token": <JWT token>
  }
}
```
---

Refresh a token.

`PUT /tokens`
```js
Authorization: <user token>
```
```js
HTTP/1.1 201 Created
Authorization: <refreshed token>

{
  "data": {
    "token": <access token>
  }
}
```

## Creations

Get all creations.

`GET /creations`
```js
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    {
      "id": <creation id>
      "title": <creation title>
      "description": <creation description>
      "thumbPath": <creation thumbnail path>
      "alias": <creation alias>
      "nbUse": <number of people who uses the creation>
      "version": <creation latest version>
      "versions": [<creation versions>]

      "creator": {
        "name": <creator name>
      }
      "isOwner": <true if the authenticated user is the owner of the creation>
      "createdAt": <creation date created>
      "updatedAt": <creation last updated date>
    }
    {
      ...
    }
  ]
}
```
---

Get a creation.

`GET /creations/:creaID`
```js
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "id": <creation id>
    "title": <creation title>
    "description": <creation description>
    "alias": <creation alias>
    "thumbPath": <creation thumbnail path>
    "previewUrl": <creation preview URL>
    "nbUse": <number of people who uses the creation>
    "version": <creation latest version>
    "versions": [<all creation versions>]

    "script": <creation script code>
    "document": <creation document code (HTML for instance)
    "style": <creation style>
    "params": [<creation parameters>]
    "functions": [<creation functions>]

    "creator": {
      "name": <creator name>
    }
    "isOwner": <true if the authenticated user is the owner of the creation>
    "createdAt": <creation date created>
    "updatedAt": <creation last updated date>
  }
}
```
---

Create a creation.

`POST /creations`
```js
Content-Type: application/json
Authorization: <user token>
Title string `json:"title" validate:"required,min=3,max=30"`

{
  "title": <creation title>
  "engine"?: <engine, JSES5 default>
  "alias"?: <creation alias, "woobly" by default>
  "state"?: <creation state, delete | draft | publish>
  "description"?: <creation description>
  "thumbPath"?: <creation thumbnail path>
  "params"?: [<creation parameters>]
  "functions"?: [<creation functions>]
}
```
```js
HTTP/1.1 201 Created
Location: /creations/:<id of the new creation>
Authorization: <refreshed token if expired>

{
  "data": {
    "id": <creation id>
    "title": <creation title>
    "creator": {
      "name": <creator name>
    }
    "versions": <creation versions>
    "createdAt": <creation date created>
    "updatedAt": <creation last updated date
  }
}
```
---
Update a creation.

`PUT /creations/:encid`
```js
Content-Type: application/json
Authorization: <user token>

{
  "title": <creation title>
  "description": <creation description>
  "alias": <creation alias>
  "thumbPath": <creation thumbnail path>
  "params": [<creation params>]
  "functions": [<creation functions]
  "version": <creation version>
}
```
```js
HTTP/1.1 204 NoContent
Location: /creations/:<id of the creation>
Authorization: <refreshed token if expired>
```
---
Delete the creation if nobody uses it, otherwise it will make the creation unlisted by putting its state to 'delete'.

`DELETE /creations/:encid`
```js
Authorization: <user token>
```
```js
HTTP/1.1 204 NoContent
Authorization: <refreshed token if expired>
```
---
Get a creation codes.

`GET /creations/:encid/code`
```js
Authorization: <user token>
```
```js
HTTP/1.1 200 OK
Content-Type: application/json
Authorization: <refreshed token if expired>

{
  "data": {
    "script": <script code>
    "document"?: <document code>
    "style"?: <style code>
  }
}
```
---
Create a new version. It will copy all the code of the current version and create a new version.

`POST /creations/:encid/versions`
```js
Content-Type: application/json
Authorization: <user token>

{
  "version": <version to create>
}
```
```js
HTTP/1.1 204 NoContent
Authorization: <refreshed token if expired>
Location: /creations/<creation id>/code
```
---
Save the code for the latest version, works only if the creation is in draft state.

`PUT /creations/:encid/versions`

```js
Content-Type: application/json
Authorization: <user token>

{
  "script": <script code>
  "document"?: <document code>
  "style"?: <style code>
}
```
```js
HTTP/1.1 204 NoContent
Authorization: <refreshed token if expired>
Location: /creations/<creation id>/versions
```
---
Regenerate default thumbnail.

`PATCH /creations/:encid`

```js
Content-Type: application/json
Authorization: <user token>

{
  "operation": <generateDefaultThumb>
}
```
```js
HTTP/1.1 200 OK
Authorization: <refreshed token if expired>

{
  "data": {
    "path": <thumbnail path>
    "base64": <base64 representation of the thumbnail>
  }
}
```

## Packages

Get all authenticated user packages.

`GET /packages`
```js
Authorization: <user token>
```
```js
HTTP/1.1 200 OK
Content-Type: application/json
Authorization: <refreshed token if expired>

{
  "data": [
    {
      "id": <package id>
      "title": <package title>
      "source": <package CDN source>
      "referer":[<referers associated to the package>]
      "nbCreations": <number of creations inside the package>
      "createdAt": <package creation date>
      "updatedAt"?: <package last update date>
    },
    {
      ...
    }
  ]
}
```
---
Get a package.

`GET /packages/:encid`
```js
Content-Type: application/json
Authorization: <user token>
```
```js
HTTP/1.1 200 OK
Content-Type: application/json
Authorization: <refreshed token if expired>

{
  "data": {
    "id": <package id>
    "title": <package title>
    "referer":[<referer associated to the package>]
    "createdAt": <package creation date>
    "updatedAt"?: <package last update date>
    "creations"?: [...]
  }
}
```
---
Create a package.

`POST /packages`
```js
Content-Type: application/json
Authorization: <user token>

{
  "title": <package title>
  "referer"?: [<referer with which the package will work>]
}
```
```js
HTTP/1.1 201 Created
Content-Type: application/json
Location: /packages/:<id of the new package>
Authorization: <refreshed token if expired>
```
---
Update a package.

`PUT /packages/:encid`
```js
Content-Type: application/json
Authorization: <user token>

{
  "title": <package title>
  "referer"?: [<referer with which the package will work>]
}
```
```js
HTTP/1.1 204 NoContent
Location: /packages/:<id of the new package>
Authorization: <refreshed token if expired>
```
---
Update some data of a package. Can be used to run `build` command.

`PATCH /packages/:encid`
```js
Content-Type: application/json
Authorization: <user token>

{
  "title"?: <package title>
  "referer"?: [<referer with which the package will work>]
  "operation"?: <command to run, build>
}
```
```js
HTTP/1.1 200 OK
Authorization: <refreshed token if expired>

{
  "data":{
    "source"?: <package source (CDN)>
  }
}
```
---
Delete a package.

`DELETE /packages/:encid`
```js
Authorization: <user token>
```
```js
HTTP/1.1 204 NoContent
Authorization: <refreshed token if expired>
```
---
Put a creation in a package.

`POST /packages/:encid/creations`
```js
Content-Type: application/json
Authorization: <user token>

{
  "creation": [<encoded ID of creations to push in package>]
}
```
```js
HTTP/1.1 204 NoContent
Location: /packages/:<id of the package>
Authorization: <refreshed token if expired>
```
---
Remove a creation from a package.


`DELETE /packages/:encid/creations/:encidDelete`
```js
Authorization: <user token>
```
```js
HTTP/1.1 204 NoContent
Authorization: <refreshed token if expired>
```

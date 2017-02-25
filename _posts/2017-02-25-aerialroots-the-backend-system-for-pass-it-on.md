---
layout: post
title: "AerialRoots, the backend system for Pass It On!"
description: ""
category: 
tags: []
---
{% include JB/setup %}

<!-- ## AerialRoots, the backend system for Pass-It-On!  -->

This document explains the network calls to the api endpoints related to User and Item.

## Endpoint security

The backend uses an HTTPS connection, and the android app lets users sign in using Identity(Email and password) or Facebook credentials. A [JsonWebToken](https://jwt.io/) is assigned to the user on successful login and the token validates incoming requests. The token expires after an expiry date which is configured to 30 days. The authentication process is implemented using the [Guardian Ueberauth](https://github.com/ueberauth/guardian) framework. The authentication service is easy to extend to multiple Auth providers, like Google, Twitter etc.

## Routes and Controllers
On successful login, the user can post give-away or needed items. The routes for all the accessible endpoints are configured in: 
```sh 
~/web/router.ex
```
Each route in the [router.ex](https://bitbucket.org/leapingwolf/aerialroots/src/29532f4cc9925ff12a286621c61dd158780956ad/web/router.ex?at=master&fileviewer=file-view-default) file corresponds to a function in a controller.

Example routes in  [**UserController**](https://bitbucket.org/leapingwolf/aerialroots/src/c0047f5e7608438814d648df81e70294edb8d121/web/controllers/user_controller.ex?at=master&fileviewer=file-view-default):
```elixir
    resources "/users", UserController do
      #routes to add items and get items for a user
      post "/items", UserController, :add_item
      get "/items", UserController,  :get_item
    end
```
The actions in the example correspond to  **UserController**

|Action | Function | Function parameters | Url | Request json example |
|-------|----------|---------------------|-----|----------------------|
| GET Items listed by a user |  ~/web/controllers/user_controller/get_item | user_id, page,orderBy | /auth/users/${userId}/items?page=${page}&orderBy=${orderBy} |  NONE |
|POST - Item posted by a user | ~/web/controllers/user_controller/add_item | user_id, item | /auth/users/${user_id}/items   | {"user_id": ${user_id}, "item": {"available": ${available}, "description": ${description}, "id": ${id}, "reservationEndDate": ${reservationEndDate}, "title": ${title}}} |

Example routes in  [**ItemController**](https://bitbucket.org/leapingwolf/aerialroots/src/b2b08a1f2585cf11a8c04c9c35ac83997f1f411d/web/controllers/item_controller.ex?at=master&fileviewer=file-view-default):
Actions for item update and delete to the item are configured as 
```sh
resources "/items", ItemController, except: [:new, :edit, :create] do
end
```
The actions in the example correspond to **ItemController**

| Action | Function | Function parameters | Url | Request json example |
|--------|----------|---------------------|-----|----------------------|
| GET - All items | ~/web/controllers/item_controller/index | page,orderBy | /auth/items?page=${page}&orderBy=${orderBy} |
| GET - Selected item | ~/web/controllers/item_controller/show | item_id  | /auth/items/${item_id} |
| PUT - Updating an item | ~/web/controllers/item_controller/update | item_id, item |  /auth/items/${itemId} | {“item": {"available": ${available}, "description": ${description}, "id": ${id}, "reservationEndDate": ${reservationEndDate}, "title": ${title}}
 | DELETE - Deleting an item |  /web/controllers/item_controller/delete|item_id|/auth/items/${itemId} |

Note the both the controllers **UserController** and **ItemController** have the line at the top of the file
```sh
plug Guardian.Plug.EnsureAuthenticated, handler: __MODULE__
```
This code ensures that none of the functions in these controllers can be invoked without a valid bearer token. Any attempt to do so will invoke the `unauthenticated` function in the corresponding controller.

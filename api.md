# lataamo.akuankka.fi 
# Unofficial API v1 doc

This document describes the API and some details about the inner workings of [lataamo.akuankka.fi](https://lataamo.akuankka.fi).

This information was acquired by reverse engineering the website, so some of the details may be incorrect and/or subject to change. It should only be used for learning purposes. 

## Inner workings

### Backend

Not much is known about the backend at the time. The site is running on [node.js](http://nodejs.org/) and [nginx 1.0.15](http://nginx.org/). I have no clue what database the site uses, but I would guess it's either [MongoDB](http://www.mongodb.org/) or [Redis](http://redis.io/).

### Frontend

The frontend is written in HTML5 and Javascript. Most of the frontend source code is in a single obfuscated JS file, [main.js](https://lataamo.akuankka.fi/main.js). The frontend uses a ton of third party libraries, such as:

* [jQuery 1.9.0](http://jquery.org/)
* [RequireJS 0.1.0](http://github.com/jrburke/requirejs)
* [FastClick 0.6.9](http://github.com/ftlabs/fastclick)
* [moment.js 2.0.0](http://momentjs.com/)
* [Underscore.js](http://underscorejs.org/)
* [Backbone.js](http://backbonejs.org/)
* [Backbone.Layoutmanager](http://layoutmanager.org/)
* [jQuery UI 1.10.2](http://jqueryui.com)
* [jquery-placeholder 2.0.7](http://mths.be/placeholder)
* [iScroll 4.2.5](http://iscrolljs.com/)
* [Hammer.js 1.0.5](http://eightmedia.github.io/hammer.js/)
* [Platform.js 1.0.0](http://mths.be/platform)
* [Spinners 3.0.0](http://github.com/staaky/spinners)
* [Lazy Load 1.8.4](http://www.appelsiini.net/projects/lazyload)

## API
REST + JSON. You know the drill.

The base URL for all requests is [lataamo.akuankka.fi/api/v1/](https://lataamo.akuankka.fi/api/v1/). The email address `user@example.com` is used in place of a real address.

There are some common optional parameters which work for every method that returns a list:

* offset - 0-based index from the beginning of the response list. If the list has a lot of items, you may have to send multiple requests with different offsets, since the number of items you can get at a time is limited.
* limit - The maximum number of items a request will return.

## API Methods

### POST /authenticate

#### Description

Used for authenticating a user.

#### Requires Authentication

No.

#### URL Structure

    https://lataamo.akuankka.fi/api/v1/authenticate

#### Parameters

Username (email) and password as JSON:

    {
       "username":"user@example.com",
       "password":"hunter2"
    }

#### Returns

If password or username was wrong:

403 Forbidden

    {
       "status":"error",
       "message":"Käyttäjätunnus tai salasana väärin."
    }   

If login was successful or user has already logged in:

200 Ok

    {
       "status":"success",
       "current_user":{
          "href":"https://api.akuankka.fi/v1/users/user@example.com",
          "shogun_id":"32_character_alphanumeric_hash"
       }
    }

The response sets two cookies, `smf` (session id) and `smf_login` (user email).

### POST /logout

#### Description

Used for logging out a user.

#### Requires Authentication

Yes.

#### URL Structure

    https://lataamo.akuankka.fi/api/v1/logout

#### Parameters

None.

#### Returns

    {
       "status":"success"
    }

The `smf` (session id) cookie gets a new value and `smf_login` (user email) cookie is deleted.

### GET /users/user@example.com/favorites

#### Description

Gets the stories user has added to their favorites.

#### Requires Authentication

Yes.

#### URL Structure

    https://lataamo.akuankka.fi/api/v1/users/user@example.com/favorites

#### Parameters

* User email - Doesn't have an effect. As long as there is _something_ between /users/ and /favorites, the method will return the favorites of the currently logged in user.
* _offset_
* _limit_

#### Returns

A list of stories.

#### Sample

`GET https://lataamo.akuankka.fi/api/v1/users/user@example.com/favorites`

    {
       "href":"http://api.akuankka.fi/v1/users/user@example.com/favorites",
       "count":1,
       "offset":0,
       "stories":[
          {
             "favorite_id":19370,
             "href":"https://api.akuankka.fi/v1/stories/d-97346",
             "slug_issue":"fi-aa1998-23",
             "title":"Musta ritari",
             "listing_image":"https://api.akuankka.fi/images/preview/26150/26150_12_01_@2x.jpg",
             "small_listing_image":"https://api.akuankka.fi/images/preview/26150/26150_12_01_small@2x.jpg",
             "page_count":12,
             "average_rating":4.52,
             "main_characters":[
                {
                   "name":"Roope-setä",
                   "href":"https://api.akuankka.fi/v1/characters/roope-seta"
                }
             ]
          }
       ]
    }

### GET /users/user@example.com/ratings

#### Description

Gets the stories user has rated.

#### Requires Authentication

Yes.

#### URL Structure

    https://lataamo.akuankka.fi/api/v1/users/user@example.com/ratings

#### Parameters

* User email - Once again, doesn't make a difference.
* _offset_
* _limit_

#### Returns

A list of stories.

#### Sample

`GET https://lataamo.akuankka.fi/api/v1/users/user@example.com/ratings?limit=4`

    {
       "href":"http://api.akuankka.fi/v1/users/user@example.com/ratings?limit=4",
       "count":12,
       "offset":0,
       "stories":[
          {
             "rating_id":49189,
             "rating":3,
             "href":"https://api.akuankka.fi/v1/stories/w-wdc-169-03",
             "title":"Travelling Truants",
             "page_count":10
          },
          {
             "rating_id":49079,
             "rating":4,
             "href":"https://api.akuankka.fi/v1/stories/w-wdc-145-01",
             "title":"The Hypno-Gun",
             "page_count":10
          },
          {
             "rating_id":49637,
             "rating":5,
             "href":"https://api.akuankka.fi/v1/stories/d-90314",
             "title":"Takaisin Xanaduun",
             "page_count":10
          },
          {
             "rating_id":49080,
             "rating":4,
             "href":"https://api.akuankka.fi/v1/stories/w-wdc-141-02",
             "title":"Ajatuslaatikot",
             "page_count":10
          }
       ]
    }    

### GET /users/user@example.com/readlimits

#### Description

Gets how many and which stories the user can read and has read this month.

#### Requires Authentication

Yes.

#### URL Structure

    https://lataamo.akuankka.fi/api/v1/users/user@example.com/readlimits

#### Parameters 

* User email - Doesn't matter.
* _offset_
* _limit_

#### Returns

Two numbers, `stories_limit` (amount of stories the user can read this month, currently 500) and `stories_read` (amount of stories the user has read this month), and a list of stories.

#### Sample

`GET https://lataamo.akuankka.fi/api/v1/users/user@example.com/readlimits?limit=4`

    {
       "href":"http://api.akuankka.fi/v1/users/user@example.com/readlimits?limit=4",
       "stories_limit":500,
       "stories_read":32,
       "stories":[
          {
             "slug":"zd-44-10-29"
          },
          {
             "slug":"d-2575"
          },
          {
             "slug":"w-pb-7-02"
          },
          {
             "slug":"zd-42-03-08"
          }
       ]
    }

### GET /users/user@example.com/notifications

Gets the current notifications.

#### Requires Authentication

Yes.

#### URL Structure

    https://lataamo.akuankka.fi/api/v1/users/user@example.com/notifications

#### Parameters

* User email - This time, for some inexplicable reason, it must be the same as the real user email. No idea why.

#### Returns

A list of notifications, but the list always contains only a single notification.

#### Sample

`GET https://lataamo.akuankka.fi/api/v1/users/user@example.com/notifications`

    {
       "notifications":[
          {
             "id":39,
             "short_content":"Lystiä laskiaistiistaita! Lumen puutteessa voi lohduttautua lukemalla vaikkapa tarinan Liukasta luikua (AA 3\/2009).",
             "content":"Lystiä laskiaistiistaita! Lumen puutteessa voi lohduttautua lukemalla vaikkapa tarinan Liukasta luikua (AA 3\/2009).",
             "published_start":"2014-03-04T18:00:00+0200"
          }
       ]
    }   

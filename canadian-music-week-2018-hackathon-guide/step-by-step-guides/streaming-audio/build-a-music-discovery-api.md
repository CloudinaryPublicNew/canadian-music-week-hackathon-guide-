# Build a Music Discovery API

## Build a Music Discovery API

By Dan Zeitman

In this Step-By-Step Guide we'll show you how to build an API wrapper for multiple web services in order to create a music discovery service backend.

We'll create a wrapper for 7Digital's node APIs. 

We're going to build a series of api endpoints that will be hosted on Auth0's Webtask. The design pattern is based off Express so it will be familiar to many.

## Let's get started:

#### Setting Up Cloudinary

The first thing you’re going to have to do is set up a Cloudinary account.

So, go ahead and pop on over to the [signup page](https://cloudinary.com/signup?utm_source=CMW&utm_medium=Gitbook&utm_campaign=Evangelism&utm_term=Hackathon-Guide&utm_content=Signup_CMW).

![Screenshot of the signup page](https://eric-cloudinary-res.cloudinary.com/image/upload/q_auto,f_auto,w_900/v1518532546/Screen_Shot_2018-02-13_at_06.35.17.png)

Once you’ve filled out the form \(don’t forget to edit and customize your cloud name, if you’re so inclined!\) and clicked through a little customer survey, you’re all set. Feel free to click around and check out your new account – there’s a lot of interesting stuff here to see. Today, however, we’re going to be using almost none of it.

#### Setting Auth0 / Webtask

> [Signup for Auth0's Webtask service](https://webtask.io/make)

Once you've logged in create an empty task and run it to get familiar with the platform and editor.

Create a new task called:

> **music-discovery-service**

Copy the source code of our [Music Discovery Service API](https://raw.githubusercontent.com/cloudinary-developers/music-discovery-service/master/music-discovery-service.js) \(GitHub\)

```text
###Add The Secrets

What is the Context Object

In Webtask, there is a context object available with quite a few useful properties that can be accessed while running your tasks.
```code 

var context = {
  body,
  meta,
  storage,
  params,
  query,
  secrets,
  headers,
  data
};
```

You will want to store your api and access keys in the context.secrets

#### Add the NPM Modules

#### Test The service:

[https://\(your-cloud-url\)/music-discovery-service/\(api-end-point\)/\(params](https://%28your-cloud-url%29/music-discovery-service/%28api-end-point%29/%28params)\)

## APIS:

### Browse

**/browse/\(letter\)**

**Returns artists id.**

```text
https://evangelism.cloudinary.auth0-extend.com/sxsw-music-discovery-service/browse/b
```

### Search:

**/search/\(name\)**

**Returns Artist Info:**

```text
https://evangelism.cloudinary.auth0-extend.com/sxsw-music-discovery-service/search/Cyndi%20Lauper
```

### Releases

**/releases/\(artistid\)**

**Returns  releases by that Artist.**

```text
https://evangelism.cloudinary.auth0-extend.com/sxsw-music-discovery-service/releases/11319
```

### tracks

**/tracks/\(released\)**

**Returns all the tracks for that release:**

```text
https://evangelism.cloudinary.auth0-extend.com/sxsw-music-discovery-service/tracks/5514991
```

### Lyrics

**/lyrics/\(ssrc\)**

**Returns lyrics and a NLP reduced keyword list**

```text
https://evangelism.cloudinary.auth0-extend.com/sxsw-music-discovery-service/lyrics/USCJ81000500
```


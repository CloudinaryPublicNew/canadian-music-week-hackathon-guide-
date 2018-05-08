# Build a Music Discovery API

## Build a Music Discovery API

By Dan Zeitman

In this Step-By-Step Guide we'll show you how to build an API wrapper for multiple web services in order to create a music discovery service backend.

We'll create an express based app, running on **Auth0 Webtask,**  that will combine primary API's from [CLOUDINARY](https://cloudinary.com/signup?utm_source=CMW&utm_medium=Gitbook&utm_campaign=Evangelism&utm_term=Hackathon-Guide&utm_content=Signup_CMW) and [7DIGITAL](http://docs.7digital.com/#catalogue).   In this demonstration, we're going to build a series of api endpoints that will be hosted on Auth0's Webtask. The design pattern is based off Express so it will be familiar to many.

The 7Digital APIs provide many methods for browsing and streaming a catalog of tracks made available by Capitol Music Group \(CMG\). Capitol Music Group \(CMG\) is comprised of Capitol Records, Virgin Records, Motown Records, Blue Note Records, Astralwerks, Harvest Records, Capitol Christian Music Group, and CMG’s independent distribution division, Caroline.

The Cloudinary API's provide an end-to-end image solution for managing and transformation  image and video assets. 

With this sample application you'll  be able to Stream Songs like this:

[https://canadian-music-week.cloudinary.auth0-extend.com/music-discovery-service/song/40349901/stream](https://canadian-music-week.cloudinary.auth0-extend.com/music-discovery-service/song/40349901/stream)

Search for artists, get the tracks and integrate with Cloudinary.

### Brief Overview of the Music Discovery Service Demo:

1. Signup for Cloudinary and Auth0 Webtask.
2. Create a new Webtask and replace with our source code example.
3. Add each NPM modules to the Webtask.
4. Create and populate the Webtask SECRETS with your various API credentials.
5. Review and Test each Endpoint
6. Extend the Service with additional features and functionality.

### Let's get started:

#### Setting Up Cloudinary

The first thing you’re going to have to do is set up a Cloudinary account.

So, go ahead and pop on over to the [signup page](https://cloudinary.com/signup?utm_source=CMW&utm_medium=Gitbook&utm_campaign=Evangelism&utm_term=Hackathon-Guide&utm_content=Signup_CMW).

Once you’ve filled out the form \(don’t forget to edit and customize your cloud name, if you’re so inclined!\) and clicked through a little customer survey, you’re all set. Feel free to click around and check out your new account – there’s a lot of interesting stuff here to see. Today, however, we’re going to be using almost none of it.

### Find your Cloud Name and Credentials

![Cloudinary Signup](https://eric-cloudinary-res.cloudinary.com/image/upload/q_auto,f_auto,w_900/v1518532546/Screen_Shot_2018-02-13_at_06.35.17.png)

{% hint style="success" %}
Cloudinary Setup
{% endhint %}

#### Setting Auth0 / Webtask

> [Signup for Auth0's Webtask service](https://webtask.io/make)

Once you've logged in create an empty task and run it to get familiar with the platform and editor.

{% hint style="success" %}
Auth0 Webtask Setup
{% endhint %}

Log in to Cloudinary,  you can view your credentials on the main Dashboard:

![Copy your credentials](../../../.gitbook/assets/credentials.png)

Create a new task called:

> **music-discovery-service**

Copy the source code of our [Music Discovery Service API](https://gist.github.com/dzeitman/8e78b79e39de113e75945bd201c6baaa) \(GIST\)



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


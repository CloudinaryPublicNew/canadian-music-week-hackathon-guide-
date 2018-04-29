# Build a Music Discovery API

## Build a Music Discovery API

By Dan Zeitman

In this Step-By-Step Guide we'll show you how to build an API wrapper for multiple web services in order to create a music discovery service backend.

We'll create an express based app, running on **Auth0 Webtask,**  that will combine primary API's from CLOUDINARY and 7DIGITAL.   In this demonstration, we're going to build a series of api endpoints that will be hosted on Auth0's Webtask. The design pattern is based off Express so it will be familiar to many.

The 7Digital APIs provide many methods for browsing and streaming a catalog of tracks made available by Capitol Music Group \(CMG\). Capitol Music Group \(CMG\) is comprised of Capitol Records, Virgin Records, Motown Records, Blue Note Records, Astralwerks, Harvest Records, Capitol Christian Music Group, and CMG’s independent distribution division, Caroline.

The Cloudinary API's provide an end-to-end image solution for managing and transformation  image and video assets. 

### Let's get started:

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

{% embed data="{\"url\":\"https://github.com/cloudinary-developers/music-discovery-service/blob/master/music-discovery-service.js\",\"type\":\"link\",\"title\":\"cloudinary-developers/music-discovery-service\",\"description\":\"music-discovery-service - Music Discovery Service\",\"icon\":{\"type\":\"icon\",\"url\":\"https://github.com/fluidicon.png\",\"aspectRatio\":0},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://avatars1.githubusercontent.com/u/33458021?s=400&v=4\",\"width\":400,\"height\":400,\"aspectRatio\":1}}" %}

```text
/*
Music Discovery Service 
*/
const cloudinary = require('cloudinary');
const express    = require('express');
const Webtask    = require('webtask-tools');
const bodyParser = require('body-parser');
const request = require('request');
const JSONP = require('node-jsonp');
const Algorithmia = require('algorithmia');
var app = express();
var algorithmia_key, musicmatch_api_key, api, artists, tracks, releases , consumerkey, consumersecret;
app.use(bodyParser.json());
// Our Middleware to setup API 
var apiContext = function (req, res, next) {
  const context = req.webtaskContext;
  // config cloudinary  
  cloudinary.config({
      "cloud_name": context.secrets.cloud_name,
      "api_key": context.secrets.api_key,
      "api_secret": context.secrets.api_secret
    });
  // paging 
  const page = context.data.page || 1;
  const pageSize = context.data.pageSize || 100;
  //Algorithmia key
  algorithmia_key = context.secrets.algorithmia_key;
  //Music Match
  musicmatch_api_key = context.secrets.musicmatch_api_key;
  // 7digital-api
  consumerkey = context.secrets.oauth_consumer_key;
  consumersecret =  context.secrets.oauth_consumer_secret;
  api = require('7digital-api').configure({
   format: 'JSON',
   consumerkey: context.secrets.oauth_consumer_key,
   consumersecret: context.secrets.oauth_consumer_secret,
   defaultParams: { 
       country: 'GB', 
        shopId: context.secrets.shop_id,
       usageTypes: 'adsupportedstreaming',  
       pageSize: pageSize, 
       page:page, 
       imageSize:800,
       sort: 'popularity desc'
   }
});
// create instances of individual apis
  artists = new api.Artists();
  releases = new api.Releases();
  tracks = new api.Tracks();
  console.log('API Inited.')
  next()
}
// Use our API Middleware
app.use(apiContext)
var getLyrics = function(params){
  const data = {
    format:'jsonp',
    callback: 'callback',
    q_track: params.q_track,
    q_artist: params.q_artist,
    track_isrc: params.track_isrc,
    apikey: musicmatch_api_key
  };
  // musixmatch api
  const url = 'https://api.musixmatch.com/ws/1.1/matcher.lyrics.get';
  return new Promise(function (resolve, reject) {
  JSONP(url,data,'callback',function(response){
     console.log(response.message.body);
     const lyrics =  response.message.body.lyrics;
       if(lyrics){
         resolve(lyrics);  
       }else{
         reject("There was an error getting lyrics");
       }
    });
  });
}
// USMC14673497  
// 'USCJ81000500'// 'GBAFL1700342';  //?Spacewoman
app.get('/lyrics/:isrc', function (req, res) {
  var track_isrc = req.params.isrc  || 'GBAFL1700342'; 
  const context = req.webtaskContext;
  const data = { track_isrc: track_isrc };
   getLyrics(data)
   .then(function(lyrics){
    Algorithmia.client(algorithmia_key)
    .algo("nlp/AutoTag/1.0.1")
    .pipe(lyrics.lyrics_body)
    .then(function(response) {
        console.log(response.get());
        var lyrics_body = lyrics.lyrics_body.replace('******* This Lyrics is NOT for Commercial use *******','');
        var results = { words:response.get(), lyrics: lyrics_body};
        res.send(results);
    });
   })
   .catch(function(error){
          res.send(error);
   });
});
var getSong = function(context, trackid){
// Create a Signed URL
var oauth = new api.OAuth();
    return new Promise(function (resolve, reject) {
       var apiUrl = 'https://stream.svc.7digital.net/stream/catalogue?country=GB&trackid=' + trackid;
       var signedURL = oauth.sign(apiUrl);
       if(signedURL){
          console.log(signedURL)
          resolve({url:signedURL});
       }else{
          reject('we had an error');
       }
      });
}
// /song/70540913/stream/
app.get('/song/:trackid/?:stream', function ( req, res) {
  const trackid = req.params.trackid  || '123456';  // /song/12345
  const context = req.webtaskContext;
  const shouldStream = req.params.stream  || "url";
  console.log(trackid);
  console.log(shouldStream);
  getSong(context, trackid).then(function(data){
      if(shouldStream == 'stream'){
        request(data.url).pipe(res);
      }else{
        res.send( data);   
      }
   }).catch(function(err){
      console.log('ERR:', Err);
      res.send(err);
   })
});
var getClip = function(trackid){
    return new Promise(function (resolve, reject) {
      var clipUrl = 'http://previews.7digital.com/clip/' + trackid;
      const oauth = new api.OAuth();
      var previewUrl = oauth.sign(clipUrl);
       if(previewUrl){
          resolve({ url:previewUrl });
       }else{
          reject('we had an error');
       }
      });
}
 // /song/70540913/stream/
 app.get('/clip/:trackid/?:stream', function ( req, res) {
  var trackid = req.params.trackid || '12345';   // /clip/12345
  const context = req.webtaskContext;
  const shouldStream = req.params.stream  || "url";
  getClip(trackid)
  .then(function(data){
      if(shouldStream == 'stream'){
        request(data.url).pipe(res);
      }else{
        res.send( data);   
      }
   })
   .catch(function(err){
      console.log('ERR:', Err);
      res.send(err);
   })
});
var browse = function(letter) {  
  return new Promise(function (resolve, reject) {
       artists.browse({ letter: letter }, function(err, data) {
              if(err){
               reject(err)
              }
              if(data){
                resolve(data);
              } 
            });    
  });
}
app.get('/browse/:letter', function ( req, res) {
  const letter = req.params.letter;   // /browse/letter
  browse(letter).then(function(data){
        console.log(JSON.stringify(data,null,5));
        res.send( data);   
   }).catch(function(error){
      console.log('error:', error);
      res.send(error);
   })
});
  var search = function(query) {  
  return new Promise(function (resolve, reject) {
        artists.search({ q: query }, function(err, data) {
        if(err){
          console.log(err);
            reject(err)
        }
        if(data){
          console.log(JSON.stringify(data,null,5));
          resolve(data);
        } 
      });
  })
}
app.get('/search/:query', function ( req, res) {
  const query = req.params.query || 1;
  search(query).then(function(data){
        res.send( data);   
   })
   .catch(function(error){
      console.log('error: ', error);
      res.send(error);
   })
});
  //14643 The Breeders
var getReleases = function(artistID) {  
  return new Promise(function (resolve, reject) {
        artists.getReleases({ artistid: artistID }, function(err, data) {
        if(err){
          console.log(err);
            reject(err)
        }
        if(data){
          console.log(data);
          resolve(data);
        } 
      });
  })
}
app.get('/releases/:artistid', function ( req, res) {
const artistid = req.params.artistid || '14643';
  getReleases(artistid)
  .then(function(data){
        res.send( data);   
   }).catch(function(error){
      console.log('error:', error);
      res.send(error);
   })
});
// Save Cover Image 
var uploadImage = function(coverImageURL, public_id){
return  new Promise(function (resolve, reject) {
  var url = coverImageURL || 'http://res.cloudinary.com/de-demo/video/upload/v1520429530/test-audio.mp3' ; 
        // uses upload preset:  https://cloudinary.com/console/settings/upload
        cloudinary.v2.uploader.upload(url, 
              { 
              upload_preset: 'sxsw',  
              public_id: public_id,  
              type: "upload",
              resource_type: "image", 
              }, 
          function(error, result) {
            if(error){
                   reject( error);
            }
            if(result){
              console.log(result);
                    resolve(result);
            }
          });
        });
}
app.get('/upload', function ( req, res) {
var url = req.params.url || 'http://artwork-cdn.7static.com/static/img/sleeveart/00/055/149/0005514991_800.jpg';
///'http://artwork-cdn.7static.com/static/img/artistimages/00/000/113/0000011319_300.jpg';
var public_id = req.params.publicid || 'Cyndi_Lauper_cover';
console.log(url, public_id);
// res.send({url:url, public_id:public_id}); 
  uploadImage(url,public_id)
  .then(function(data){
        res.send(data);   
  }).catch(function(error){
      console.log('error:', error);
      res.send(error);
  })
});
var getImagesByTags = function(tags){
return  new Promise(function (resolve, reject) {
           cloudinary.v2.api.resources_by_tag(tags,{max_results:100, tags:true}, 
           function(error, result){
             if(error){
               reject(error);
             }
             if(result){
               console.log(result);
               resolve(result);
             }
           });
        });
}
// Get tracks by releaseID: 
var getTracks = function(releaseid) {  
  return new Promise(function (resolve, reject) {
        releases.getTracks({ releaseid: releaseid }, function(err, data) {
        if(err){
          console.log(err);
            reject(err)
        }
        if(data){
          resolve(data);
        } 
      });
  })
}
// Get tracks by releaseID: 7456808
app.get('/tracks/:releaseid', function ( req, res ) {
  const releaseid = req.params.releaseid || '7026306';
    console.log(releaseid);
    getTracks(releaseid)
    .then(function(data){
      console.log(JSON.stringify(data,null,5));
      res.send(data); 
    }).catch(function(error){
       res.send(error);
    });
});
app.get('/', function (req, res) {
  const html = `<a href="https://cloudinary.gitbook.io/canadian-music-week-hackathon-guide">Hackathon Guide<a>`;
    res.send(html); 
});
module.exports = Webtask.fromExpress(app);
p.p1 {margin: 0.0px 0.0px 0.0px 0.0px; font: 12.0px 'Helvetica Neue'; color: #454545}
p.p2 {margin: 0.0px 0.0px 0.0px 0.0px; font: 12.0px 'Helvetica Neue'; color: #454545; min-height: 14.0px}
span.Apple-tab-span {white-space:pre}

/*
Music Discovery Service 
*/
const cloudinary = require('cloudinary');
const express    = require('express');
const Webtask    = require('webtask-tools');
const bodyParser = require('body-parser');
const request = require('request');
const JSONP = require('node-jsonp');
const Algorithmia = require('algorithmia');
var app = express();
var algorithmia_key, musicmatch_api_key, api, artists, tracks, releases , consumerkey, consumersecret;
app.use(bodyParser.json());
// Our Middleware to setup API 
var apiContext = function (req, res, next) {
  const context = req.webtaskContext;
  // config cloudinary  
  cloudinary.config({
      "cloud_name": context.secrets.cloud_name,
      "api_key": context.secrets.api_key,
      "api_secret": context.secrets.api_secret
    });
  // paging 
  const page = context.data.page || 1;
  const pageSize = context.data.pageSize || 100;
  //Algorithmia key
  algorithmia_key = context.secrets.algorithmia_key;
  //Music Match
  musicmatch_api_key = context.secrets.musicmatch_api_key;
  // 7digital-api
  consumerkey = context.secrets.oauth_consumer_key;
  consumersecret =  context.secrets.oauth_consumer_secret;
  api = require('7digital-api').configure({
   format: 'JSON',
   consumerkey: context.secrets.oauth_consumer_key,
   consumersecret: context.secrets.oauth_consumer_secret,
   defaultParams: { 
       country: 'GB', 
        shopId: context.secrets.shop_id,
       usageTypes: 'adsupportedstreaming',  
       pageSize: pageSize, 
       page:page, 
       imageSize:800,
       sort: 'popularity desc'
   }
});
// create instances of individual apis
  artists = new api.Artists();
  releases = new api.Releases();
  tracks = new api.Tracks();
  console.log('API Inited.')
  next()
}
// Use our API Middleware
app.use(apiContext)
var getLyrics = function(params){
  const data = {
    format:'jsonp',
    callback: 'callback',
    q_track: params.q_track,
    q_artist: params.q_artist,
    track_isrc: params.track_isrc,
    apikey: musicmatch_api_key
  };
  // musixmatch api
  const url = 'https://api.musixmatch.com/ws/1.1/matcher.lyrics.get';
  return new Promise(function (resolve, reject) {
  JSONP(url,data,'callback',function(response){
     console.log(response.message.body);
     const lyrics =  response.message.body.lyrics;
       if(lyrics){
         resolve(lyrics);  
       }else{
         reject("There was an error getting lyrics");
       }
    });
  });
}
// USMC14673497  
// 'USCJ81000500'// 'GBAFL1700342';  //?Spacewoman
app.get('/lyrics/:isrc', function (req, res) {
  var track_isrc = req.params.isrc  || 'GBAFL1700342'; 
  const context = req.webtaskContext;
  const data = { track_isrc: track_isrc };
   getLyrics(data)
   .then(function(lyrics){
    Algorithmia.client(algorithmia_key)
    .algo("nlp/AutoTag/1.0.1")
    .pipe(lyrics.lyrics_body)
    .then(function(response) {
        console.log(response.get());
        var lyrics_body = lyrics.lyrics_body.replace('******* This Lyrics is NOT for Commercial use *******','');
        var results = { words:response.get(), lyrics: lyrics_body};
        res.send(results);
    });
   })
   .catch(function(error){
          res.send(error);
   });
});
var getSong = function(context, trackid){
// Create a Signed URL
var oauth = new api.OAuth();
    return new Promise(function (resolve, reject) {
       var apiUrl = 'https://stream.svc.7digital.net/stream/catalogue?country=GB&trackid=' + trackid;
       var signedURL = oauth.sign(apiUrl);
       if(signedURL){
          console.log(signedURL)
          resolve({url:signedURL});
       }else{
          reject('we had an error');
       }
      });
}
// /song/70540913/stream/
app.get('/song/:trackid/?:stream', function ( req, res) {
  const trackid = req.params.trackid  || '123456';  // /song/12345
  const context = req.webtaskContext;
  const shouldStream = req.params.stream  || "url";
  console.log(trackid);
  console.log(shouldStream);
  getSong(context, trackid).then(function(data){
      if(shouldStream == 'stream'){
        request(data.url).pipe(res);
      }else{
        res.send( data);   
      }
   }).catch(function(err){
      console.log('ERR:', Err);
      res.send(err);
   })
});
var getClip = function(trackid){
    return new Promise(function (resolve, reject) {
      var clipUrl = 'http://previews.7digital.com/clip/' + trackid;
      const oauth = new api.OAuth();
      var previewUrl = oauth.sign(clipUrl);
       if(previewUrl){
          resolve({ url:previewUrl });
       }else{
          reject('we had an error');
       }
      });
}
 // /song/70540913/stream/
 app.get('/clip/:trackid/?:stream', function ( req, res) {
  var trackid = req.params.trackid || '12345';   // /clip/12345
  const context = req.webtaskContext;
  const shouldStream = req.params.stream  || "url";
  getClip(trackid)
  .then(function(data){
      if(shouldStream == 'stream'){
        request(data.url).pipe(res);
      }else{
        res.send( data);   
      }
   })
   .catch(function(err){
      console.log('ERR:', Err);
      res.send(err);
   })
});
var browse = function(letter) {  
  return new Promise(function (resolve, reject) {
       artists.browse({ letter: letter }, function(err, data) {
              if(err){
               reject(err)
              }
              if(data){
                resolve(data);
              } 
            });    
  });
}
app.get('/browse/:letter', function ( req, res) {
  const letter = req.params.letter;   // /browse/letter
  browse(letter).then(function(data){
        console.log(JSON.stringify(data,null,5));
        res.send( data);   
   }).catch(function(error){
      console.log('error:', error);
      res.send(error);
   })
});
  var search = function(query) {  
  return new Promise(function (resolve, reject) {
        artists.search({ q: query }, function(err, data) {
        if(err){
          console.log(err);
            reject(err)
        }
        if(data){
          console.log(JSON.stringify(data,null,5));
          resolve(data);
        } 
      });
  })
}
app.get('/search/:query', function ( req, res) {
  const query = req.params.query || 1;
  search(query).then(function(data){
        res.send( data);   
   })
   .catch(function(error){
      console.log('error: ', error);
      res.send(error);
   })
});
  //14643 The Breeders
var getReleases = function(artistID) {  
  return new Promise(function (resolve, reject) {
        artists.getReleases({ artistid: artistID }, function(err, data) {
        if(err){
          console.log(err);
            reject(err)
        }
        if(data){
          console.log(data);
          resolve(data);
        } 
      });
  })
}
app.get('/releases/:artistid', function ( req, res) {
const artistid = req.params.artistid || '14643';
  getReleases(artistid)
  .then(function(data){
        res.send( data);   
   }).catch(function(error){
      console.log('error:', error);
      res.send(error);
   })
});
// Save Cover Image 
var uploadImage = function(coverImageURL, public_id){
return  new Promise(function (resolve, reject) {
  var url = coverImageURL || 'http://res.cloudinary.com/de-demo/video/upload/v1520429530/test-audio.mp3' ; 
        // uses upload preset:  https://cloudinary.com/console/settings/upload
        cloudinary.v2.uploader.upload(url, 
              { 
              upload_preset: 'sxsw',  
              public_id: public_id,  
              type: "upload",
              resource_type: "image", 
              }, 
          function(error, result) {
            if(error){
                   reject( error);
            }
            if(result){
              console.log(result);
                    resolve(result);
            }
          });
        });
}
app.get('/upload', function ( req, res) {
var url = req.params.url || 'http://artwork-cdn.7static.com/static/img/sleeveart/00/055/149/0005514991_800.jpg';
///'http://artwork-cdn.7static.com/static/img/artistimages/00/000/113/0000011319_300.jpg';
var public_id = req.params.publicid || 'Cyndi_Lauper_cover';
console.log(url, public_id);
// res.send({url:url, public_id:public_id}); 
  uploadImage(url,public_id)
  .then(function(data){
        res.send(data);   
  }).catch(function(error){
      console.log('error:', error);
      res.send(error);
  })
});
var getImagesByTags = function(tags){
return  new Promise(function (resolve, reject) {
           cloudinary.v2.api.resources_by_tag(tags,{max_results:100, tags:true}, 
           function(error, result){
             if(error){
               reject(error);
             }
             if(result){
               console.log(result);
               resolve(result);
             }
           });
        });
}
// Get tracks by releaseID: 
var getTracks = function(releaseid) {  
  return new Promise(function (resolve, reject) {
        releases.getTracks({ releaseid: releaseid }, function(err, data) {
        if(err){
          console.log(err);
            reject(err)
        }
        if(data){
          resolve(data);
        } 
      });
  })
}
// Get tracks by releaseID: 7456808
app.get('/tracks/:releaseid', function ( req, res ) {
  const releaseid = req.params.releaseid || '7026306';
    console.log(releaseid);
    getTracks(releaseid)
    .then(function(data){
      console.log(JSON.stringify(data,null,5));
      res.send(data); 
    }).catch(function(error){
       res.send(error);
    });
});
app.get('/', function (req, res) {
  const html = `<a href="https://cloudinary.gitbook.io/canadian-music-week-hackathon-guide">Hackathon Guide<a>`;
    res.send(html); 
});
module.exports = Webtask.fromExpress(app);
```

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


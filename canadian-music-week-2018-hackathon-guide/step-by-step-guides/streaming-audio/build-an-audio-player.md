# Build an Audio Player

by Eric Portis

Now that we have [an endpoint](build-a-music-discovery-api.md), let’s use it to build a streaming audio player.

## Step 1: Get the data

First, let’s get data from our endpoint, and into a browser.

{% embed data="{\"url\":\"https://codepen.io/eeeps/pen/eMJNLQ/\",\"type\":\"rich\",\"title\":\"Streaming Audio Player, Step 1: Get the data\",\"description\":\"...\",\"icon\":{\"type\":\"icon\",\"url\":\"https://codepen.io/favicons/favicon-192x192.png\",\"width\":192,\"height\":192,\"aspectRatio\":1},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://s3-us-west-2.amazonaws.com/i.cdpn.io/881825.eMJNLQ.small.f573d511-a329-41d9-a75f-4d6e907dc159.png\",\"width\":384,\"height\":225,\"aspectRatio\":0.5859375},\"embed\":{\"type\":\"app\",\"url\":\"https://codepen.io/eeeps/embed/preview/eMJNLQ?height=300&slug-hash=eMJNLQ&default-tabs=js,result&host=https://codepen.io&embed-version=2\",\"html\":\"<iframe src=\\"https://codepen.io/eeeps/embed/preview/eMJNLQ?height=300&amp;slug-hash=eMJNLQ&amp;default-tabs=js,result&amp;host=https://codepen.io&amp;embed-version=2\\" style=\\"border: 0; width: 100%; height: 300px;\\" allowfullscreen></iframe>\",\"height\":300,\"aspectRatio\":null}}" %}

## Step 2: Set up a bunch of boilerplate

Before we can do anything, we’ll need to initialize our video player stack.

For this tutorial, we’'ll be using [VideoJS](https://videojs.com), and its excellent [playlist plugin](https://github.com/brightcove/videojs-playlist) and [playlist-ui](https://github.com/brightcove/videojs-playlist-ui) picker.

Here’s all of the boilerplate we’ll need to load all of those components:

{% embed data="{\"url\":\"https://codepen.io/eeeps/pen/yKeNoK/\",\"type\":\"rich\",\"title\":\"Streaming Audio Player, Step 2: Set up a bunch of boilerplate\",\"description\":\"...\",\"icon\":{\"type\":\"icon\",\"url\":\"https://codepen.io/favicons/favicon-192x192.png\",\"width\":192,\"height\":192,\"aspectRatio\":1},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://s3-us-west-2.amazonaws.com/i.cdpn.io/881825.yKeNoK.small.6ebd91f3-4566-4019-9da3-f0689a22248e.png\",\"width\":384,\"height\":225,\"aspectRatio\":0.5859375},\"embed\":{\"type\":\"app\",\"url\":\"https://codepen.io/eeeps/embed/preview/yKeNoK?height=300&slug-hash=yKeNoK&default-tabs=html,result&host=https://codepen.io&embed-version=2\",\"html\":\"<iframe src=\\"https://codepen.io/eeeps/embed/preview/yKeNoK?height=300&amp;slug-hash=yKeNoK&amp;default-tabs=html,result&amp;host=https://codepen.io&amp;embed-version=2\\" style=\\"border: 0; width: 100%; height: 300px;\\" allowfullscreen></iframe>\",\"height\":300,\"aspectRatio\":null}}" %}

\(I’ll tuck all of that into our pen’s settings, for our next and last example.\)

## Step 3: Data + boilerplate = a player!

Our player expects a playlist array that looks [like this](https://github.com/brightcove/videojs-playlist-ui/blob/master/example.html#L48).

```javascript
[{
    name: 'A Cool Vid',
    duration: 48,
    sources: [
        { src: 'http://vjs.zencdn.net/v/oceans.mp4', type: 'video/mp4' },
        { src: 'http://vjs.zencdn.net/v/oceans.webm', type: 'video/webm' },
  ],
  poster: 'https://demo-res.cloudinary.com/sample.jpg'
},
{
    name: 'Another Cool Vid',
    duration: 42,
    sources: [
        { src: 'http://vjs.zencdn.net/v/oceans.mp4', type: 'video/mp4' },
        { src: 'http://vjs.zencdn.net/v/oceans.webm', type: 'video/webm' },
  ],
  poster: 'https://demo-res.cloudinary.com/sample.jpg'
}]
```

So, what we need to do is:

1. map the returned data from our endpoint, into an array that looks like that
2. feed it to our player using the `player.playlist()` method
3. build the playlist UI, using the `player.playlistUi()` method.

Like this:

{% embed data="{\"url\":\"https://codepen.io/eeeps/pen/OvyZmK/\",\"type\":\"rich\",\"title\":\"Streaming Audio Player, Step 3: Done!\",\"description\":\"...\",\"icon\":{\"type\":\"icon\",\"url\":\"https://codepen.io/favicons/favicon-192x192.png\",\"width\":192,\"height\":192,\"aspectRatio\":1},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://s3-us-west-2.amazonaws.com/i.cdpn.io/881825.OvyZmK.small.b27550fa-6816-4177-82a0-809159f9e1e2.png\",\"width\":384,\"height\":225,\"aspectRatio\":0.5859375},\"embed\":{\"type\":\"app\",\"url\":\"https://codepen.io/eeeps/embed/preview/OvyZmK?height=300&slug-hash=OvyZmK&default-tabs=js,result&host=https://codepen.io&embed-version=2\",\"html\":\"<iframe src=\\"https://codepen.io/eeeps/embed/preview/OvyZmK?height=300&amp;slug-hash=OvyZmK&amp;default-tabs=js,result&amp;host=https://codepen.io&amp;embed-version=2\\" style=\\"border: 0; width: 100%; height: 300px;\\" allowfullscreen></iframe>\",\"height\":300,\"aspectRatio\":null}}" %}

🎉 Done!


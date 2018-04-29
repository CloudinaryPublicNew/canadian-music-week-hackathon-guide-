# Build an Audio Player

by Eric Portis

Now that we have [an endpoint](build-a-music-discovery-api.md), let’s use it to build a streaming audio player.

## Step 1: Get the data

First, let’s get data from our endpoint, and into a browser.

See the Pen [Streaming Audio Player, Step 1: Get the data](https://codepen.io/eeeps/pen/eMJNLQ/) by Eric Portis \([@eeeps](https://codepen.io/eeeps)\) on [CodePen](https://codepen.io).

## Step 2: Set up a bunch of boilerplate

Before we can do anything, we’ll need to initialize our video player stack.

For this tutorial, we’'ll be using [VideoJS](https://videojs.com), and its excellent [playlist plugin](https://github.com/brightcove/videojs-playlist) and [playlist-ui](https://github.com/brightcove/videojs-playlist-ui) picker.

Here’s all of the boilerplate we’ll need to load all of those components:

See the Pen [Streaming Audio Player, Step 2: Set up a bunch of boilerplate](https://codepen.io/eeeps/pen/yKeNoK/) by Eric Portis \([@eeeps](https://codepen.io/eeeps)\) on [CodePen](https://codepen.io).

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
}
]
```

So, what we need to do is:

1. map the returned data from our endpoint, into an array that looks like that
2. feed it to our player using the `player.playlist()` method
3. build the playlist UI, using the `player.playlistUi()` method.

Like this:

See the Pen [Streaming Audio Player, Step 3: Done!](https://codepen.io/eeeps/pen/OvyZmK/) by Eric Portis \([@eeeps](https://codepen.io/eeeps)\) on [CodePen](https://codepen.io).

🎉 Done!


# How to automatically download YouTube videos from channels you like

## Disclaimer

1. I'm not an expert, you'll probably see a thousand ways to do it better, this is just what I came up with and it works fine for me.
2. If you like a YouTuber, support their job by watching on YouTube website, so it can give views to them.
3. Please ignore my grammar, English is not my main language.

## What you'll need

1. Yt-dlp (https://github.com/yt-dlp/yt-dlp)
2. Newsboat (https://pkgs.org/download/newsboat)

## Setup

#### 1. YouTube File

First, you need a file named `urls` with all the YouTube RSS URLs you want to download videos from. YouTube channel URLs use the following format:

`https://www.youtube.com/feeds/videos.xml?channel_id={channel_ID}`

There are several ways to get a channel ID, but I think the easiest one is to find the channel you want on https://yewtu.be website, it will show the Channel ID for you easily. Example for GNOME YouTube channel:

https://yewtu.be/channel/UCH9zoXlRn8iiaiSmuazAFZw

At the end, you'll have a file that looks something like this (one URL per line):

```
https://www.youtube.com/feeds/videos.xml?channel_id=UC0e3QhIYukixgh5VVpKHH9Q
https://www.youtube.com/feeds/videos.xml?channel_id=UC2eYFnH61tmytImy1mTYvhA
```

### 2. Newsboat

Newsboat is a RSS program that runs on the terminal, and it will be used to grab the URL of new videos from your channels. Install it for your Linux distro, and run the command `newsboat` once. It will tell you that you don't have any file with RSS URLs.

# How to automatically download YouTube videos from channels you like

## Disclaimer

1. I'm not an expert, you'll probably see a thousand ways to do it better, this is just what I came up with and it works fine for me.
2. If you like a YouTuber, support their job by watching on YouTube website, so it can give views to them.
3. Please ignore my grammar, English is not my main language.

## What you'll need

1. Yt-dlp (https://github.com/yt-dlp/yt-dlp)
2. Newsboat (https://pkgs.org/download/newsboat)
3. Bash Script

## Setup

### 1. YouTube File

First, you need a file named `urls` with all the YouTube RSS URLs you want to download videos from. YouTube channel URLs use the following format:

`https://www.youtube.com/feeds/videos.xml?channel_id={channel_ID}`

There are several ways to get a channel ID, but I think the easiest one is to find the channel you want on https://yewtu.be website, it will show the Channel ID for you easily.

At the end, you'll have a file that looks something like this (one URL per line):

```
https://www.youtube.com/feeds/videos.xml?channel_id=UC0e3QhIYukixgh5VVpKHH9Q
https://www.youtube.com/feeds/videos.xml?channel_id=UC2eYFnH61tmytImy1mTYvhA
```

### 2. Newsboat

Newsboat is a RSS program that runs on the terminal, and it will be used to grab the URL of new videos from your channels. Install it for your Linux distro, and run the command `newsboat` once. It will tell you that you don't have any files with RSS URLs. Now follow these steps below:

1. Grab that file you created and move it to `~/.newsboat/`.
2. Run the `newsboat` command again to open the application.
3. Press `[Shift] + [R]` to update all YouTube channels.
4. Press `[Shift] + [C]` to mark all as read and confirm by pressing `[Y]`.
5. Press `[Q]` to close.

We did this just to mark all YouTube videos as watched so we can start fresh from here, otherwise you'll download around 30 past videos from each channel from that file.

### 3. Bash Script

Finally, let's make a bash script. Create it and give a name like `youtube.sh` and set it as executable with `chmod +x youtube.sh`. This bash script will do the following:

1. Update Yt-dlp repo on your machine (there are updates almost daily)
2. Get new video URLs from newsboat app
3. Put all these URLs in a TXT file (to make it easier) and use Yt-dlp to read URLs from that file and download all videos
4. If a video gives an error for some reason (usually happens when a video URL is ready, but the video is scheduled to air later), the script will tell newsboat to mark that video as unread, so you can try to download it later again (it will also print the video URLs with problems at the end).

Here's the simple bash script I use (feel free to adapt it for your needs and make it better).

1. Modify the folder locations to match yours.
2. I have a Full HD monitor, so I set videos to download at max resolution 1080p 60fps, change it as needed.

```
#!/bin/bash

# ===================================
# SETTINGS
# ===================================

DATABASE="/home/$USER/.newsboat/cache.db"

QUERY="select url from rss_item where unread=1 order by url asc"

# ===================================
# GET VIDEOS
# ===================================

# run newsboat and update feeds
newsboat -x reload print-unread

# get urls of new videos
sqlite3 "$DATABASE" "$QUERY" > /home/$USER/Downloads/links.txt

echo
echo "============================================="
echo "                 NEW VIDEOS                  "
echo "============================================="
echo
sqlite3 "$DATABASE" "$QUERY"
echo
echo "============================================="
echo

# ===================================
# UPDATE YT-DLP
# ===================================

cd /home/$USER/Downloads/GIT/yt-dlp;
git pull;

# ===================================
# SORT AND REMOVE DUPLICATES
# ===================================

sort -ru -o /home/$USER/Downloads/links.txt ~/Downloads/links.txt;

# ===================================
# DOWNLOAD VIDEOS
# ===================================

ERRORS=()

for url in $(cat /home/$USER/Downloads/links.txt); do
	/home/$USER/Downloads/GIT/yt-dlp/yt-dlp.sh --embed-thumbnail --embed-subs --sub-langs "en" -f 'bestvideo[height<=1080][fps<=60]+bestaudio/best[height<=1080][fps<=60]' -o "/home/$USER/Downloads/Videos/%(uploader)s_-_%(title)s.%(ext)s" --no-abort-on-error --restrict-filenames -R 10 -N 10 --fragment-retries 10 --force-ipv4 $url;

	# check for errors
	if [ $? -eq 1 ]; then
		ERRORS+=($url);
		sqlite3 "$DATABASE" "update rss_item set unread=1 where url='$url'";
	else
		sqlite3 "$DATABASE" "update rss_item set unread=0 where url='$url'";
	fi;

done;

# print errors

echo
echo "============================================="
echo "             VIDEOS WITH ERROR               "
echo "============================================="
echo
printf '%s\n' "${ERRORS[@]}"
echo

```

### 4. Cron

Now the last part is to put this script on cron so it will run automatically in the background. In my case, I set it to run every 1 hour. Mine looks like this:

`@hourly /home/user/Downloads/youtube.sh`

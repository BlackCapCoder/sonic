Added ability to read input from STDIN / write output to STDOUT using underscore.


Downloading, speeding up, and playing audio from youtube in mpv can be done in a single pipeline like so:
```bash
SPEED="2.5"

youtube-dl -f bestaudio "$1" --quiet -o - \
  | ffmpeg -loglevel panic -i - -f wav - \
  | sox -t wav - -t wav - \
  | sonic -c -s $SPEED _ _ \
  | mpv -
```

Downloading, speeding up, and playing a video from youtube in mpv can be done like so:
```bash
SPEED="2.5"

urls=$(youtube-dl -f 'bestaudio,bestvideo[ext=mp4]/bestvideo[height<=1800]/bestvideo' "$1" --quiet --get-url)
audio=$(echo "$urls" | sed '1q;d')
video=$(echo "$urls" | sed '2q;d')

ffmpeg -loglevel panic -i "$audio" -f wav - \
  | sox -t wav - -t wav - \
  | sonic -c -s $SPEED _ _ \
  | ffmpeg -i - \
           -i "$video" \
           -loglevel panic \
           -filter:v "setpts=$(lua -e "print(1/$SPEED)")*PTS" \
           -vsync vfr -r 60 \
           -c:v libx264 -crf 22 -preset ultrafast -tune zerolatency \
           -f avi - \
  | mpv --cache-secs 60 -
```

The reason I pipe the output of ffmpeg to sox is that:
1) sonic cannot parse the output from ffmpeg
2) sox cannot read webm files, which is what `bestaudio` usually is

https://lists.ffmpeg.org/pipermail/ffmpeg-devel/2007-May/035617.html

https://video.stackexchange.com/questions/4563/how-can-i-crop-a-video-with-ffmpeg

https://stackoverflow.com/questions/3937387/rotating-videos-with-ffmpeg


yt-dlp "https://www.youtube.com/watch?v=7Gh7yJ4sj8k" -f 'bestvideo[height<=360]' --external-downloader ffmpeg --external-downloader-args "-ss 00:00:00 -to 00:00:55" -o "%(title)s_method2.%(ext)s"


ffmpeg -i water-full.webm -vf "crop=90:360:300:0,transpose=2,scale=trunc(oh*a/2)*2:64" -pix_fmt gbrp water.webm

ffmpeg -i riverani-full.webm -vf "crop=90:360:150:0,transpose=2,scale=trun
c(oh*a/2)*2:64" -pix_fmt gbrp riverani.webm

scp riverani.webm simon@192.168.50.168:/home/simon/Videos/riverani.webm


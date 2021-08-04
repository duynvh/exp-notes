# Một vài command giúp xử lý audio

1. Convert to wav from data without header
```
ffmpeg -f s16le -ar 8000 -i input.wav output.wav
```

2. Show info of media
```
mediainfo audio.wav
```
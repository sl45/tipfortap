# Anything Video
ffmpeg, quicktime, or supporting codecs.
_____

## QuickTime
Bringing back [QuickTime 7 Pro](http://techdechdude.blogspot.com/2010/06/or-any-other-version-entering-your.html) by using the following registration info. 

Macs:<br>
Mac Users:<br>
Registered To: Pablo/nop<br>
Registration code:<br>
SMH2-6F4K-8SK9-ST5W-QKU3<br>

For Macs:<br>
Register Name:Pablo/nop<br>
Register Code:SMH2-6F4K-8SK9-ST5W-QKU3<br>

Others:<br>
Register Name = In Memory of Abysm<br>
Register Code = YBN7Y-9K77Y-F67B5-PNCCQ<br>

Register Name: www.macmofo.com<br>
Register Code: 3DX76-9KJ4F-CBQL9-BJBKH<br>

Registered To: Apple Sales<br>
Registration Code: TTVN-B66S-28V6-X84D-EBWN<br>

Registered to: apple employee<br>
Registration Code:42VB-ZLGG-QF99-2NK6-CKEC<br>

## ffmpeg
[ffmpeg](https://ffmpeg.org/documentation.html) provides fast audio and video conversion, and can be used to extract av materials for various uses. A few useful ffmpeg recipes I used.

### changing the audio
1. Merging audio and video <br>
 Simply copying audio and video streams without re-encoding.
 
       ffmpeg -i (video file) -i (audio file) -c copy (outputfile)

2. Replacing audio stream with re-encoding <br>
 If the input video file has an audio track which needs to be replaced by another audio file, ffmpeg will take the video stream from the first input file combined with the audio stream from the second input file. In this case, when the audio file is longer than the video, the video will stop at the last frame as a still image with audio running till the end.

       ffmpeg -i (video file) -i (audio file) -c:v libx264 -preset veryslow 
       -qp 18 -pix_fmt yuv240p -c:a aac -strict experimental -map 0:v:0 -map 1:a:0 (outputfile.mp4)
    
    *``-c:v libx264`` use the H.264 video codec for video stream. ``-preset`` value for H.264 are "veryslow", "slow", "medium", "fast", and "veryfast". Slower encoding gives better compression rate. ``-qp 18`` quality parameter of 18 means a visually lossless compression. ``-pix_fmt yuv 240p`` use YUV colour space with 4:2:0 chroma subsampling.* <br>
    
    *``-c:a aac`` re-encode audio with [AAC](https://trac.ffmpeg.org/wiki/Encode/AAC). ``-strict experimental`` is the second highest-quality AAC encoder. The [map](https://trac.ffmpeg.org/wiki/Map) option is to indicate which stream to be used for the output file.*
  
3. Cutting audio to fit the length of the video <br>

       ffmpeg -i (video file) -i (audio file) -shortest (outputfile.mov)
  
    *``-shortest`` takes the shortest file to be the final length for the output file.*
    
### changing the video
1. Cutting up the video

        ffmpeg -ss 00:00:30.0 -i input.wmv -c copy -t 00:00:10.0 output.wmv
        ffmpeg -ss 30 -i input.wmv -c copy -t 10 output.wmv

    ``-ss`` option to specify a start timestamp, and the ``-t`` option to specify the encoding duration. The timestamps need to be in HH:MM:SS.xxx format or in seconds (s.msec)

2. Converting MXF file

        ffmpeg -i input.MXF -acodec libmp3lame -vcodec libx264 -s 4096x2160 -b:v 5000k -t 2 07-4k.mp4 
        
    Keeping the size at 4K, to work on anything else but a "film", seems quite of an overkill. It makes the encoding very slow, although the bit rate might be the culpit, as we are still at 36492 kb/s, but also size. More recipes can be found [here](http://oooooooooo.io/index.php/Convert_MXF_files_with_ffmpeg).

### frame-based checksums
A frame-based md5 checksum report can be generated with this command: 

       ffmpeg -i MOVIE.mov -f framemd5 MOVIE.framemd5

### batch script
A batch script to batch convert files with ffmpeg:

       dir/b/s "\\folder\*.mkv" >mkvlist.txt
       for /F "delims=;" %%F in (mkvlist.txt) do ffmpeg.exe  -i "%%F" (corresponding cmd) "%%~dF%%~pF%%~nF.mov"
       del mkvlist.txt
   
   *This batch script would take all the mkv files under the indicated folder and subfolders, then save the newly re-encoded file under the same folder with the same file name.*
   
       for %%a in ("\*.mov) do ffmpeg.exe -i %%a -c:v ffv1 -level 3 -coder 1 -context 1 -g 1 -slices 24 -slicecrc 1 -c:a copy "\filepath\%%~na.mkv"
       
   *This batch script would take all the .mov files under the indicated folder (exclude anything under subfolders) and save the newly made ffv1 mkv files to the other indicated folder with the same file name. More details about the ffv1 vs Matroska option can be found [here](https://avpres.net/FFmpeg/im_MKV.html).*


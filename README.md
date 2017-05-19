# Tip-for-Tap
Find some tips before tapping.
_____
For artists and institutions with a small budget, digital preservation can appear to be a daunting matter. Instead of reading helluva manual of instructions, this post attempts to compile workflows and useful commands that can provide basic care to validate the integrity of digital asset.

### Open source tools from library and archive community
Bitcurator, Virtual Box and a write blocker will be needed in order to facilitate the following workflows: 1) create forensic disk images, 2) analyze files type and the associated systems, 3) extract metadata, 4) identify sensitive information before public access, 5) locate and remove duplicate files.
- [Bitcurator](https://wiki.bitcurator.net/index.php?title=Main_Page) environment includes a set of digital forensic and data analysis tools to process digital collections. 
- Bitcurator environment requires [Virtual Box](https://www.virtualbox.org/wiki/Downloads) to be installed. It is a virtualization tool that can run on Windows, Linux, Macintosh, and Solaris.
- [Forensic ComboDock FCDv5.5](http://www.cru-inc.com/products/wiebetech/forensic-combodock-v5-5/) is a professional hard drive write blocker, supporting USB 3.0, USB 2.0, eSATA, and FireWire 800 connections.

Suggested workflows here is only for audiovisual materials on external carrier <br>
Guymager or ddrescue -> ffmpeg -> mediainfo -> bagit-python <br>

## ddrescue 
To capture disc image, both Guymager and ddrescue can be found in BitCurator environment. Since BitCurator wiki has provided thorough tutorials for [Guymager](https://wiki.bitcurator.net/index.php?title=Creating_a_Disk_Image_Using_Guymager), this post will only list useful syntax for [ddrescue](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html). In additon, ddrescue is better for copying recordable optical media because:
> data loss develops slowly with read errors growing from the outer media region towards the inside. Two (or more) copies of the same disc can be used for data recovery by employing ddrescue.

Find the location of the external carrier and take a note.

    diskutil list

1. if the locatoin data is on a CD-ROM at /dev/cdrom <br>
       
       ddrescue -n -b2048 /dev/cdrom (cdimage) (mapfile)
    *``-n`` skip the scraping phase. ``-b`` show sector size of input device in bytes.*
       
       ddrescue -d -r1 -b2048 /dev/cdrom (cdimage) (mapfile)
    *``-d`` use direct disc access to read from input file. ``-r`` stop after the given number of retry passes.*

2. if the location data is on a CD-ROM from 2 copies at /dev/cdrom.

       ddrescue -n -b2048 /dev/cdrom cdimage mapfile 
       ddrescue -d -b2048 /dev/cdrom cdimage mapfile
       (insert second copy)
       ddrescue -d -r1 -b2048 /dev/cdrom cdimage mapfile

## ffmpeg
[ffmpeg](https://ffmpeg.org/documentation.html) provides fast audio and video conversion, and can be used to extract av materials for various uses.

1. Ripping DVD <br>
 Video on DVD is usually divided into several .vob files which can be located under "VIDEO_TS" folder. ffmpeg will re-encode the output video with H.264 codec as .mkv file.<br>

       ffmpeg -i (.vob file) -map 0:v -map 0:a -c:v libx264 -crf 18 -vf yadif -c:a flac (outputfile.mkv)
    *``-map 0:v`` copy/transcode all video streams. ``-map 0:a`` copy/transcode all audio streams. `` -c:v libx264`` use libx264 codec. ``-crf 18`` use "Constant Rate Factor" value 18. ``-vf yadif`` use YADIF deinterlacing.``-c:a flac`` use FLAC (Free Loseless Audio Codec) for audio streams. Instead of ``-crt 18``, ``-qp 18`` can also provide visually lossless result.* 
    > The range of the quantiser scale for crt and qp is from 0 to 51, where 0 is lossless, approximately 18 is "visually lossless", 23 is the default value and 51 is worst possible. Most of the non-FFmpeg-based players cannot decode H.264 files having lossless content.
     
2. Merging audio and video <br>
 Simply copying audio and video streams without re-encoding.
 
       ffmpeg -i (video file) -i (audio file) -c copy (outputfile)

3. Replacing audio stream with re-encoding <br>
 If the input video file has audio track which needs to be replaced by another audio file, ffmpeg will take the video stream from the first input file combined with the audio stream from the second input file. In this case, when the audio file is longer than the video, the video will stop at the last frame as still image with audio running till the end.

       ffmpeg -i (video file) -i (audio file) -c:v libx264 -preset veryslow 
       -qp 18 -pix_fmt yuv240p -c:a aac -strict experimental -map 0:v:0 -map 1:a:0 (outputfile.mp4)
    
    *``-c:v libx264`` use the H.264 video codec for video stream. ``-preset`` value for H.264 are "veryslow", "slow", "medium", "fast", and "veryfast". Slower encoding gives better compression rate. ``-qp 18`` quality parameter of 18 means a visually lossless compression. ``-pix_fmt yuv 240p`` use YUV colour space with 4:2:0 chroma subsampling.* <br>
    
    *``-c:a aac`` re-encode audio with [AAC](https://trac.ffmpeg.org/wiki/Encode/AAC). ``-strict experimental`` is the second highest-quality AAC encoder. The [map](https://trac.ffmpeg.org/wiki/Map) option is to indicate which stream to be used for the output file.*
  
4. Cutting audio to fit the length of the video <br>

       ffmpeg -i (video file) -i (audio file) -shortest (outputfile.mov)
  
    *``-shortest`` takes the shortest file to be the final length for the output file.*

## MediaInfo
[MediaInfo](https://mediaarea.net/en/MediaInfo) provides detailed descriptions for AV materials. This command will export technical characteristics of the media asset as a separate XML file. 
    
    mediainfo (inputfile) --output=XML --logfile=(outputfile.log)

## Bagit Python
[Bagit-python](https://github.com/LibraryOfCongress/bagit-python) can use python library to generate bagit style package. Bagit, developed by Library of Congress, is a
> hierarchical file packaging format designed to support disk-based storage and network transfer of arbitrary digital content. A "bag" consists of a "payload" (the arbitrary content) and "tags", which are metadata files intended to document the storage and transfer of the bag. A required tag file contains a manifest listing every file in the payload together with its corresponding checksum.
 
After placing all the digital asset under the designated folder with metadata, ``bagit.py`` can create a digital package with md5 checksum.
 
    bagit.py --process 4 (inputfile)

Once the bag is moved/copied to another location, ``--validate`` can examine the new bag and detect errors.  
    
    bagit.py --validate (outputfile)
    
At this stage, [rync](https://wiki.archlinux.org/index.php/rsync) command can come in handy to transfer one package remotely to a storage location. It can be used as an alternative option for the ``cp`` command, especially for copying larger files.

    rync -a --progress --remove-source-files (inputfile) (destination folder)

*``-a`` means all files are archived and their characteristics are preserved. ``--progress`` show progress during transfer. ``--remove-surce-files`` delete the source files after the transfer is complete.*

## Related readings and resources
- [Archiving and distribution of CD-ROM artworks, a study of the Emulation as a Service (EaaS) tool and other proposals](http://li-ma.nl/site/sites/default/files/201611_DE_Houdbaar_Final_report_CD-ROM_Archiving_DEF.pdf), The Hague, 1 November 2016. <br>
- [FFmpeg Cookbook for Archivists](https://avpres.net/FFmpeg/)<br>
- [Correcting for audio/video sync issues with the ffmpeg program’s ITSOFFSET switch](https://wjwoodrow.wordpress.com/2013/02/04/correcting-for-audiovideo-sync-issues-with-the-ffmpeg-programs-itsoffset-switch/)

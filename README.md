# Tip-for-Tap
Find some tips before tapping.
_____
For artists and institutions with a small budget, digital preservation can appear to be a daunting matter. Instead of reading a helluva manual of instructions, this post attempts to compile workflows and useful commands that can provide basic care to validate the integrity of digital assets.

### Open source tools from library and archive community
Bitcurator, Virtual Box and a write blocker will be needed in order to facilitate the following workflows: 1) create forensic disk images, 2) analyze files type and the associated systems, 3) extract metadata, 4) identify sensitive information before public access, 5) locate and remove duplicate files.
- [BitCurator](https://wiki.bitcurator.net/index.php?title=Main_Page) environment includes a set of digital forensic and data analysis tools to process digital collections. 
- Bitcurator environment requires [Virtual Box](https://www.virtualbox.org/wiki/Downloads) to be installed. It is a virtualization tool that can run on Windows, Linux, Macintosh, and Solaris.
- [Forensic ComboDock FCDv5.5](http://www.cru-inc.com/products/wiebetech/forensic-combodock-v5-5/) is a professional hard drive write blocker, supporting USB 3.0, USB 2.0, eSATA, and FireWire 800 connections.

Suggested workflows here is only for audiovisual materials on external carrier <br>
Guymager or ddrescue -> ffmpeg -> mediainfo -> bagit-python <br>

## ddrescue 
To capture a disc image, both Guymager and ddrescue can be found in the BitCurator environment. Since BitCurator wiki has provided thorough tutorials for [Guymager](https://wiki.bitcurator.net/index.php?title=Creating_a_Disk_Image_Using_Guymager), this post will only list useful syntax for [ddrescue](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html). In addition, ddrescue is better for copying recordable optical media because:
> data loss develops slowly with read errors growing from the outer media region towards the inside. Two (or more) copies of the same disc can be used for data recovery by employing ddrescue.

Using either commands to find the location of the external carrier and take a note.

    diskutil list
    df -h

Using the following commands to find out about block size.

    diskutil info /dev/disk2 | grep “Block Size”
    stat -f %k .

1. if the location data is on a CD-ROM at /dev/disk2 <br>
       
       ddrescue -n -r3 -b 2048 /dev/disk2 cdimage.iso mapfile.log
    *``-n`` skip the scraping phase. ``-b`` sector size of input device in bytes (usually 512 for hard discs and 3.5" floppies, 1024 for 5.25" floppies, and 2048 for cdroms). More about [disk sector](https://en.wikipedia.org/wiki/Disk_sector). ``-r`` stop after the given number of retry passes.*

2. if the location data is on a CD-ROM from 2 copies at /dev/disk2.

       ddrescue -n -b 2048 /dev/cdrom cdimage.iso mapfile.log 
       ddrescue -d -b 2048 /dev/cdrom cdimage.iso mapfile.log
       (insert the second copy)
       ddrescue -d -r1 -b 2048 /dev/cdrom cdimage.iso mapfile.log

## ffmpeg
[ffmpeg](https://ffmpeg.org/documentation.html) provides fast audio and video conversion, and can be used to extract av materials for various uses. More ffmpeg recipes can be found under the video folder.

Video on DVD is usually divided into several .vob files which can be located under the "VIDEO_TS" folder. ffmpeg can be used to re-encode the output video with H.264 codec as .mp4 file.<br>

    ffmpeg -i (.vob file) -map 0:v -map 0:a -c:v libx264 -crf 18 -vf yadif -c:a aac -strict experimental /outputfile.mkv
   
   ``-map 0:v`` copy/transcode all video streams. ``-map 0:a`` copy/transcode all audio streams. `` -c:v libx264`` use libx264 codec. ``-crf 18`` use "Constant Rate Factor" value 18. ``-vf yadif`` use YADIF deinterlacing.``-c:a flac`` use FLAC (Free Lossless Audio Codec) for audio streams. Instead of ``-crf 18``, ``-qp 18`` can also provide visually lossless result. The range of the quantiser scale for crt and qp is from 0 to 51, where 0 is lossless, approximately 18 is "visually lossless", 23 is the default value and 51 is worst possible. 

If larger than 1 GB, it will be splitted into several vobs. This recipe can be useful to combine multiple .vob into one mp4.<br> 

    ffmpeg -i "concat:/VIDEO_TS/VTS_01_1.VOB|/VIDEO_TS/VTS_01_2.VOB" -c:v copy -c:a copy /outputfile.mp4

## MediaInfo
[MediaInfo](https://mediaarea.net/en/MediaInfo) provides detailed descriptions for AV materials. This command will export technical characteristics of the media asset as a separate XML file. 
    
    mediainfo (inputfile) --output=XML --logfile=(outputfile.log)

This script can batch generate xml files for videos inside a folder. It needs to be saved as .sh for Mac OS.
    
    cd  /folder path
    for f in *
    do
    mediainfo "$f" --output=XML > /folder path/"${f%.mov}".xml
    done

## DFXML & md5deep
Digital Forensic XML of the folder content can be generated via [md5deep](http://md5deep.sourceforge.net/md5deep.html#toc). The following command can be useful to generate dfxml and md5 checksum for the destination folder.

    cd /folder path
    md5deep -rl -e -d . >/folder path/dfxml.txt
   
*``-r`` enables recursive mode. ``-l`` uses relative paths. ``-e`` displays progress and estimate time. ``-d`` outputs in dfxml format. ``.`` recursive starting at the current folder. ``>`` redirects output to the indicated file.*

To compare content between two folders and verify md5 checksums, this command can come in handy:

    md5deep -rx checksumlist.md5 /folder path
    md5deep -rm checksumlist.md5 /folder path
    
*``-x`` negative matching mode, will display the differences. ``-m`` matching mode.*

To batch process asset on folder-level with folder-level dfxml and associated checksums of the files:

    dir /b /ad "/MainFolderPath" >folderlist.txt
    for /f "delims=" %%i in (folderlist.txt) do "md5deep64" -rl -e -d "MainFolderPath\%%i" > "MainFolderPath\%%i\%%~ni.txt"
    del folderlist.txt
    
*This batch script will process every subfolder under the main folder individually while generating dfxml to describe content of each subfolder. The dfxml .txt file will then be saved under the associated subfolder under the same name as the subfolder*

## Bagit Python
[Bagit-python](https://github.com/LibraryOfCongress/bagit-python) can use the python library to generate a bagit style package. Bagit, developed by Library of Congress, is a
> hierarchical file packaging format designed to support disk-based storage and network transfer of arbitrary digital content. A "bag" consists of a "payload" (the arbitrary content) and "tags", which are metadata files intended to document the storage and transfer of the bag. A required tag file contains a manifest listing every file in the payload together with its corresponding checksum.
 
After placing all the digital assets under the designated folder with metadata, ``bagit.py`` can create a digital package with sha256 and sha512 checksum.
 
    bagit.py --processes 4 /input folder path

Once the bag is moved/copied to another location, ``--validate`` can examine the new bag and detect errors.  
    
    bagit.py --validate /folder path

## .DS_Store & hidden files
.DS_Store file and other hidden files on Mac OS systems can often cause issues during bagit validation. Run the command to prevent the creation of .DS_Store file.

    defaults write com.apple.desktopservices DSDontWriteNetworkStores -boolean true

Run to check if it works. If it does, it should return “true” or “1”

    defaults read com.apple.desktopservices DSDontWriteNetworkStores

Remove .DS_Store file

    cd /path to volume you want to clean
    find . -name '.DS_Store' -type f -delete

Or, remove other hidden files

    find . -type f -name '._*' -delete

## rsync 
At this stage, [rsync](https://wiki.archlinux.org/index.php/rsync) command can come in handy to transfer one package remotely to a storage location. It can be used as an alternative option for the ``cp`` command, especially for copying large files.

    rsync -a --progress --remove-source-files (inputfile) (destination folder)

*``-a`` means all files are archived and their characteristics are preserved. ``--progress`` show progress during transfer. ``--remove-surce-files`` delete the source files after the transfer is complete.*

It can also be used to compare items in two folders

    rsync -rcnv (/folder1) (/folder2)

*``-r`` will recurse into the directories. ``-c`` compares file checksum. ``-n`` will do a "dry run" and make no changes. ``-v`` prints the output verbosely.*

To [schedule a rsync](https://www.marksanborn.net/howto/use-rsync-for-daily-weekly-and-full-monthly-backups/), use this command.

## Related readings and resources
- [Archiving and distribution of CD-ROM artworks, a study of the Emulation as a Service (EaaS) tool and other proposals](http://li-ma.nl/site/sites/default/files/201611_DE_Houdbaar_Final_report_CD-ROM_Archiving_DEF.pdf), The Hague, 1 November 2016. <br>
- [FFmpeg Cookbook for Archivists](https://avpres.net/FFmpeg/)<br>
- [Correcting for audio/video sync issues with the ffmpeg program’s ITSOFFSET switch](https://wjwoodrow.wordpress.com/2013/02/04/correcting-for-audiovideo-sync-issues-with-the-ffmpeg-programs-itsoffset-switch/)
- [File Signature Table](https://www.garykessler.net/library/file_sigs.html)
- [Sustainability of Digital Formats: Planning for Library of Congress Collections](https://www.loc.gov/preservation/digital/formats/fdd/descriptions.shtml)
- [What is DVD?](https://www.videohelp.com/dvd#struct)
- [NYPL Media Preservation Documentation Portal](https://nypl.github.io/ami-preservation/)

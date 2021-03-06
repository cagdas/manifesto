[![Go Report Card](https://goreportcard.com/badge/github.com/gitfu/manifesto)](https://goreportcard.com/report/github.com/gitfu/manifesto)

# Manifesto
Manifesto is an HLS tool for creating multiple variants, a master.m3u8 file, and converting 608 captions to segmented webvtt subtitles via ffmpeg.

## ``` Setup ```

#### ```Required``` 
* Go 
* Ffmpeg

#### ```Install go```
      https://golang.org/doc/install

#### ```Set your Environment```
```
mkdir  ~/go
export GOPATH=~/go
```
#### ``` Add one library ```
```
go get -u github.com/logrusorgru/aurora
```

#### ```Install ffmpeg with libx264 support```


#### ```Git clone the repo ```
```
git clone https://github.com/gitfu/manifesto
cd manifesto
go build manifesto.go
```

## ``` How It Works ```

Manifesto transcodes and segments video into multiple variants and creates the master.m3u8 file. 
608 Closed captions are extracted and converted to webvtt segment files.


## ``` Quick Start```

* ``` cd ~/manifesto ```
* ``` ./manifesto -i vid.ts ```

This will create the following directory structure and files 

```
vid:
hd720  low640  master.m3u8  med960  subs

vid/med960:
index0.ts  index1.ts  index2.ts  index3.ts  index4.ts  index.m3u8

vid/hd720:
index0.ts   index1.ts   index2.ts   index3.ts   index4.ts   index.m3u8

vid/low640:
index0.ts   index1.ts   index2.ts   index3.ts   index4.ts   index.m3u8

vid/subs:
index0.vtt  index1.vtt  index2.vtt  index3.vtt  index4.vtt  index_vtt.m3u8
```

* The default toplevel directory name is the video file name without the file extention.
* The variants are read from the hls.json file, variants can be added or removed as needed. 
* The command used to traanscode is specified in the cmd.template file, it can be modified. 

### ```Command line switches```
```

  -d string
    	override top level directory for hls files (optional)
  -i string
    	Video file to segment (either -i or -b is required)
  -j string
    	JSON file of variants (optional) (default "./hls.json")
  -s string
    	subtitle file to segment (optional)
  -t string
    	command template file (optional) (default "./cmd.template")
  -u string
    	url prefix to add to index.m3u8 path in master.m3u8 (optional)

```

## ``` Usage ```

```
./manifesto -i vid.mp4
```

* This is single mode a master.m3u8 and variants will be created in a new directory named vid. It will also attempt to extract 608 captions and convert them to segmented webvtt subtitles. 

###


```
./manifesto -i vid.mp4 -s sub.srt
```
* As above but instead of extracting 608 captions, sub.srt will be converted to a webvtt file and then segmented.

###

```
./manifesto -i vid.mp4 -s sub.srt -u http://example.com
```
* As above and also adds the url prefix to each variant listed in the m3u8 file. 



# How to Modify

* manifesto  works right of the box.I wanted it to be easy to use with minimal configuation needed. 
If you do feel the need to tune it a bit. 


##  ```Variants``` 


* Variant data is stored in the hls.json file. 
* Add or edit or remove as desired.

```
[
{"name": "med960", "aspect": "960x540", "framerate":"29.97","vbitrate": "2000","bufsize":"4000","abitrate": "96k"}
,{"name": "low640", "aspect": "640x360", "framerate":"29.97","vbitrate": "730","bufsize":"1460","abitrate": "64k"}
,{"name":"hd720","aspect": "1280x720", "framerate" :"29.97","vbitrate": "4500","bufsize":"9000","abitrate": "128k"}
]
```

## ```Modifying The Ffmpeg Command```


* cmd.template is the default ffmeg command template 
* NAME,ASPECT,FRAMERATE,VBITRATE,BUFSIZE,ABITRATE will be replaced with variant's values from hls.json.
* Newlines are stripped. 

 
 ## ```Values in the master.m3u8 file```
 
The values used in the master.m3u8 file are read an /or calculated for each variant 
as follows.

* Bandwidth ( bit_rate is parsed the mpegts container ) 
* Resolution ( width and height for the video resolution are parsed from the video stream) 
* Level     ( video level is parsed from the video stream )
* Profile  (video profile is parsed from the video stream, audio profile from the audio stream)
* Codecs  (Calculated as described here  https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/FrequentlyAskedQuestions/FrequentlyAskedQuestions.html )
```
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=%v,RESOLUTION=%s,CODECS=\"avc1.%v00%x%v\""
```


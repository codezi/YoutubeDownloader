Youtube Downloader
==================

Youtube video downloader

[![Build Status](https://travis-ci.org/masihyeganeh/YoutubeDownloader.svg?branch=master)](https://travis-ci.org/masihyeganeh/YoutubeDownloader)
[![Latest Stable Version](https://poser.pugx.org/masih/youtubedownloader/v/stable)](https://packagist.org/packages/masih/youtubedownloader)
[![Latest Unstable Version](https://poser.pugx.org/masih/youtubedownloader/v/unstable)](https://packagist.org/packages/masih/youtubedownloader)
[![Coverage Status](https://coveralls.io/repos/github/masihyeganeh/YoutubeDownloader/badge.svg?branch=master)](https://coveralls.io/github/masihyeganeh/YoutubeDownloader?branch=master)
[![Total Downloads](https://poser.pugx.org/masih/youtubedownloader/downloads)](https://packagist.org/packages/masih/youtubedownloader)
[![Dependency Status](https://www.versioneye.com/user/projects/5281d3db632baca88e000127/badge.svg)](https://www.versioneye.com/user/projects/5281d3db632baca88e000127)
[![License](https://poser.pugx.org/masih/youtubedownloader/license)](https://packagist.org/packages/masih/youtubedownloader)


CLI
---

If you are not a developer and just need this tool to download videos, only read this part. Next parts are for developers who want to use this package in their projects.
We assume that you already installed [PHP](http://php.net/) and [Composer](http://getcomposer.org/). To install the CLI, open terminal and write this command:
```
composer global require Masih/YoutubeDownloader
```
After installation you'll have a new `youtube` command. Just write `youtube --help` to get more information.  
  
_Please note that there are lots of Unicode characters in CLI, so it may not work as expected in Microsoft Windows._

Installation
------------

Youtube Downloader is PSR-0 compliant and can be installed using [composer](http://getcomposer.org/).  Simply add `masih/youtubedownloader` to your composer.json file. 
```json
    {
        "require": {
            "masih/youtubedownloader": "~2.6"
        }
    }
```

and run `composer update` command

Needs PHP 5.5 or newer. Tested with PHP 5.5, 5.6, 7.0, hhvm and nightly

Usage
-----

### Video or Playlist info

To get video or playlist information, you should instantiate `YoutubeDownloader` with video or playlist url or id.
for example for `http://youtube.com/watch?v=gmFn62dr0D8`, video id is `gmFn62dr0D8`.
or for `https://www.youtube.com/playlist?list=PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ`, playlist id is `PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ`.

```php
<?php
include 'vendor/autoload.php';

use Masih\YoutubeDownloader\YoutubeDownloader;

$youtube = new YoutubeDownloader('gmFn62dr0D8');

$result = $youtube->getInfo();

header('Content-type: application/json');
print json_encode($result);
```

the `getInfo()` method will call `getVideoInfo()` or `getPlaylistInfo()` according to url or id. there is a `response_type` field in result of each one, indicating type of response that can be `"playlist"` or `"video"`. 

`getVideoInfo()` method will return an object containing video title, images, url, captions and `itag` of all formats (full and adaptive), ...

`getPlaylistInfo()` method will return an object containing title, description, author, videos, views, likes, dislikes, ...

Both `getInfo(true)` and `getPlaylistInfo(true)` can get an optional boolean parameter to get result of `getVideoInfo()` for each videos in playlists.
  
Video urls can be in these formats (Vevo videos are also supported):
* `gmFn62dr0D8` (Video Id)
* `http://www.youtube.com/watch?v=gmFn62dr0D8`
* `http://www.youtube.com/embed/gmFn62dr0D8`
* `http://www.youtube.com/v/gmFn62dr0D8`
* `http://youtu.be/gmFn62dr0D8`
* `PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ` (Playlist Id)
* `https://www.youtube.com/watch?v=7gY_sq9uOmw&list=PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ`
* `https://www.youtube.com/playlist?list=PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ`
* `https://www.youtube.com/embed/videoseries?list=PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ`
   
Video formats are in two category; Full & adaptive
in Full formats, videos and sounds are muxed and are in one file. but in Adaptive formats, videos and sounds are in separated urls.
  
  
Each format has it's own `itag`. it's just an identifier.
You can get list of known `itags` and their descriptions by calling `getItags()` static method, or description of a single itag by calling `getItagInfo($itag)` with the itag number.
You can call `setDefaultItag($itag)` method with an itag number to download further videos in that format.

   
### Download video(s)

the `download()` method gets `itag` of a format and downloads it.
if no `itag` is passed, it will download highest quality of Full format.
For playlists, it will download all videos one by one. 

```php
<?php
set_time_limit(0); // Downloading a video may take a lot time

include 'vendor/autoload.php';

use Masih\YoutubeDownloader\YoutubeDownloader;

$youtube = new YoutubeDownloader('gmFn62dr0D8');

$youtube->download();
```

Second parameter of `download` method is a boolean named `$resume`. When it is `true`, it will continue downloading videos that are partially downloaded last time.
Thirt parameter of `download` method is a mixed named `$caption`. When it is `false` (default), it will not download caption (subtitle). When it is `null`, it will download caption in default language (english). Else, it should be a name of language. See Caption section below.
   
### Download progress

Download progress is available via `onProgress` parameter.
it's a closure and has four parameters `$downloadedBytes`, `$fileSize`, `$index` and `$count`.
`$index` and `$count` are useful for tracking download progress of playlist and both values are `1` for single videos.

```php
<?php
$youtube = new YoutubeDownloader('PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ');


$youtube->onProgress = function ($downloadedBytes, $fileSize, $index, $count) {
    if ($count > 1) echo '[' . $index . ' of ' . $count . ' videos] ';
	if ($fileSize > 0)
		echo 'Downloaded ' . $downloadedBytes . ' of ' . $fileSize . ' bytes [%' . number_format($downloadedBytes * 100 / $fileSize, 2) . '].' . "\n";
	else
		echo 'Downloading...'; // File size is unknown, so just keep downloading
};


$youtube->download();
```
   
### Download complete

Download complete event is available via `onComplete` parameter.
it's a closure and has four parameters `$downloadedBytes`, `$fileSize`, `$index` and `$count`.
`$index` and `$count` are useful for tracking download progress of playlist and both values are `1` for single videos.

```php
<?php
$youtube = new YoutubeDownloader('PLbjM1u8Yb9I0rK4hkPa9TWe4N_idJOnrJ');


$youtube->onComplete = function ($filePath, $fileSize, $index, $count) {
    if ($count > 1) echo '[' . $index . ' of ' . $count . ' videos] ';
	echo 'Downloading of ' . $fileSize . ' bytes has been completed. It is saved in ' . $filePath . "\n";
};


$youtube->download();
```
   
### Caption (Subtitle)

When you get video info, and array is also return named `captions` which it's keys are short language names and values are long names.
You should pass one of those short names as the third parameter of `download` method to download in that language.
Captions (subtitle) are downloaded in `srt` format. you can change this by passing one on `srt`, `sub` or `ass` to `setCaptionFormat($format)` method.  
You can call `setDefaultCaptionLanguage($language)` with a short language name to download further captions in that language.

_Please note that `sub` format depends on videos FPS (frame per second) and FPS is not known for all `itag`s. So this may not work as excepted. It will be fixed in future._
   
   
   
Changes
-------

see [CHANGELOG.md](CHANGELOG.md).

   
License
-------

MIT, see [LICENSE](LICENSE).

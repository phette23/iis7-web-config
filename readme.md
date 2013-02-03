# web.config Tweaks for IIS7.5

This is a sample web.config file for applying performance optimizations & security tweaks in Microsoft's proprietary server software. The overall goals are:

- GZIP responses (the httpCompression section)
- Setup a caching framework (the staticContent section of static/web.config)
- Force the latest IE rendering engine or Chrome Frame (the X-UA-Compatible header)
- Don't unnecessarily reveal server information

Much of the code comes from the [HTML5 Boilerplate server configs](https://github.com/h5bp/server-configs/) repo, but I've had to break off on my own because A) there are some serious problems with their approach & B) no one seems to maintain it. I filed [an issue](https://github.com/h5bp/server-configs/issues/90) that's been left open for months without response. It's still an excellent project, and maybe the non-IIS settings are of higher quality, but I need a better source of IIS defaults.

## Caching is hard

What's wrong with the H5BP web.configs? They *far-future cache an appcache manifest* which is [a horrible idea](https://speakerdeck.com/jaffathecake/application-cache-douchebag?slide=35 "Pertinent slide from the Appcache Douchebag deck"). That makes it impossible to update a website for the duration set in the expires header. HTML is also far-future cached which makes it impossible to update references to other resources. Their ETags removal doesn't work either & in the process screws up the Chrome Frame header. So the caching is screwed up & you won't force the best available rendering engine in IE. Chances are if you don't use an appcache (I do, sometimes) you'll be fine with their web.config.

In order to cache static content _without_ caching HTML and .manifest files, I've unfortunately had to resort to using a directory structure that disables caching at the root and then enables it in subdirectory which holds all the static assets. So the root holds the server-side script or HTML file and the appcache, while JS, CSS, and images would all be in subdirectories which use the static/web.config in this repo.

## Testing Notes

I test using `curl` in bash. If there's a better way, I don't know it.

```bash
curl -LI http://domain.tld/path/to/file
```

Should return something like:

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 31991
Content-Type: text/html; charset=UTF-8
Last-Modified: Mon, 05 Jan 1904 16:20:17 GMT
Accept-Ranges: bytes
Server:
X-UA-Compatible: IE=Edge,chrome=1
Date: Mon, 16 Jun 1904 22:58:15 GMT
```

while

```bash
curl -LI http://domain.tld/path/to/static/file.png
```

Should return something like:

```
HTTP/1.1 200 OK
Cache-Control: max-age=5184000
Content-Length: 7825
Content-Type: image/png
Last-Modified: Tue, 18 Jan 1904 20:37:35 GMT
Accept-Ranges: bytes
Server:
X-UA-Compatible: IE=Edge,chrome=1
Date: Mon, 16 Jun 1904 22:59:29 GMT
```

Note the differences in the Cache-Control header.

## Caveats

- I'm not a server guy, I'm a frontend web guy. These files might suck.
- Not tested anywhere except our dev & production servers. Probably IIS7.5 specific, definitely won't work on IIS6.
- A lot of this is probably easier done through the Server Manager GUI, but damn am I confused by that thing.

## License

[WTFPL](http://sam.zoy.org/wtfpl/)

The image included in the static directory is in the Public Domain (yay!) & I found it via the awesome [Noun Project](http://thenounproject.com/noun/library/#icon-No191).

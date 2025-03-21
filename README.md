# davidm42/docker-streamripper

[Streamripper](http://streamripper.sourceforge.net/) is an application that lets you record streaming mp3 to your hard drive. This is a [Docker](https://www.docker.com) image that eases setup.

## About Streamripper

> *From [the official about page](http://streamripper.sourceforge.net/about.php):*

Streamripper started as a way to separate tracks via Shoutcast's title-streaming feature. This has now been expanded into a much more generic feature, where part of the program only tries to "hint" at where one track starts and another ends, thus allowing a mp3 decoding engine to scan for a silent mark, which is used to find an exact track separation.

Streamripper allows you to download an entire station of music. Many of these mp3 radio stations only play certain genres, so you can now download an entire collection of goa/trance music, an entire collection of jazz, punk rock, whatever you want. 

## Usage

This docker image is available as an [automated build on Docker Hub](https://hub.docker.com/r/clue/streamripper/), so using it is as simple as running:

```bash
$ docker run ghcr.io/davidm42/docker-streamripper:master -h
```

Using this image for the first time will start a download automatically. Further runs will be immediate, as the image will be cached locally.

Docker containers run in an isolated environment, so in order to access your ripped mp3 files you will have to mount a volume (shared directory) like this:

```bash
-v $HOME/MyMusic:/home/streamripper
```

We're almost done. Now you need to get the streaming URL of your favorite online radio station. The [SHOUTcast](http://www.shoutcast.com/) index is probably a good start if you're unsure. The streaming URL to your online radio station will need to be passed as an argument to the streamripper container.

You can also supply any number of additional streamripper arguments that will be passed through unmodified. Describing all of them here is a bit out of scope, so you're recommended to familiarize yourself with the help (see the above `-h` example).

A common way to put this all together looks like this:

```bash
$ docker run -d -v $HOME/MyMusic:/home/streamripper ghcr.io/davidm42/docker-streamripper:master streamripper http://mystation.local/radio.pls -s -m 30 --xs2 -o never -T
```

This will start the streamripper container in a detached session in the background.

## Cleanup

This container is disposable, as it doesn't store any sensitive information at all. If you don't need it anymore, you can `stop` and `remove` it.

## Docker Compose

Similarly, you can also use [Docker Compose](https://docs.docker.com/compose/) to configure and run this image. Simply create a `docker-compose.yml` file like this:

```yml
services:
  streamripper:
    image: ghcr.io/davidm42/docker-streamripper:master
    restart: always
    volumes:
      - $HOME/MyMusic:/home/streamripper
    command: streamripper http://mystation.local/radio.pls -s -m 30 --xs2 -o never -T
```

You can then run this as a background service like this:

```bash
$ docker-compose up -d
```

## My personal trueNAS docker compose

I run it in TrueNAS scale with following docker-compose.yml. It includes some maybe unecessary, maybe needed codeset stuff for german streams, user & group of TrueNAS scale apps (568) and restart preferences.

```yml
services:
  streamripper:
    command: >-
      https://mp3.planetradio.de/plrchannels/hqtheclub.aac -o larger -s
      --codeset-filesys=UTF-8 --codeset-id3=UTF-8 --codeset-metadata=UTF-8
    environment:
      - PUID=568
      - PGID=568
    image: ghcr.io/davidm42/docker-streamripper:master
    restart: unless-stopped
    volumes:
      - $HOME/MyMusic:/home/streamripper
```

> Disclaimer: It goes without saying that it should be in your best interest to support the artists and stations. Sharing copyrighted files with the public is not a good idea unless you have permission to do so. Running this in your local network to discover new music may or may not be legal in your jurisdiction, sharing this with the public probably is not (IANAL).

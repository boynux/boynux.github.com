---
layout: post
permalink: /live-stream-music-from-s3
title: How to stream music tracks from S3
tags: Cloud Streaming AWS
category: Programming
excerpt: Let's see how to easily stream large video or audio files from cloud object stores like S3 or Azure Files directly to your mobile app or IoT devices.
---

Recently I started to work on a new pet project *[Daastani](https://www.daastani.com)*, check it out, it's really cool. I wanted to stream audio files directly from cloud to a RaspberryPi and from the other end to a mobile app. I didn't want to download the entire file, because it's slow and my storage is limited on both devices.

# How it works

This method makes use of object stores pre-signed URLs which is a common method in most of providers, here I'll explain the process for AWS S3 but a similar approach can be used for others as well.

Your device does not need to necessarily have access to the S3 object. The signed URL can be generated from another source, for example another microservice via a HTTP call or using MQTT protocol. After the application received the pre-signed URL it can directly fetch the data from the URL and plays the stream.

# Code

Let's use Python and use Boto3 + pygame packages. I also assume you you have configured you S3 bucket correctly and also AWS credentials are available in your environment.

```python
import boto3
import pygame
import requests


# File like object that streams the object from URL
class Stream(object):

    def __init__(self, url):
        self._file = requests.get(url, stream=True)

    def read(self, *args):
        if args:
            return self._file.raw.read(args[0])
        else:
            return self.file.raw.read()

    def close(self):
        self._file.close()

# Create boto3 S3 client and get the pre-signed URL
ession = boto3.Session()
s3 = session.client('s3', region_name='eu-central-1')
signedUrl = s3.generate_presigned_url(
    ClientMethod="get_object",
    ExpiresIn=1800,  # valid for 30 minutes
    HttpMethod='GET',
    Params={
        "Bucket": "myBucket",
        "Key": "myFile.mp3",
    }
)

stream = Stream(signedUrl)
pygame.init()
pygame.mixer.init()
pygame.mixer.music.set_volume(0.5)
pygame.mixer.music.load(stream)
pygame.mixer.music.play()
```


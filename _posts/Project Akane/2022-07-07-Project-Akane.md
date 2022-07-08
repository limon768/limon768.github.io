---
title: "Project: Akane"
date: 2022-07-07 09:45:47 +07:00
modified: 2022-07-07 09:09:47 +07:00
tags: [Python, AI , Project]
description: A personal AI use voice recognition to perform actions
---

## Introduction
A personal AI made with python use voice recognition to perform actions depending on the users voice command

## Requirements

The script requires python 3.X. and pip

You need the following modules installed for Akane

Speech_recognition

Google Text-to-Speech

Playsound

Requests

```python
pip install SpeechRecognition
pip install gTTS
pip install playsound
pip install requests
```


```python
import speech_recognition as sr
import time
import webbrowser as wb
from gtts import gTTS
import os
import playsound
import random
import urllib.request
import re
```

## Usage

To run the script

`python ai.py`

## Voice Commands

"Help" -> Tells the user all the command available

 What is your name -> says name of the AI

Tell me the time -> tells the current time

search -> search something in web
    opens up a browser to show result
    Default browser is Google Chrome

Subreddit -> search in reddit

Find Location -> open up location in google map

Play Music -> Plays music in youtube
    opens up a browser to play music

Games -> Guess the number game with AI

## Project Link
[Github](https://github.com/limon768/Akane-1.7)





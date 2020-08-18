---
layout: post
title: Speech Recognized Music Player
categories: [AI]
tags: [Python]
---
## Beta version
- support only for Python3
- Ubuntu

### Requirements

install flac
```bash
$ sudo apt-get install flac
```
install SpeechRecognition
```bash
$ pip3 install SpeechRecognition
```
install Pyaudio
```bash
$ sudo apt-get install python3-pyaudio
```
if you want to use other music player than modify
main.py
```python
os.system("cvlc " + ans)
```

### Running
run the main.py script
```bash
$ python3 main.py
```
[Code Link](https://github.com/MadPlayer/SpeechRecognizedMusicPlayer)

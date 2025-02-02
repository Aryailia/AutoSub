# AutoSub

- [About](#about)
- [Motivation](#motivation)
- [Installation](#installation)
- [Docker](#docker)
- [How-to example](#how-to-example)
- [How it works](#how-it-works)
- [TO-DO](#to-do)
- [Contributing](#contributing)
- [References](#references)

## About

AutoSub is a CLI application to generate subtitle files (.srt, .vtt, and .txt transcript) for any video file using [Mozilla DeepSpeech](https://github.com/mozilla/DeepSpeech). I use the DeepSpeech Python API to run inference on audio segments and [pyAudioAnalysis](https://github.com/tyiannak/pyAudioAnalysis) to split the initial audio on silent segments, producing multiple small files.

⭐ Featured in [DeepSpeech Examples](https://github.com/mozilla/DeepSpeech-examples) by Mozilla

## Motivation

In the age of OTT platforms, there are still some who prefer to download movies/videos from YouTube/Facebook or even torrents rather than stream. I am one of them and on one such occasion, I couldn't find the subtitle file for a particular movie I had downloaded. Then the idea for AutoSub struck me and since I had worked with DeepSpeech previously, I decided to use it. 


## Installation

* Clone the repo. All further steps should be performed while in the `AutoSub/` directory
    ```bash
    $ git clone https://github.com/abhirooptalasila/AutoSub
    $ cd AutoSub
    ```
* Create a pip virtual environment to install the required packages
    ```bash
    $ python3 -m venv sub
    $ source sub/bin/activate
    $ pip3 install -r requirements.txt
    ```
* Download the model and scorer files from DeepSpeech repo. The scorer file is optional, but it greatly improves inference results.
    ```bash
    # Model file (~190 MB)
    $ wget https://github.com/mozilla/DeepSpeech/releases/download/v0.9.3/deepspeech-0.9.3-models.pbmm
    # Scorer file (~950 MB)
    $ wget https://github.com/mozilla/DeepSpeech/releases/download/v0.9.3/deepspeech-0.9.3-models.scorer
    ```
* Create two folders `audio/` and `output/` to store audio segments and final SRT and VTT file
    ```bash
    $ mkdir audio output
    ```
* Install FFMPEG. If you're running Ubuntu, this should work fine.
    ```bash
    $ sudo apt-get install ffmpeg
    $ ffmpeg -version               # I'm running 4.1.4
    ```
    
* [OPTIONAL] If you would like the subtitles to be generated faster, you can use the GPU package instead. Make sure to install the appropriate [CUDA](https://deepspeech.readthedocs.io/en/v0.9.3/USING.html#cuda-dependency-inference) version. 
    ```bash
    $ source sub/bin/activate
    $ pip3 install deepspeech-gpu
    ```


## Docker

* Installation using Docker is pretty straight-forward.  
    * First start by downloading training models by specifying which version you want:
        * if you have your own, then skip this step and just ensure they are placed in project directory with .pbmm and .scorer extensions
    ```bash
    $ ./getmodel.sh 0.9.3
    ```

    * Then for a CPU build, run: 
    ```bash
    $ docker build -t autosub .
    $ docker run --volume=`pwd`/input:/input --name autosub autosub --file /input/video.mp4
    $ docker cp autosub:/output/ .
    ```

    * For a GPU build that is reusable (saving time on instantiating the program):
    ```bash
    $ docker build --build-arg BASEIMAGE=nvidia/cuda:10.1-cudnn7-runtime-ubuntu18.04 --build-arg DEPSLIST=requirements-gpu.txt -t autosub-base . && \
    docker run --gpus all --name autosub-base autosub-base --dry-run || \
    docker commit --change 'CMD []' autosub-base autosub-instance
    ```
    * Then
    ```bash
    $ docker run --volume=`pwd`/input:/input --name autosub autosub-instance --file video.mp4
    $ docker cp autosub:/output/ .
    ```

## How-to example

* Make sure the model and scorer files are in the root directory. They are automatically loaded
* After following the installation instructions, you can run `autosub/main.py` as given below. The `--file` argument is the video file for which SRT file is to be generated
    ```bash
    $ python3 autosub/main.py --file ~/movie.mp4
    ```
* After the script finishes, the SRT file is saved in `output/`
* Open the video file and add this SRT file as a subtitle, or you can just drag and drop in VLC.
* The optional `--split-duration` argument allows customization of the maximum number of seconds any given subtitle is displayed for. The default is 5 seconds
    ```bash
    $ python3 autosub/main.py --file ~/movie.mp4 --split-duration 8
    ```
* By default, AutoSub outputs in a number of formats. To only produce the file formats you want use the `--format` argument:
    ```bash
    $ python3 autosub/main.py --file ~/movie.mp4 --format srt txt
    ```


## How it works

Mozilla DeepSpeech is an amazing open-source speech-to-text engine with support for fine-tuning using custom datasets, external language models, exporting memory-mapped models and a lot more. You should definitely check it out for STT tasks. So, when you first run the script, I use FFMPEG to **extract the audio** from the video and save it in `audio/`. By default DeepSpeech is configured to accept 16kHz audio samples for inference, hence while extracting I make FFMPEG use 16kHz sampling rate. 

Then, I use [pyAudioAnalysis](https://github.com/tyiannak/pyAudioAnalysis) for silence removal - which basically takes the large audio file initially extracted, and splits it wherever silent regions are encountered, resulting in smaller audio segments which are much easier to process. I haven't used the whole library, instead I've integrated parts of it in `autosub/featureExtraction.py` and `autosub/trainAudio.py` All these audio files are stored in `audio/`. Then for each audio segment, I perform DeepSpeech inference on it, and write the inferred text in a SRT file. After all files are processed, the final SRT file is stored in `output/`.

When I tested the script on my laptop, it took about **40 minutes to generate the SRT file for a 70 minutes video file**. My config is an i5 dual-core @ 2.5 Ghz and 8 gigs of RAM. Ideally, the whole process shouldn't take more than 60% of the duration of original video file. 


## TO-DO

* Pre-process inferred text before writing to file (prettify)
* Add progress bar to `extract_audio()`
* GUI support (?)


## Contributing

I would love to follow up on any suggestions/issues you find :)


## References
1. https://github.com/mozilla/DeepSpeech/
2. https://github.com/tyiannak/pyAudioAnalysis
3. https://deepspeech.readthedocs.io/

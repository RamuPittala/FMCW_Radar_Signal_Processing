<img width="1920" height="975" alt="2019_05_29_pbms007_range_over_time" src="https://github.com/user-attachments/assets/d4d70d50-0ca1-43c0-95b7-ce18f61b9745" />
# FMCW_Radar_Signal_Processing

## Introduction
This repository contains the work which attempts to solve 2D-positioning of human being using an open source FMCW radar data. It also highlights the challenges with current approach and future possible directions.
The approach here is to explore usage of real time geometry to solve positioning of the objects, which has been inspired from the work "3D Tracking via Body Radio Reflections" (link: https://witrack.csail.mit.edu/witrack-paper.pdf). In the mentioned paper, researchers use elliptic equations (transmitter and receiver as focii of an ellipse with time delay from an object as distance) to solve for the positioning of the object. I request you to go through the paper as it has amazing work. 

## What I am doing here?
In the mentioned paper, researchers used customized setup (T-shape, ends are receivers and contact point as transmitter) to point the directed radiation and solving for positioning. Here we will be using already available open source data, where typical setup is used in real world (like MxN array of transmitters and receivers to do virtual beamforming). Here an ADC data has been used whose configuration details are provided below.
radar_configs = {
"startFreqConst_GHz": 77.0,
"bandwidth_GHz": 0.67,
"chirpDuration_usec": 60.0, # single chirp per antenna
"freqSlopeConst_MHz_usec": 21.0, # slope: MHz/us
"numAdcSamples": 128, # samples
"digOutSampleRate": 4000.0, # sampling rate: Ksps
"numLoops": 255, # chirp loops
"framePeriodicity_msec": 33.33333, # frame rate 30 fps
}

Dataset Link: https://drive.google.com/file/d/1OfqXXgoFg6xRYZkRqPJye4cQ29Fomh3l/view?usp=sharing (No need to worry. It is safe to download :).)

## Signal/Radar Cube Processing
If we look at the data, each frame shape would be (128, 255, 4, 2) which exactly represents  samples (128), chirps (255), receivers (4), transmitters (2). The data collection was done on the campus, road, and parking lot during the daytime, with the focus of capturing the data for six main objects: pedestrian, cyclist, car, motorbike, bus, truck. The collected objects can be either moving (mostly) or static. A single data collection run consisted of multiple objects listed above moving or being static at a normal speed for 30 seconds in front of the testbed. 

What we do in general is, apply FFT on one radar frame data to get range FFT. And further apply FFT along slow-time (across frames/chirps) axis to get doppler FFT and proceed for range and angle estimation. Here we try to do it in different way i.e., we do apply FFT on fast-time (ADC data per each chirp) axis to get range-FFTs and average them together per each 128-frames to get one average range-FFT by fixing one virtual tranceiver pair. By doing so, we do get range-FFT over time (1D-FFTs stacked horizontally to mimic like graph where x-axis is time and y-axis is range). Finally, we do have range map over time which look like below.
<img width="1920" height="975" alt="2019_05_29_pbms007_range_over_time" src="https://github.com/user-attachments/assets/f9053489-9caa-41b6-b4f6-3bbf5f451d0c" /> 
And corresponding one sample optical image (from sequency of many images) is shown below. It contain one pedestrian and one cyclist performing walking and cycling activities simultaneously. 
<img width="1440" height="1080" alt="image" src="https://github.com/user-attachments/assets/32e495ef-0033-4d32-aaef-a28ed4bc5e7b" /> 
Entire sequence of images are provided in a data folder and corresponding radar data are also provided in another corresponding subfolder. So, above range-time map reflects the activities the mentioned objects in a sequency manner. 
To highlight the activity region in range-time map, we used 1D-CFAR thresholding. We had done it for each column of range-time map. The result can be seen below.
<img width="1920" height="975" alt="2019_05_29_pbms007_CFAR_detections" src="https://github.com/user-attachments/assets/79d0e749-bcdb-4542-8e24-3e96886fce02" />
We can clearly see those two curves reflect two person's activities. We want to get seperate each of them into continuous individual tracks so that we can get distance estimation for each person at any time. But the challenges here are: 1. Some outliers are there instead of continuous track detection points. We need get rid of them first. 2. Here's the main challenge i.e., we need to sepearate them into individual tracks. Sometimes, the track can discontinue in between as well (PFA picture below). And multiple tracks will will be vanishing in between.
<img width="1920" height="975" alt="2019_04_30_pcms001_cfar_detections" src="https://github.com/user-attachments/assets/232faed7-5908-422a-a09e-7fed95a30149" />






# Advent of Cyber Day 1: Maybe SOC-mas music, he thought, doesn't come from a store?
This task was dealing with OPSEC (Operational Security). You, as an investigator, have decided to dig into the YouTube-to-MP3 converter other SOC-mas organizers have been sharing.

## Why are we investigating the converter?
They can be used to deliver malicious software and be used for phishing credentials or other sensitive information. 

## Where to start?
1. Start the machine that was created for this challenge.
2. Navigate to the IP address generated if you are on the attack box. It will take you to the converter.
3. Using the link provided in the challenge https[:]//www[.]youtube[.]com/watch?v=dQw4w9WgXcQ which I have defanged is the YouTube video that have been suggested to use.
4. Choose either file extensions (it honestly doesn't matter which) and download the file and extract it.

## What to do next?
Navigate to where the file downloaded and run the `file` command like this `file song.{mp3|mp4}` This command out puts information on what the file "looks" like to the system. 

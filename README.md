# obs-udp-srt Stream distribution
Sending a test stream, from a local OBS instance to an OBS instance in the cloud using srt-live-transmit.  
I have installed ffmpeg with the SRT library on my Windows 10 computer. I also have srt-live-transmit installed on my local computer.  
## send with OBS Studio to the local network to a multicast address
setting -> stream -> Server:
`udp://224.0.0.1:9999` (multicast address on the LAN)
## receive and view with ffplay
`C:\ffmpeg\bin>ffplay.exe udp://@224.0.0.1:9999`
## receive and forward with srt-live-transmit to OBS in the Cloud
`C:\srt-live>srt-live-transmit.exe udp://@224.0.0.1:9999?mode=listener srt://xxx.xxx.xxx.xxx:10000?mode=caller`

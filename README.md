# obs-udp-srt Stream distribution
Sending a test stream, from a local OBS instance to an OBS instance in the cloud using srt-live-transmit.  
I have installed ffmpeg with the SRT library on my local "Windows 10 computer". I also have srt-live-transmit installed on my local computer.  

## About this Repository

Dieses Repository dokumentiert frühere Experimente mit **OBS Studio, UDP und SRT**, wie sie in praktischen Live-Streaming-Szenarien aufgetreten sind.

Es zeigt, wie ein lokaler OBS-Stream zunächst über **UDP (z. B. Multicast im LAN)** verteilt und anschließend mit **srt-live-transmit** oder **FFmpeg** in ein SRT-Signal überführt wird, das weiter zu einem Cloud-OBS oder Restream-Server gesendet werden kann.

Der Fokus liegt nicht auf einer fertigen Lösung, sondern auf **echten Versuchen, Fehlern, Fallstricken und Zwischenergebnissen**, z. B.:

- Empfang eines Multicast-UDP-Streams mit ffplay oder OBS
- Übergabe in ein SRT-Transport-Tool (srt-live-transmit, FFmpeg)
- Herausforderungen mit Ports, Firewalls und Cloud-Zugriff
- Erfahrungen aus der Praxis (4G/LTE, Cloud-Server usw.)

Dieses Repo ist kein stabiles Framework, sondern ein **historisches Laborprotokoll**, das zeigt, wie Live-Streaming-Probleme mit SRT und OBS in der Praxis angegangen wurden — bevor klarere Patterns und stabilere Tools (wie MediaMTX oder systemd-basierte Architekturen) etabliert wurden.


## send with OBS Studio to the local network to a multicast address
setting -> stream -> Server:  
`udp://224.0.0.1:9999` (multicast address on the LAN)  

## receive and view with ffplay
`C:\ffmpeg\bin>ffplay.exe udp://@224.0.0.1:9999`  
Attention: ffplay must be started first. Only then start the stream with OBS.  

## received on the local network on another computer with OBS
new Source -> Media Source -> Properties:  
Input: `udp://@224.0.0.1:9999`  
## receive on the local network and forward the stream with srt-live-transmit to OBS in the Cloud  
### srt-live-transmit
Again, it is important that srt-live-transmit is started as listener before OBS sends the stream.  
`C:\srt-live>srt-live-transmit.exe udp://@224.0.0.1:9999?mode=listener srt://xxx.xxx.xxx.xxx:10000?mode=caller`  
This does not work. Is port 10000 actually open for incoming UDP traffic?  

### ffmpeg (transmitter)
`C:\ffmpeg\bin>ffmpeg.exe -i udp://@224.0.0.1:9999 -c copy -f mpegts "srt://xxx.xxx.xxx.xxx:10000?mode=caller&latency=50000"`  

### ffmpeg (receiver on nginx rtmp-module - restream server)
`root@restreamServer-02:/etc/nginx# systemctl cat ffmpeg_srt.service`  
`ffmpeg -loglevel panic -re -i srt://0.0.0.0:xxxxx?mode=listener -c copy -f flv rtmp://localhost:xxxx/xaplication-namex`  
show Server log  
`root@restreamServer-02:/etc/nginx# journalctl -f -u ffmpeg_srt.service`  

## Streaming from camera encoders to OBS in the cloud
SRT: `Caller`  
Caller Server: `xxx.xxx.xxx.xxx`  
Caller Port: `10000`  
latency: `2000` (2 seconds)  

## Streaming from local OBS to OBS in the cloud
`srt://xxx.xxx.xxx.xxx:10000?mode=caller`  
This does not work.  

---
**tested for a solution with apps4rent.com**  
![apps4rent](SRTover4G_Apps4rent.png)  

---

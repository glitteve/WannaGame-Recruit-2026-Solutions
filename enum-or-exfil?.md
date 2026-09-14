# WannaGame-Recruit-2026-Solutions
**Statement:** https://ctf.uithacking.club/training/6?challenge=27

First of all, we're given a Packet Capture file (PCAP), there are lots of ICMP here.

<img width="1767" height="1064" alt="Screenshot 2026-09-14 224018" src="https://github.com/user-attachments/assets/486c9d0e-df0d-4f01-b433-a585ba1e4684" />


At the beginning of the inspection, there are multiple HTTP requests that contain outlandish information, which seems to be Base64.

<img width="1493" height="295" alt="Screenshot 2026-09-14 225126" src="https://github.com/user-attachments/assets/d16e979f-82af-4809-b07f-af1a74927ffc" />

We will use `tshark` to capture all of the Base64-encoded data from conversation streams.

```bash
$ tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed -e 's/firefly\://g' | base64 -d | file -
/dev/stdin: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 544x600, components 3

$ tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed -e 's/firefly\://g' | base64 -d > img.jpg
```

The file names `img.jpg` as we see: 

<img width="544" height="600" alt="img" src="https://github.com/user-attachments/assets/10543b51-ebca-4618-9e9d-b81c49ddeb98" />

Is there any secret data (or flag) hidden in the picture? Let's inspect it. First, we use `binwalk -e `







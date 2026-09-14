# WannaGame-Recruit-2026-Solutions
**Statement:** https://ctf.uithacking.club/training/6?challenge=27

First of all, we're given a Packet Capture file (PCAP), there are lots of packets here.

<img width="1767" height="1064" alt="Screenshot 2026-09-14 224018" src="https://github.com/user-attachments/assets/486c9d0e-df0d-4f01-b433-a585ba1e4684" />


At the beginning of the inspection, there are multiple HTTP requests that contain outlandish information, which seems to be Base64. We need to decode those twice. 
Fortunately, when we inspect the first TCP conversation stream and decode this outlandish information twice, we discover the magic bytes of this: 

<img width="502" height="161" alt="image" src="https://github.com/user-attachments/assets/8707c931-227d-4c7c-94a6-fdc2785286f9" />

After that, we conclude that the data we need is in a picture format (JFIF).

<img width="1493" height="295" alt="Screenshot 2026-09-14 225126" src="https://github.com/user-attachments/assets/d16e979f-82af-4809-b07f-af1a74927ffc" />

We will use `tshark` to capture and encode twice all of the Base64-encoded data from conversation streams and merge them to create a full image.

```bash
$ tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed -e 's/firefly\://g' | base64 -d | file -
/dev/stdin: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 544x600, components 3

$ tshark -r challenge.pcap -Y "http" -T fields -e http.authbasic | sed -e 's/firefly\://g' | base64 -d > img.jpg
```

The file names `img.jpg` as we see: 

<img width="544" height="600" alt="img" src="https://github.com/user-attachments/assets/10543b51-ebca-4618-9e9d-b81c49ddeb98" />

Is there any data or even a secret flag hidden in the picture? Let's inspect it. First, we use `binwalk -e img.jpg` and `exiftool img.jpg`, but there isn't any suspicious information. Next, we try inspecting it with `steghide extract -sf img.jpg` to extract encrypted secret data, but it requires us to have a passphrase to decrypt the secret data.

How do we find this passphrase? Let's have a look at ICMP packets.<img width="1516" height="739" alt="Screenshot 2026-09-14 231030" src="https://github.com/user-attachments/assets/7efd4567-2d76-4c13-89bb-481df9da633f" />

When we try clicking each ICMP packet, we can see suspicious data in it

<img width="937" height="278" alt="Screenshot 2026-09-14 231206" src="https://github.com/user-attachments/assets/7d303fd6-a733-4348-b01a-4f182c45e969" />

As we can see, every ICMP packet has a packet that includes the suspicious first 2 letters and the rest of repeated letters. Maybe the letters excepting first 2 letters, are not really important.

We convert every 2 letters (representing two hex values) of all ICMP packets into ASCII values and merge all of them using `tshark`.

```bash
$ tshark -r challenge.pcap -Y "icmp" -T fields -e data.data | cut -c1-2 | uniq | xxd -r -p
Result: sneaky_network
```

Now, we have the passphrase, which is named `sneaky_network`. Using that to decrypt the secret data from the picture and finally get a flag.
```bash
$ steghide extract -p "sneaky_network" -sf hello.jpg
wrote extracted data to "flag.txt".

$ cat flag.txt
W1{1t's_n0t_that_hard_t0_solve_th1s_r1ght?_(*^_^*)}
```





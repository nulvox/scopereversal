# NVG50

I was gifted this scope and found the IP camera side of it very interesting but alarmingly open. This writeup covers my efforts to know more about how it works and perhaps improve it some for my own use. While the product packaging is all NVG30, the firmware bootscreen says NVG50. :shrug: Upon further inspection, they seem to be very similar in the software stack, but distinct products. I have the NVG50 and I'm pretty pleased about that. 😅

# Endpoints
This is equipped with a USB interface and a self-hosted WiFi endpoint.

## USB
This is pretty boring:
```
[ 6001.204655] usb 3-2: new high-speed USB device number 5 using xhci_hcd
[ 6001.341299] usb 3-2: New USB device found, idVendor=0603, idProduct=8611, bcdDevice= 1.00
[ 6001.341303] usb 3-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 6001.341311] usb 3-2: Product: DEMO1
[ 6001.341313] usb 3-2: Manufacturer: NOVATEK
[ 6001.341315] usb 3-2: SerialNumber: 96680-00000-001
[ 6001.352710] usb-storage 3-2:1.0: USB Mass Storage device detected
[ 6001.352856] scsi host0: usb-storage 3-2:1.0
```

the exposed device is the sd card. No other USB endpoints are exposed.

## WiFi
This is where things get juicy. The system uses a hard-coded password of `12345678` with a static SSID. 
I found these services:
```
Nmap scan report for 192.168.1.254
Host is up (0.0039s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
554/tcp  open  rtsp
3333/tcp open  dec-notes
MAC Address: CC:64:1A:3B:C5:D9 (Shenzhen Bilian Electronic，LTD)Nmap scan report for 192.168.1.254
Host is up (0.0039s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
554/tcp  open  rtsp
3333/tcp open  dec-notes
MAC Address: CC:64:1A:3B:C5:D9 (Shenzhen Bilian Electronic，LTD)
```

Cool, cheap wifi cam with http, rtsp, and dec-notes (which I was unfamiliar with until this, but is apparently a second conferencing solution... :shrug:)

### http
This is a simple webpage to manage the contents of the SD card. It lets the user upload new files to the various folders and download what is there. A curious "Upload Custom" form is present which plays by different rules somehow and passes the url param `?custom=1` and the files don't seem to appear on the SD card anywhere, suggesting they may end up somewhere on the root. 

#### Dir Traversal
The first thing I wanted to do was escape the http root, no dice on trivial inputs from the Webpage. Still need to try it out from curl or python

#### Clobber the Index
While it kinda looks like we can upload to the http root at first glance, there is strange behavior. Uploading anything like `index.html` in the standard upload box causes the session to be closed by the host, but uploading with the `upload custom` dialog takes the file and it never shows up (as normal for that form).

#### Codec Abuse
video codecs are complicated and easy to mess up. I suspect I can upload a speciallyc rafted video file to trigger a crash on the playback feature of the scope. Default format is MP4

### rtsp
easy wins here, just popped a stream up with `ffplay rtsp://192.168.1.254:554`, easily saved it with `ffmpeg -i rtsp://192.168.1.254:554 -r 15 output.mp4`

This doesn't really take input, so it's likely not a good way to get more capabilities, but the simplicity of the output it generates makes it easy for improving the local workflow. 

### dec-notes
@TODO look into this protocol, try connecting, see if it's a good foothold

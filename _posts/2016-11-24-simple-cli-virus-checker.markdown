---
layout: post
title:  "Simple CLI Virus Checker"
date:   2016-11-24
---

Couple of days ago I downloaded an app for mac and I suspected it was a virus. Since I don’t have an antivirus, I decided that it would be too risky to run the app without making sure I won’t suffer from any ransomware this winter.

There’s a nice service called  [VirusTotal](https://www.virustotal.com/) which allows you to upload a file, it then scans it using the most common antiviruses and shows you which ones think the file is malicious and which don’t.

The only problem is that the file was pretty big (~300MB) and I didn’t want to waste time uploading it to VirusTotal on the super slow Israeli upload bandwidth. Lucky for me, VirusTotal allow doing the same operation by entering the  [checksum](https://en.wikipedia.org/wiki/Checksum) of the file.

Since I’m lazy as hell, getting the checksum of the file and searching for it on VirusTotal every time I wanted to check a file seemed too difficult; So I wrote a bash function that does it for me 🙂

```bash
# Ephi's virus checker
 export PYTHONIOENCODING=utf8
 export VT_API_KEY=""
 
function isvirus {
     CHECKSUM="$(md5 -q $1)"
     curl -sS --request POST --url 'https://www.virustotal.com/vtapi/v2/file/report' -d   apikey=${VT_API_KEY} -d "resource=${CHECKSUM}" | \
     python -c "import sys, json; parsed = json.load(sys.stdin); print 'Don\\'t know..' if 'positives' not in parsed else 'nope..' if parsed['positives'] == 0 else 'yep..'"
 }
 # End Ephi's virus checker
```

Just copy and paste it into the end of your  `.bashrc` file (it’s located in your home directory), replace the VirusTotal api key with your key (you can obtain one for free by signing up to VirusTotal) and then run  `source ~/.bashrc` or reopen your terminal and you’re good to go!

To use this function just type  `isvirus FILENAME.EXTENTION` in the terminal, and it would print either  `Don't know` ,  `yep` or `nope` .. Simple as that!
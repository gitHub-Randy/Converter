
## Handleiding en html+ngingx conf om een video stream te maken
Hoe zet ik een Live stream server en client op<br>
versie 2.0<br>
Auteur: Randy Anft<br>

Doel van deze handleiding is het om een server en een client daarvoor op te zetten waardoor het mogelijk is om een Video stream te maken die nooit eindigt en waar de content dynamisch aangepast kan worden.

Deze handleiding houdt de volgende zaken in:

- Nginx + rtmp module
- FFMPEG
- html client pagina


Deze impelmentatie is op het moment van schrijven op een windows machine(client) met daarop een linux Virtual machine( server) getest.
Er wordt ervan uitgegaan dat er een virtual machine met linux(ubuntu 18.x) geinstalleerd is waarbij de porten 1935 en 80 voor zowel udp als ook tcp opengezet zijn.


# Nginx + rtmp

1) zorg ervoor dat ubuntu up to date is:<br>
	`sudo apt update` <br>
	`sudo apt upgrade`

2) installeer nginx:<br>
	`sudo apt install nginx`

3) installeer de rtmp module:<br>
	`sudo add-apt-repository universe`<br>
	`sudo apt install libnginx-mod-rtmp`

4) pas de configuratie van nginx aan:<br>
	`sudo nano /etc/nginx/nginx.conf`


  maak het bestand leeg en paste de inhoudt van de volgende link erin:<br>
  https://github.com/gitHub-Randy/Converter/blob/master/nginx.conf.txt


5) herstart nginx. Dit moet altijd gedaan worden als dit bestand vernadert wordt:<br>
	`sudo systemctl restart nginx`



# FFMPEG
1) Installeer FFMPEG( Deze moet misntens versie 4.X hebben)<br>
`sudo add-apt-repository ppa:jonathonf/ffmpeg-4`<br>
`sudo apt-get update`<br>
`sudo apt-get install ffmpeg`<br> 


## Hoe Content Streamen:

alle content die gestream zal worden moet in een video bestand zijn. dit kan mp4, mov, flv etc. zijn.
Verder moet er om ervoor te zorgen dat de stream continu gaat door streamen een playlist aangemaakt worden in de vorm van 2 text bestanden.

Maak 2 txt bestanden 'root.txt' en nested.txt aan:<br>
### root.txt:<br>
`file 'nested.txt'<br>`
 <br>
### nested.txt:<br>
  
    ffconcat version 1.0  
    file 'bestand1.mp4'
    file 'bestand2.mp4' 
    file 'nested.txt'



## Converten(OPTIONEEL):
Deze stap is optioneel omdat het niet altijd nodig is om een bestand te converten. Deze stap dient ervoor om te zorgen dat de 		bestand in het juiste format(codec, timebase) gezet wordt:
<br>
`sudo ffmpeg -i inputVideo.mp4 -c:v libx264 -video_track_timescale 90000 outputVideo.mp4`
<br>	
`-i`: is de input file <br>
`-c:v libx264`: is de codec die het video in h264 encoded(moet bij elk bestand gelijk zijn)<br>
`-video_track_timescale`: zorgt ervoor dat de timeabse aangepast wordt(deze moet bij elk bestand gelijk zijn)<br>
`outputVideo.mp4`: is de output bestand <br>

optionele parameters:<br>
`-s WxH`: hiermee kan de groote van het video ingesteld worden. voorbeeld: -s 1280x720<br>
	

# Streamen:
Dit is het eigenlijke streamen. de playlist wordt ingelezen, geloopt nog een keer encoded(verplicht) en wordt vervolgens naar de rtmp module gestuurd:
<br>
<br>
 ```sudo ffmpeg -fflags +genpts -f concat -safe 0 -stream_loop -1 -i ./root.txt  -preset ultrafast -vcodec libx264 -s 600x900 -b:v 3000k -b:a 0  -f flv rtmp://127.0.0.1:1935/hls/test```
<br>
<br>
`-fflags`: zet formaat flags aan<br>
<br>
`+genpts`: zorgt ervoor dat PTS(https://en.wikipedia.org/wiki/Presentation_timestamp) gegenereerd worden als DTS(Decoding Time Stamp) aanwezig zijn.<br>
<br>
`-f`: format of functie<br>
<br>
`concat`: zorgt ervoor dat de files in de playlist samen ingelezen worden alsof het een file is<br>
<br>
`-safe 0`: zorgt ervoor dat elke file path geaccepteerd wordt.<br>
<br>
`-stream_loop -1`: zorgt ervoor dat de input oneindig vaak geloopt wordt<br>
<br>
`-i`: input(playlist in dit geval)<br>
<br>
`-preset`: bepaald hoesnel iets gedaan moet worden. in dit geval is het ultrafast(snelste) hoe sneller het ingesteld is des te minder kwaliteit kan de stream hebben.<br>
<br>
`-vcodec`: welke codec moet gebruikt worden. in dit geval libx264<br>
<br>
`-s`: stelt de groote van de content in<br>
<br>
`-b:v`: bitrate van de video stream. Hoe hoger des te meer data dus betere kwaliteit<br>
<br>
`-b:a`: bitrate van de audio stream<br>
<br>
`-f flv`: exporteer het als flv bestand<br>
<br>
`rtmp://server-ip-adress:port/applicatie/stream_key`:<br>
is de server adress waar de content heen gestuurd moet worden. stream key kan willekeurig zijn maar moet wel onthouden worden zodat er bekend is hoe de video stream ontvangen kan worden.<br>
<br>

# Client:
De client bestaat uit een html pagina met de libary hls.js als video player(node.js nodig:https://tecadmin.net/install-latest-nodejs-npm-on-ubuntu/).
Deze wordt html pagian wordt op de nginx server geplaatst om vanuit daar gehost te worden. de locatie waar deze geplaatst moet worden is:
`/usr/local/nginx/html`

Hier een voorbeeld hoe de html eruit kan zien:
https://github.com/gitHub-Randy/Converter/blob/master/player.html

	


Deze pagina kan dan op de client machine opgeroepen worden m.b.v. de URL: http://server-ip/player.html
FFMPEG moet wel streamen daarmee de strem op de html pagina afgespeeld kan worden. 


# Gebruikte Resources onder andere:

https://opensource.com/article/19/1/basic-live-video-streaming-server	<br>
https://linuxize.com/post/how-to-install-ffmpeg-on-ubuntu-18-04/	<br>
https://www.ffmpeg.org/ffmpeg-all.html	<br>
https://dev.to/samuyi/how-to-setup-nginx-for-hls-video-streaming-on-centos-7-3jb8 <br>
https://github.com/video-dev/hls.js/ <br>
https://www.nginx.com/ <br>
https://github.com/arut/nginx-rtmp-module <br>



	

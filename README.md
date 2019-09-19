# Converter
File converter for content<br>
Hoe zet ik een Live stream server en client op<br>
versie 2.0<br>
Auteur: Rnady Anft<br>

Doel van deze handleiding is het om een server en een client daarvoor op te zetten waardoor het mogelijk is om een Video stream te maken die nooit eindigt en waar de content dynamisch aangepast kan worden.

Deze handleiding houdt de volgende zaken in:

- Nginx + rtmp module
- FFMPEG
- html client pagina


Deze impelmentatie is op het moment van schrijven op een windows machine(client) met daarop een linux Virtual machine( server) getest.
Er wordt ervan uitgegaan dat er een virtual machine met linux(ubuntu 18.x) geinstalleerd is waarbij de porten 1935 en 80 voor zowel udp als ook tcp opengezet zijn.


# Nginx + rtmp

1) zorg ervoor dat ubuntu up to date is:
	sudo apt update
	sudo apt upgrade

2) installeer nginx: 
	sudo apt install nginx

3) installeer de rtmp module:
	sudo add-apt-repository universe
	sudo apt install libnginx-mod-rtmp

4) pas de configuratie van nginx aan:
	sudo nano /etc/nginx/nginx.conf


  maak het bestand leeg en paste de inhoudt van de volgende link erin:<br>
  https://github.com/gitHub-Randy/Converter/blob/master/nginx.conf.txt


5) herstart nginx. Dit moet altijd gedaan worden als dit bestand vernadert wordt:<br>
	sudo systemctl restart nginx



# FFMPEG
1) Installeer FFMPEG( Deze moet misntens versie 4.X hebben)
	sudo add-apt-repository ppa:jonathonf/ffmpeg-4
	sudo apt-get update
	sudo apt-get install ffmpeg


Hoe Content Streamen:

alle content die gestream zal worden moet in een video bestand zijn. dit kan mp4, mov, flv etc. zijn.
Verder moet er om ervoor te zorgen dat de stream continu gaat door streamen een playlist aangemaakt worden in de vorm van 2 text bestanden.

1) maak 2 txt bestanden 'root.txt' en nested.txt aan:<br>
	root.txt:<br>
	file 'nested.txt'<br>
  <br>
	nested.txt:<br>
  
  ffconcat version 1.0<br>
	file 'bestand1.mp4'<br>
	file 'bestand2.mp4'<br>
	file 'nested.txt'


Dit is onze playlist. de root.txt bestand is onze entry point. hier worden de echte playlists toegevoegd in de vorm van txt bestanden.
de nested.txt moet in iedergeval ffconcat version 1.0 beinhouden waarnaar alle bestanden die in de playlist horen staan.<br>
LET OP: ALLE BESTANDEN IN DE GEHELE PLAYLIST MOETEN DE ZELFDE CODECS EN TIMEBASES HEBBEN (bv: alleen maar .mp4 of alleen maar .mov)

1.5)Converten(OPTIONEEL):
	Deze stap is optioneel omdat het niet altijd nodig is om een bestand te converten. Deze stap dient ervoor om te zorgen dat de bestand in het juiste format(codec, timebase) gezet wordt:
	<br>
  sudo ffmpeg -i inputVideo.mp4 -c:v libx264 -video_track_timescale 90000 outputVideo.mp4 
	
	-i: is de input file
	-c:v libx264: is de codec die het video in h264 encoded(moet bij elk bestand gelijk zijn)
	-video_track_timescale: zorgt ervoor dat de timeabse aangepast wordt(deze moet bij elk bestand gelijk zijn)
	outputVideo.mp4: is de output bestand
	
optionele parameters:<br>
	-s WxH: hiermee kan de groote van het video ingesteld worden. voorbeeld: -s 1280x720<br>
	

# Streamen:
	Dit is het eigenlijke streamen. de playlist wordt ingelezen, geloopt nog een keer encoded(verplicht) en wordt vervolgens naar de rtmp module gestuurd:
	<br>
  <br>
  ```sudo ffmpeg -fflags +genpts -f concat -safe 0 -stream_loop -1 -i ./root.txt  -preset ultrafast -vcodec libx264 -s 600x900 -b:v 3000k -b:a 0  -f flv rtmp://127.0.0.1:1935/hls/test```
	<br>
	-fflags: zet formaat flags aan<br>
	+genpts: zorgt ervoor dat PTS(https://en.wikipedia.org/wiki/Presentation_timestamp) gegenereerd worden als DTS(Decoding Time Stamp) aanwezig zijn.<br>
	-f: format of functie<br>
	concat: zorgt ervoor dat de files in de playlist samen ingelezen worden alsof het een file is<br>
	-safe 0: zorgt ervoor dat elke file path geaccepteerd wordt.<br>
	-stream_loop -1: zorgt ervoor dat de input oneindig vaak geloopt wordt<br>
	-i: input(playlist in dit geval)<br>
	-preset: bepaald hoesnel iets gedaan moet worden. in dit geval is het ultrafast(snelste) hoe sneller het ingesteld is des te minder kwaliteit kan de stream hebben.<br>
	-vcodec: welke codec moet gebruikt worden. in dit geval libx264<br>
	-s: stelt de groote van de content in<br>
	-b:v bitrate van de video stream. Hoe hoger des te meer data dus betere kwaliteit<br>
	-b:a bitrate van de audio stream<br>
	-f flv: exporteer het als flv bestand<br>
	rtmp://server-ip-adress:port/applicatie/stream_key<br>
	is de server adress waar de content heen gestuurd moet worden. stream key kan willekeurig zijn maar moet wel onthouden worden zodat er bekend is hoe de video stream ontvangen kan worden.<br>

# Client:
	De client bestaat uit een html pagina met de libary hls.js als video player.
	Deze wordt html pagian wordt op de nginx server geplaatst om vanuit daar gehost te worden. de locatie waar deze geplaatst moet worden is:
	/usr/local/nginx/html

	De html pagina ziet moet de volgende inhoudt hebben(is ook op github te vinden: https://github.com/gitHub-Randy/Converter/blob/master/player.html):

	<!DOCTYPE html>
	<html lang="en">
	<head>
    		<meta charset="UTF-8">
    		<meta name="viewport" content="width=device-width, initial-scale=1.0">
   		<meta http-equiv="X-UA-Compatible" content="ie=edge">
    		<title>Axi-Cast</title>
	</head>
	<body>
    		<h2>Stream</h2>
    		<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    		<script src="node_modules/hls.js/src/controller/buffer-controller.js"></script>
		<!-- Or if you want a more recent canary version -->
		<!-- <script src="https://cdn.jsdelivr.net/npm/hls.js@canary"></script> -->
		<video id="video" muted="true" controls="true"></video>
		<script>
     			var config = {
        			debug: true,
  			};
			var video = document.getElementById('video');
  			if(Hls.isSupported()) {
    				var hls = new Hls(config);
    				hls.loadSource('http://127.0.0.1/hls/test.m3u8');
    				hls.attachMedia(video);
    				hls.on(Hls.Events.MANIFEST_PARSED,function() {
      				video.play();
				});
 			}
 			// hls.js is not supported on platforms that do not have Media Source Extensions (MSE) enabled.
 			// When the browser has built-in HLS support (check using `canPlayType`), we can provide an HLS manifest (i.e. .m3u8 URL) directly to the video element through the `src` property.
 			// This is using the built-in support of the plain video element, without using hls.js.
 			// Note: it would be more normal to wait on the 'canplay' event below however on Safari (where you are most likely to find built-in HLS support) the video.src URL must be on the user-driven
 			// white-list before a 'canplay' event will be emitted; the last video event that can be reliably listened-for when the URL is not on the white-list is 'loadedmetadata'.
 			else if (video.canPlayType('application/vnd.apple.mpegurl')) {
    				video.src = 'http://127.0.0.1/hls/test.m3u8';
   			
    				video.addEventListener('loadedmetadata',function() {
      					video.play();
    				});
  			}
		</script>
	</body>

</html>


Deze pagina kan dan op de client machine opgeroepen worden m.b.v. de URL: http://server-ip/player.html
FFMPEG moet wel streamen daarmee de strem op de html pagina afgespeeld kan worden. 





	

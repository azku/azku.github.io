---
layout: page
title: Txirrindularitza Proiektua
permalink: /txirrindularitza/
---

## Informazioa prozesatzea
Ubuntu zerbitzari instalazio batetik habiatuta, YOLOV8 modeloa erabiliz argazkien predikzioak egiteko ondorengo pausoak jarraitu behar dira.
Lehenengo eta behin, beharrezkoak diren pakete pare bat instalatu eta python ingurune bat sortu beharko dugu. Ingurunea sortzearen arrazoia, ``pip`` komandoaren erabilera da. Instalazio berrietan pythonen paketeak instalatzeko ingurune kontrolatuak sortzeko gomendia bultzatzen da:

{% highlight bash %}
sudo apt-get install -y libgl1-mesa-dev
sudo apt-get install -y libglib2.0-0
sudo apt install python3.12-venv
python3 -m venv .venv/

.venv/bin/pip install ultralytics
.venv/bin/pip install imutils
{% endhighlight %}


Ondorengo python kodeak, direktorio batean dauden argazkiak aztertu eta obketu detekzioa aplikatuko die, CSV batean emaitzak hutsiz.
Kode guztia [github.com/azku/txirrindularitza](https://github.com/azku/txirrindularitza) URLan aurkitu daiteke.


## Dokumentuak Igotzea
Probatzen gabiltzan gailuek, wifi sarera konektatzeko dauketen zailtasuna dela eta, hasierako bertsio baterako datuak webgune baten bitartez igotzea proposatu dugu.
Horretarako OVHCloud plataforman zerbitzari bat jarri dugu Ubuntu 24.04 Sistema Eragile batekin instalaturik.

Bertan Docker kontainerrak erabilita, Wordpress webgune bat ezarriko dugu eta hau editatu bertan fitxategiak igo ahal izateko.

Webgunea http://fptxurdinaga.in:8080/ helbidean jarri da eskuragarri.

Worpressen **Contact Form 7**, **Drag and Drop Multiple File Upload - Contact Form 7** eta **Flamingo**  pluginak instalatu dira fitxategiak zerbitzarira web bitartez igotzeko.
Lehenengo 2 pluginak Wordpressen inprimakiak sortzeko eta fitxategi anitz batera igotzeko aukera izateko erabili dira. Flamingo inprimaki bidalketa guztiak gordeta izateko erabili da.

## Worpressetik (Docker) datuak ateratzea
Datuak bi fasetan aterako dira. Alde batetik, iruadiak eskuz aterako dira docker komandoak erabilita.

{% highlight bash %}
docker cp wordpress:/var/www/html/wp-content/uploads/wp_dndcf7_uploads/wpcf7-files/*.png data/
{% endhighlight %}

Metadatuak, Flamengok Mysql datu-basean gordetzen ditu, eta hauek argazki fisikoekin lotzeko, python funtzio bat sortu da. Aurrez aipatu den bezala, kode guztia [github.com/azku/txirrindularitza](https://github.com/azku/txirrindularitza) URLan aurkitu daiteke. Fitxategi hau ``argazkien_metadatoak_ekarri.py`` deitzen da.

## Datuei begirada
Prozesamenduak emaitza bezala eskaintzen duen fitxategia estatistikoki interpretatzeko, Metabase sistema bat instalatu da. Sistema horretara CSV fitxategia inportatu eta aztertu daiteke.

Metabase martxan jartzeko Docker Compose erabili dugu. Hona hemen fitxategia:

{% highlight yaml %}
services:
  metabase:
    image: metabase/metabase:latest
    container_name: metabase
    hostname: metabase
    volumes:
      - /dev/urandom:/dev/random:ro
    ports:
      - 3000:3000
    environment:
      MB_DB_TYPE: postgres
      MB_DB_DBNAME: metabaseappdb
      MB_DB_PORT: 5432
      MB_DB_USER: metabase
      MB_DB_PASS: mysecretpassword
      MB_DB_HOST: postgres
    networks:
      - metanet1
    healthcheck:
      test: curl --fail -I http://localhost:3000/api/health || exit 1
      interval: 15s
      timeout: 5s
      retries: 5
  postgres:
    image: postgres:latest
    container_name: postgres
    hostname: postgres
    environment:
      POSTGRES_USER: metabase
      POSTGRES_DB: metabaseappdb
      POSTGRES_PASSWORD: mysecretpassword
    networks:
      - metanet1
networks:
  metanet1:
    driver: bridge
{% endhighlight %}

## Yolov modeloak hasierako emaitzak 
Yolov modelo gordinak lehenengo pausoko argazkietan nahiko emaitza onak lortzen ditu. Autobusak, kotxeak eta kamioiak estatistikoki bereiztea lortzen ditu.

## Matrikulak
Oinarrizko modeloetan ez dator matrikula identifikaziorako ahalmenik. Horretarako modeloa espezifikoki entrenatu beharra dago.
https://github.com/Ammar-Abdelhady-ai/Licence-Plate-Detection-and-Recognition-using-YOLO-V8

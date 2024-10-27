Ubuntu zerbitzari instalazio batetik habiatuta, YOLOV8 modeloa erabiliz argazkien predikzioak egiteko ondorengo pausoak jarraitu behar dira.
Lehenengo eta behin, beharrezkoak diren pakete pare bat instalatu eta python ingurune bat sortu beharko dugu. Ingurunea sortzearen arrazoia, ``pip`` komandoaren erabilera da. Instalazio berrietan pythonen paketeak instalatzeko ingurune kontrolatuak sortzeko gomendia bultzatzen da:

{% highlight bash %}
sudo apt-get install -y libgl1-mesa-dev
sudo apt-get install -y libglib2.0-0
sudo apt install python3.12-venv
python3 -m venv .venv/
{% endhighlight %}


Ondorengo python kodeak, argazki bat jaitsi eta yolov8m.pt aurre elikatutako modeloa erabilz, predikzioak burutzen ditu.

{% highlight python %}
import urllib.request
from ultralytics import YOLO


url = 'https://www.freecodecamp.org/news/content/images/2023/04/cat_dog.jpg'
file_name = 'cat_dog.jpg'
req = urllib.request.Request(url=url, headers={'User-Agent': 'Mozilla/5.0'})

with urllib.request.urlopen(req) as response, open(file_name, 'wb') as out_file:
    data = response.read() # a `bytes` object
    out_file.write(data)

model = YOLO("yolov8m.pt")

results = model.predict("cat_dog.jpg")
{% endhighlight %}

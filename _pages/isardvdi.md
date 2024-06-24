---
layout: page
title: IsardVDI
permalink: /isardvdi/
---

Zerbitzu eta aplikazioak birtualki sortzeko eta kudeatzeko plataforma libre bat da IsardVDI. Errekurtsoak era sinple eta eroso batean kudeatzea eskaintzen digu.

Honen funtzionamenduaren adibide bat eskainiko dut ondoren. Big Data sismetak azaltzeko, beharrezkoa izaten da hainbat datu iturri desberdinekin elkarlanean ibiltzea.

Horretarako, Linux sistema eragilea duen oinarrizko instalazio bat prestatuko dugu lehenik. Instalazio honen gainean docker jarriko dugu eta hau oinarri hartuta zerbitzu desberdinak prestatuko ditugu: Hadoop, Spark, Postgres DBKS bat, No SQL sistema bat...

Hasteko beraz archlinux instalazioarekin habiatuko gara. Lehenik eta behin, media atalean archlinuxen iso-a nondik eskuratu daitekeen esango diogu:

![Archlinux media sortu](/assets/images/isardvdi_1.png)

URL, izena eta deskripzioa sartu ondoren, media sortutzen da:
![Archlinux media sortu](/assets/images/isardvdi_2.png)

![Archlinux media sortu](/assets/images/isardvdi_3.png)

Orain media horretan oinarritutako mahaigain bat sortuko dugu. Horretarako mahaigain atalera joan eta bertan "Mahaingain berria" zapalduko dugu. Izena eta deskriopzioa jarri eta median sortutako archlinux ISOa aukeratu behar da:
![Archlinux media sortu](/assets/images/isardvdi_4.png)

Memoria, CPU eta gainontzeko aukeraketak egin eta jadanik prest izango da mahaigaina Archlinux instalazioa burutzeko.
![Archlinux media sortu](/assets/images/isardvdi_5.png)

Archlinux Isarden instalatzerakoan 3 gauza kontutan izatea komeni da. Lehenengoa dagokigun teklatu mota aukeratzea **loadkeys** erabilita. Bigarrena, **archinstall** scripta erabiltzea instalaziorako.

![Archlinux media sortu](/assets/images/isardvdi_6.png)

Azkenik, instalazioa burututakoan, abiotik  **CD/DVD** kendu behar da sistema instalaziotik abiatu dadin.

![Archlinux media sortu](/assets/images/isardvdi_7.png)

Behin Archlinuxen oinarrizko instalazioa burututakoan komenigarria da erabiltzaile bat sortu eta konpotik ssh bidez konektatzeko sshd aktibatzea. Horretarako openssh instalatu eta systemd erabilita aktibatu:
{% highlight bash %}
pacman -S openssh
systemctl start sshd
systemctl enable sshd
{% endhighlight %}

Kanpoko bezero batetik Wireguard erabilita VPN bidez konektatzeko, konektatu nahi dugun mahagainean **Wireguard VPN** sarea gehitu behar dugu.
![Archlinux media sortu](/assets/images/isardvdi_8.png)

Behin hau egin eta bezerotik mahaigainak adierazten digun IP helbidean ssh bidez konektatzeko aukera izango dugu.

Docker instalazioa...

Apache/Hadoop docker ofiziala erbiltzeko docker irudi ofiziala erabili dezakegu.
Bertan adierazten den konfigurazioa erabilita exekutatzerakoan arazoak izan ditugu "ulimits" delakoarekin. Konfigurazio fitxategian mugak aldatuta funtzionatzen du. Hona hemen adibide bat:

{% highlight config %}
services:
   namenode:
      image: apache/hadoop:3
      hostname: namenode
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["hdfs", "namenode"]
      ports:
        - 9870:9870
      env_file:
        - ./config
      environment:
          ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
   datanode:
      image: apache/hadoop:3
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["hdfs", "datanode"]
      env_file:
        - ./config      
   resourcemanager:
      image: apache/hadoop:3
      hostname: resourcemanager
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["yarn", "resourcemanager"]
      ports:
         - 8088:8088
      env_file:
        - ./config
      volumes:
        - ./test.sh:/opt/test.sh
   nodemanager:
      image: apache/hadoop:3
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["yarn", "nodemanager"]
      env_file:
        - ./config
{% endhighlight %}
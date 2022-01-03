---
layout: post
title:  "Datu-baseak - Ikuspegi praktiko bat"
author: azku
tags: [datubaseak]
categories: [datubaseak]
---

# 1. Datuak eskuratu eta inportatu
Informazio eta komunikazio garai hauetan saila da imaginatzen esparru desberdinetan sortu ohi diren datu kantitate eta motak. Hurrutira joan gabe ere, Euskal Autonomia Erkidegoko administrazio desberdinek biltzen dituzten datuak, Opendata Webgunean jasogarri daude. Mota desbernineko datu asko aurkitu ditzakegu bertan. Ziurtatuta dago norberaren intereseko zerbait aurkitzea.

Oraingo honetan, etxeko animaliei buruz dauden datuekin solasaldi bat egiteko pausuak deskribatuko ditut eta horrela, Datu-baseak eskaini ditzaketen gaitasun anitzak aztertuko ditugu.

Artikulu hau, X aritukuluz osatutako seriearen lehenengo atala da. Beste barik, hona hemen jarraitu beharreko pausuak.

Datuak eskuratzeko
[URL honetara](https://opendata.euskadi.eus/katalogoa/-/eaeko-animaliak-identifikatzeko-erregistro-orokorra-aieo/) joan beharko gara.

Hemen, modu desberdinetan taldekatutako datuak 2 formatutan eskeintzen dira: Excel eta CSV formatuan. Kasu honetan, CSV formatuan dauden fitxategietan gaude interesaturik. Mota honetako inportazioak egiten ditugunean, sarritan aurkituko ditugu datuak formatu honetan.

![Datuak eskuratzeko aukera desberdinak](/assets/images/EAEko_animaliak_identifikatzeko_erregistro_orokorra.png)

Behin datuak gure konputagailura jaitzi ditugunean, inportaziorako prest gaude.
Datu-basearen arabera, ondoren jarritu beharreko prosezua moldatu beharra dago.

Datu-basera inportatzeko orduan, zein DMBS edo Datu-base maneiatze sistemarekin arituko garen identifikatzea ezinbestekoa da. Orokorrean, merkatuan dauden datu-base produktu desberdinek, SQL lengoaia estandarra erabiltzen dute [^1]. Datu-base sistema komertzialak produktu konplikatuak izaten dira eta instalazioan eta exekuzioan errekurtso asko behar izaten dituzte. Hori dela eta, ikasle bakoitzak bere makinan probak egiteko aukera izateko zailtasunak izan ditzakegu.

Euskal Autonomi Erkidekogo hezkuntza sailak, Microsoft Windows eta Office instalatzen ditu konputagailu gehienetan eta hori dela eta, ikasleek eskuragarri izaten duten Datu-Base sistema bat Microsoft Access izaten da. Eskuragarritasunaz aparte badaude Access DBMS aldeko eta kontrako argudio ugari, teknikoetatik hasi eta filosofikoetara, ekonomikoetatik igarota. Artikulo honetan ez naiz puntu hauetaz mintzatuko, ikasleei zerbait praktikoa azkar erakusteko helburuarekin.

Beraz, hori dela eta, Accesen egingo dugu inportazioa eta horretarako Access ireki beharko dugu lehenik.

![Testu fitxategia inportatzen](/assets/images/access_inportazioa.png)

Aurreko irudian adierazten diren pausoak jarraituz, ondorengo pantaila azalduko zaigu:

![Testu fitxategia inportatzen 2](/assets/images/access_inportazioa_2.png)

Hemen fitxategian datuak nola banatzen diren adierazi beharko dugu. Gure kasuan, punto eta koma erabiltzen da datu mota bakoitza banatzeko. Hurrengo pantailan hori zehaztuko dugu:

![Testu fitxategia inportatzen 3](/assets/images/access_inportazioa_3.png)

Lehenengo lerroak zutabeen izenak dituenez, koadro hori aktibatu beharko dugu inportazioak kontutan hartu dezan.


Ondoren, inportazioak identifikatu dituen zutabe guztietatik igaro beharko gara eta bakoitzari ez indexatzeko adieraziko diogu eta datu mota testua dela. Ohitura ona izaten da inportazioetan dena testu bezala inportatzea. Horrela datu moten identifikazioarekin arazo gutxiago izango ditugu eta behin datuak datu basean daudela, datu mota eraldaketak egiteko aukera ahaltsuagoak izango ditugu, SQL lengoaiaren abantailak erabili bait ditzakegu.

![Testu fitxategia inportatzen 4](/assets/images/access_inportazioa_4.png)

Beti ere oinarrizko gakoa izatea interesgarria denez, Accessi esango diogu sortu dezala.

![Testu fitxategia inportatzen 5](/assets/images/access_inportazioa_5.png)

Eta listo! Inportazioarekin bukatzerakoan, animalien erregistroekin beteriko taula prest izan beharko genuke gure datu-basean.
2. Partean, inportatu berri ditugun datuak nola eraldatu eta moldatu azalduko dut. Aldaketak, bilaketetan abiadura eskainiko digute eta datuen konsistentzia trikontzea.

[^1]: Berez SQL lengoaiaren definizio estandar bat dago. Errealitatean ordea, produktu komertzial bakoitzak bere inplementazio aldaketak egiten dizkio horrela estandarretik "hurrunduz".

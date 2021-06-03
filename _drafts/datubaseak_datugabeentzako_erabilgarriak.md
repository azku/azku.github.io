---
layout: post
title:  "DatuBaseak - Ikuspegi praktiko bat"
author: azku
tags: [datubaseak]
categories: [datubaseak]
---

Euskal Autonomia Erkidegoko administrazio desberdinek biltzen dituzten datuak, Opendata Webgunean jasogarri daude. Mota desbernineko datu asko aurkitu ditzakegu bertan. Ziurtatuta dago norberaren intereseko zerbait aurkitzea.

Oraingo honetan, etxeko animaliei buruz dauden datuekin solasaldi bat egiteko pausuak deskribatuko ditut eta horrela, Datu-baseak eskaini ditzaketen gaitasun anitzak aztertuko ditugu.

# Datuak eskuratu
Datuak eskuratzeko
[URL honetara](https://opendata.euskadi.eus/katalogoa/-/eaeko-animaliak-identifikatzeko-erregistro-orokorra-aieo/) joan beharko gara.

Hemen, modu desberdinetan taldekatutako datuak 2 formatutan eskeintzen dira: Excel eta CSV formatuan. Kasu honetan, CSV formatuan dauden fitxategietan gaude interesaturik. Mota honetako inportazioak egiten ditugunean, sarritan aurkituko ditugu datuak formatu honetan.

![Datuak eskuratzeko aukera desberdinak](/assets/images/EAEko_animaliak_identifikatzeko_erregistro_orokorra.png)

Behin datuak gure konputagailura jaitzi ditugunean, inportaziorako prest gaude.
Datu-basearen arabera, ondoren jarritu beharreko prosezua moldatu beharra dago.

# Datu-basera inportatu
Datu-basera inportatzeko, Datu-base bat aukeratu beharra daukagu. Orokorrean, merkatuan dauden datu-base produktu desberdinek, SQL lengoaia estandarra erabiltzen dute [^1]. Datu-base sistema komertzialak produktu konplikatuak izaten dira eta instalazioan eta exekuzioan errekurtso asko behar izaten dituzte. Hori dela eta, ikasle bakoitzak bere makinan probak egiteko aukera izateko zailtasunak izan ditzakegu.

Euskal Autonomi Erkidekogo hezkuntza sailak, Microsoft Windows eta Office instalatzen ditu konputagailu gehienetan eta hori dela eta, ikasleek eskuragarri izaten duten Datu-Base sistema bat Microsoft Access izaten da.

Beraz, hori dela eta, Accesen egingo dugu inportazioa eta horretarako Access ireki beharko dugu lehenik.









[^1]: Berez SQL lengoaiaren definizio estandar bat dago. Errealitatean ordea, produktu komertzial bakoitzak bere inplementazio aldaketak egiten dizkio horrela estandarretik "hurrunduz".
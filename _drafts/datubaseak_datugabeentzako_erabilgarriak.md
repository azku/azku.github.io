---
layout: post
title:  "DatuBaseak - Ikuspegi praktiko bat"
author: azku
tags: [datubaseak]
categories: [datubaseak]
---

Euskal Autonomia Erkidegoko administrazio desberdinek biltzen dituzten datuak, Opendata Webgunean jasogarri daude.

Oraingo honetan, etxeko animaliei buruz dauden datuekin solasaldi bat egiteko pausuak deskribatuko ditut eta horrela, DatuBaseak eskaini ditzaketen gaitasun anitzak aztertuko ditugu.

# Datuak eskuratu
Datuak eskuratzeko ondorengo URL-ra joan beharko gara:
https://opendata.euskadi.eus/katalogoa/-/eaeko-animaliak-identifikatzeko-erregistro-orokorra-aieo/

![Datuak eskuratzeko aukera desberdinak](/assets/images/EAEko_animaliak_identifikatzeko_erregistro_orokorra.png)

# Datu basera inportatu
Datu basera inportatzeko, Datubase bat aukeratu beharra daukagu. Orokorrean, merkatuan dauden datubase produktu desberdinek, SQL lengoaia estandarra erabiltzen dute*. Orokorrean DatuBase sistema komertzialak produktu konplikatuak izaten dira eta instalazioan eta exekuzioan errekurtso asko behar izaten dituzte.

Hori dela eta, ikasle bakoitzak bere makinan probak egiteko aukera izateko zailtasunak izan ditzakegu. Euskal Autonomi Erkidekogo hezkuntza sailak, Microsoft Windows eta Office instalatzen ditu konputagailu gehienetan eta hori dela eta, ikasleek eskuragarri izaten duten DatuBase sistema bat Microsoft Access izaten da.

Beraz, hori dela eta, Accesen egingo dugu inportazioa eta horretarako Access ireki beharko dugu lehenik.




* Produktu bakoitzak bere SQL bertsioa erabiltzen du eta errealitatean desberdintasun txikiak daude konparaketan.
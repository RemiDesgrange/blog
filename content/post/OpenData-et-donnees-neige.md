---
title: "OpenData Et Donnees Neige"
date: 2019-11-02T18:34:50+01:00
draft: true
---

Je joue depuis quelques temps avec les données OpenData neige. J'ai un peu galéré, mais j'ai surtout rencontré des gens formidables qui ont pris le temps de m'expliquer tout ce que ne dit pas météofrance.

## De quoi on parle ?

Je vais parler des données neige dispo en opendata, ou presque, pour les données non opendata, au sens non réutilisable sans condition, je mettrais un encars.

On parle donc des données des:

* Bulletin Estimation Risque Avalanche ou BERA, souvent encore appelé BRA
* Poste nivo. Données de qualité car prélevé par des humains
* des données des stations de mesures comme le réseau Romma, les balises Flowcapt.

Il est à noter que les données des balises "nivo" ne sont *pas* disponible. Vous pouvez consulter les graphs mis a disposition par météofrance[^1] ex:
![Nivose hebdo bonneval. Du bonheur n'est-ce pas ?!](/images/BONNE_hebd.gif)

Mais pour récupérer les données brute, il vous en coutera :

* 65€ de frais de mises en services
* 376€ par an et par balises nivose
* 1560€ de frais de fourniture de données.

Autant pour les 376€ de fourniture, il y a du travail, de la maintenance etc... Par contre 1500 boules pour envoyer aussi peu de données... C'est les tarifs minitel...

[^1]: Je crois qu'il y a consensus pour dire que ces graphs sont moches et illisibles.


## Où récupérer les données 

Deux options :

* FTP météo france : [ftp.meteo.fr](ftp://ftp.meteo.fr). Limité à 20 connexions simultanées, vous permet de parcourir les données, mais pour de la récup batch, c'est... aléatoire.
* Le site [OpenData de météo france](https://donneespubliques.meteofrance.fr)

Il semble que météofrance fasse de l'opendata dans le cadre de sa mission de service publique. Vu la qualité des docs et du site, j'ai la lègère impression que c'est contraint et forcé qu'ils ouvrent leur données.

### Données BRA (Bulletin risque avalanche)

Dispo sur le FTP dans `ftp://ftp.meteo.fr/FDPMSP/Pdf/BRA/`. On notera le nom du dossier `Pdf` qui n'a rien à voir avec la choucroute.

Vous avez à disposition les BRA depuis 2016. Soit au format PDF (bon pour la consultation). Soit au format XML. Vous pouvez reconstituer le BRA au format HTML avec le fichier `BRA.xslt`. Exemple en python avec `lxml`

```python
import lxml.etree as ET

xslt = ET.parse("BRA.xslt")
bra = ET.parse("BRA.MAURIENNE.2019010100000.xml")
transform = ET.xslt(xslt)
bra_as_html = transform(bra)
```

Le fichier `BRA.css` vous est nécéssaire dans le cas où vous générez le BRA via XSLT. Les images également.

Vous avez également le fichier `massifs.json` qui listes les massifs concerné par le BRA. Les massifs sont groupé par départements. J'ai recensé deux soucis avec ce fichier :

* Il y a une catégorie `autres` qui ne sert à rien (pas de BRA dans le vaucluse...)
* il y a des massifs avec des fautes d'orthographes, genre `Embrunnais`. Et pas d'ID. Les massifs sont identifié `OPPXX` avec `XX` un numéro de massifs. Dans tout les documents vous avez cet ID, sauf dans `massifs.json` dommage.

On notera aussi qu'il est impossible d'avoir la date du dernier BRA. Contrairement au données poste Nivo. Si on *scrape* les données BRA, on est donc obligé de faire en mode *essai-erreur*.
Pour obtenir le nom du fichier BRA du jour, il faut récupérer le fichier `bra.%Y-%m-%d.json`, qui, pour chaques massifs, donne l'heure de sortie du BRA. Cette heure vous permet de reconstituer le nom du fichier `BRA.MAURIENNE.<heure_dans_le_json>.xml`. 


Enfin dernière chose si vous souhaitez récupérer les BRA de manière programmatique, chercher ce qui vous interesse via le FTP, et requètez via [donneespubliques.meteofrance.fr](https://donneespubliques.meteofrance.fr). Exemple. Si vous souhaitez récupèrer la feuille XSLT :
* sur le FTP : `ftp://ftp.meteo.fr/FDPMSP/Pdf/BRA/BRA.xslt`
* sur le site web : `https://donneespubliques.meteofrance.fr/donnees_libres/Pdf/BRA/BRA.xslt`

Pour les json journaliers :

`https://donneespubliques.meteofrance.fr/donnees_libres/Pdf/BRA/bra.%Y%m%d.json`

Idem pour les BRA. C'est très pratique. Par contre, si votre fichier est introuvable, le serveur vous renvoie un code 302, qui vous redirige vers la page d'erreur "fichier non trouvé", servi avec un code HTTP... 200. Du grand art 👏.



### Données poste Nivo

## Remerciement
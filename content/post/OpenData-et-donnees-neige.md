---
title: "Open data et données neige"
date: 2019-11-02T18:34:50+01:00
draft: false
---

Je joue depuis quelques temps avec les données OpenData neige. J'ai un peu galéré, mais j'ai surtout rencontré des gens formidables qui ont pris le temps de m'expliquer tout ce que ne dit pas Météo France.

## De quoi on parle ?

Je vais parler des données neige disponible en open data, ou presque, pour les données non open data, au sens non réutilisable sans condition, je mettrais un encart.

On parle donc des données:

* Bulletin Estimation Risque Avalanche ou BERA, souvent encore appelé BRA
* Poste nivo. Données de qualité car prélevé par des humains
* des données des stations de mesures comme le réseau Romma, les balises Flowcapt.

Il est à noter que les données des balises "nivo" ne sont *pas* disponible. Vous pouvez consulter les graphs mis a disposition par Météo France[^1] ex:
![Nivose hebdo bonneval. Du bonheur n'est-ce pas ?!](/images/BONNE_hebd.gif)

Mais pour récupérer les données brute, il vous en coutera :

* 65€ de frais de mises en services
* 376€ par an et par balises nivose
* 1560€ de frais de fourniture de données.

Autant pour les 376€ de fourniture, il y a du travail, de la maintenance etc... Par contre 1500 boules pour envoyer aussi peu de données... C'est les tarifs minitel...

[^1]: Je crois qu'il y a consensus pour dire que ces graphs sont moches et illisibles. Par contre si un expert en ComputerVision est capable de récupérer les données brutes depuis ces images *àlacon* je prend (l'explication **et** le code 😊)


## Où récupérer les données 

Deux options :

* FTP météo france : [ftp.meteo.fr](ftp://ftp.meteo.fr). Limité à 20 connexions simultanées, vous permet de parcourir les données, mais pour de la récupération batch, c'est... aléatoire.
* Le site [OpenData de météo france](https://donneespubliques.meteofrance.fr)

Il semble que Météo France fasse de l'open data dans le cadre de sa mission de service publique. Vu la qualité des docs et du site, j'ai la légère impression que c'est contraint et forcé qu'ils ouvrent leur données.

## Données BRA (Bulletin risque avalanche)

Disponible sur le FTP dans `ftp://ftp.meteo.fr/FDPMSP/Pdf/BRA/`. On notera le nom du dossier `Pdf` qui n'a rien à voir avec la choucroute.

Vous avez à disposition les BRA depuis 2016. Soit au format PDF (bon pour la consultation). Soit au format XML. Vous pouvez reconstituer le BRA au format HTML avec le fichier `BRA.xslt`. Exemple en python avec `lxml` :

```python
import lxml.etree as ET

xslt = ET.parse("BRA.xslt")
bra = ET.parse("BRA.MAURIENNE.2019010100000.xml")
transform = ET.xslt(xslt)
bra_as_html = transform(bra)
```

Le fichier `BRA.css` vous est nécessaire dans le cas où vous générez le BRA via XSLT. Les images également. 

Il est a noter que Météo France ne fourni pas de XSD pour les BRA.

Vous avez également le fichier `massifs.json` qui listes les massifs concerné par le BRA. Les massifs sont groupé par départements. J'ai recensé deux soucis avec ce fichier :

* Il y a une catégorie `autres` qui ne sert à rien (pas de BRA dans le vaucluse...)
* il y a des massifs avec des fautes d'orthographes, genre `Embrunnais`. Et pas d'ID. Les massifs sont identifié `OPPXX` avec `XX` un numéro de massifs. Dans tout les documents vous avez cet ID, sauf dans `massifs.json` dommage.

On notera aussi qu'il est impossible d'avoir la date du dernier BRA. Contrairement au données poste Nivo. Si on *scrape* les données BRA, on est donc obligé de faire en mode *essai-erreur*.
Pour obtenir le nom du fichier BRA du jour, il faut récupérer le fichier `bra.%Y-%m-%d.json`, qui, pour chaque massifs, donne l'heure de sortie du BRA. Cette heure vous permet de reconstituer le nom du fichier `BRA.MAURIENNE.<heure_dans_le_json>.xml`. 


Enfin dernière chose si vous souhaitez récupérer les BRA de manière programmatique, chercher ce qui vous intéresse via le FTP, et requêter via [donneespubliques.meteofrance.fr](https://donneespubliques.meteofrance.fr). Exemple. Si vous souhaitez récupérer la feuille XSLT :

* sur le FTP : `ftp://ftp.meteo.fr/FDPMSP/Pdf/BRA/BRA.css`
* sur le site web : `https://donneespubliques.meteofrance.fr/donnees_libres/Pdf/BRA/BRA.css`

Pour les json journaliers :

`https://donneespubliques.meteofrance.fr/donnees_libres/Pdf/BRA/bra.%Y%m%d.json`

Idem pour les BRA. C'est très pratique. Par contre, si votre fichier est introuvable, le serveur vous renvoie un code 302, qui vous redirige vers la page d'erreur "fichier non trouvé", servi avec un code HTTP... 200. Du grand art 👏.

![Schema à l'arrache du process de fichier non trouvé chez Météo France](/images/Meteo_France_not_found_non_sense.png)

Exemple en python avec `requests` :

```python
import requests

r = requests.get('https://donneespubliques.meteofrance.fr/donnees_libres/'
'Pdf/BRA/BRA.RENOSO-INCUDINE.20191102145846.xml', allow_redirects=False)
# n'utilisez pas le mot clé assert. Si le code est optimisé (python -O) 
# les instructions sont ignoré par l'interpréteur.
if (r.status_code != 200):
    raise AssertionError('Le bra est introuvable')
```

### Précaution d'usage

Maintenant que vous avez les données du BRA, vous voulez faire une *app* qui défonce. Mais attention, vous manipulerez des données **prévisonnelles**, résultat d'un analyse humaine forcement biaisé. Ne prenez pas le BRA pour parole d'évangile, et évitez les traitement automatique dessus, ce sont des données hypothétiques, vérifier sur le **terrain** et si vous doutez, renoncez, ou plutôt allez vers le plan B (car vous aviez prévu un plan B, non ?).

## Données postes Nivo

Les données sur les observations des postes nivo sont des données récoltées sur le terrain par 135 postes. À ces postes officies des pisteurs, les données postes nivo sont donc relativement fiables, elles ne sont pas une prévision mais un état au jour le jour du manteaux neigeux. Malheureusement les données sont souvent incomplètes.

Les données sur les postes nivo se trouve sur le ftp dans `ftp://ftp.meteo.fr/FDPMSP/Txt/Nivo/`. Les manip pour récupérer en http via `donneespubliques.meteofrance.fr` fonctionne également.

Première chose assez cool, il y a un fichier `lastNivo.js` qui vous donne la date du dernier fichier post nivo. Les enregistrements sont sous forme de CSV. Les enregistrements du mois sont en CSV, 1 CSV par jours. Plus anciens, les fichiers sont compacté par mois, et en gunzip (compressé).

La liste des stations est au format JSON dans le fichier `postesNivo.json`. On peut l'ouvrir directement dans un logiciel SIG (comme QGIS), car c'est au format `GeoJSON`. Chaque station à un ID, cet ID est utilisé dans les fichiers CSV pour identifier les stations.
Malheureusement il semble qu'il manque des stations, certain CSV ont des ID qui ne sont pas renseigné dans le fichier `postesNivo.json`. Certaines stations n'ont pas ID, ce qui les rends in-identifiables.

Passons maintenant aux données, le CSV. Quand vous avez un champ marqué `mq` cela veut dire `NULL`, case vide, pas de données.

Ensuite vous verrez plein de chiffre. Si on regarde la doc il y a des codes, mais que peuvent bien vouloir dire ces codes ? Et bien ce sont ceux que les pisteurs transmettent à Météo France, aucune description n'est faites de ces codes, j'ai dû demander des copies des livres de formations des pisteurs pour en avoir la signification. C'est clairement pas cool de la part de Météo France. 


## Données non Météo France

### Balises FlowCapt

Les balises FlowCapt sont une autres sources de données automatiques, géré par la société [IAV](http://iav-portal.com/index.php?nav=iodmisawlist&lang=en&search=&center=&sort_field=center&sort_asc=1), basé en suisse. Pour utiliser leur données vous devrez leur demander. Ils ont une API qui n'est pas documenté mais qui est assez simple à utiliser, même si les requêtes fonds mal à la tête.

Exemple : `http://www.isaw.ch/idod/idod.php?s=FCHE1&f=json&d=3` avec `s` qui est le code de la station et `d` la durée. Vous avez droit à 168h maximum de données, soit 7 jours.

### Balises Romma

Les données des balises Romma appartiennent au propriétaire de la balise, si vous voulez récupérer les données, il vous faudra passer un accord avec le propriétaire. C'est assez contraignant, je n'ai pas prévu de jouer avec leur données.

## Remerciement

Je tiens a remercier Météo France, au fond, si leur site web était moins moisi, je n'aurais pas investigué tout ceci. Merci à Aurélien, pour le lien du FTP Météo France qui m'a permis de farfouiller. Merci a Alain Duclos et à tout l'équipe de [data-avalanche.org](https://data-avalanche.org) d'avoir pris le temps de répondre à mes questions. Merci également à l'association Romma pour ses éléments.


*Stay tune* car il y a un *(not so) secret project* dans les cartons pour exploiter toute ces données.
---
title: "Pourquoi je n'aime pas GraceTHD"
date: 2018-02-20T20:46:58+01:00
draft: true
---

# Pourquoi je n'aime pas Gracethd

Fallait que ça sorte. [GraceTHD](http://www.avicca.org/content/gracethd-projets), c'est un modèle de données, à la base d'échange, entre les différent acteur des télécoms.  Ça à été concocté par l'[avicca](http://www.avicca.org). C'est très sérieux. 

Ce modèle devrait permettre à tous les acteurs des télécoms de s'entendre et de s'echanger des donnés cartographique sur leur réseaux sans peines. Use case : vous êtes un operateur (Orange, SFR, etc...), vous demandez une boite de TP de vous poser la fibre là, en bas de chez vous. Vous demandez une étude à un sous-traitant "bureau d'étude", qui va transmettre les fichiers au sous-traitant, qui vous transmettrea les données "recollé" une fois fini. La promesse c'est qu'il n'y ait plus de friction. En plus vous pourriez faire votre compta dedans, GraceImmo ça le fait non ?


# Ça partait bien

Franchement ça partait bien. Des solutions OpenSources, comme ça pas de frein économique à l'adoption. Des solutions éprouvés : 

* `Qgis`
* `postgis`
* `spatialite` : pour le test

Herbergé sur Github, une belle doc. Un redmine (ça c'est moins cool mais bon...)avec une super reactivité. Franchement, ça partait bien.

# Et puis tu regarde le "MCD"

Bon alors déjà un mcd c'est ça :
![MCD le vrais](images/MCD-levrais.jpg)

Je n'ai jamais trouvé un MCD chez GraceTHD. J'ai trouvé ça :

![MCD sauce gracethd](https://github.com/GraceTHD-community/GraceTHD-MCD/blob/master/docs/GraceTHD-MCD/gracethd-mcd-v2.0_schema_v01.png)

C'est jolie ça decrit bien les choses. Mais ce n'est _pas_ un MCD. Modèle physique ? genere le toi même pépére... Sinon regarde la feuille excel. 

# Et puis tu regarde les sources

[les sources](https://github.com/GraceTHD-community/GraceTHD-MCD/tree/master/sql_postgis)

Franchement ça partait pas si mal. A la base GraceTHD était censé être un modèle _d'échange_ donc au final même si le schema était un peu moche on s'en foutait, si on voulait un SI différent on le faisait et puis :

> Ce n'est pas un format, c'est un modèle de données. (page 7)
> Pas seulement pour l'échange, gestion patrimoniale (stockage, analyses, contrôles, etc.).
Depuis [avicca](http://www.avicca.org/document/17214/dl)  

Oui alors... Il y a plusieurs choses qui me chiffonne :

TODO
* c'est moche (example des tables toute plus dégeux les une que les autre)
* c'est limité a 10 charactères pour rester au siecle dernier (shapefile : link shapefilemustdie)
* c'est inconsistant (positon vs masque).
* 0 utilisation du moteur de la base. Grace-Check à la rescouse (ou pas)

# Conclusion

Oui j'ai une versionb "jacky" de GraceTHD, mais ma version est exploitablke au jour le jour pour :

* de l'étude
* de l'exploitation
* de la gestion
* de l'analytics 







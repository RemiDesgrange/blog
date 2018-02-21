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
![MCD le vrais](/images/MCD_levrais.jpg)

Je n'ai jamais trouvé un MCD chez GraceTHD. J'ai trouvé ça :

![MCD sauce gracethd](https://raw.githubusercontent.com/GraceTHD-community/GraceTHD-MCD/master/docs/GraceTHD-MCD/gracethd-mcd-v2.0_schema_v01.png)

C'est jolie ça decrit bien les choses. Mais ce n'est _pas_ un MCD. Modèle physique ? genere le toi même pépére... Sinon regarde la feuille excel. 

# Et puis tu regarde les sources

[les sources](https://github.com/GraceTHD-community/GraceTHD-MCD/tree/master/sql_postgis)

Franchement ça partait pas si mal. A la base GraceTHD était censé être un modèle _d'échange_ donc au final même si le schema était un peu moche on s'en foutait, si on voulait un SI différent on le faisait et puis :

> Ce n'est pas un format, c'est un modèle de données. (page 7)

> Pas seulement pour l'échange, gestion patrimoniale (stockage, analyses, contrôles, etc.).
Depuis [avicca](http://www.avicca.org/document/17214/dl)  

Franchement ça partait bien, parce que je suis d'accord sur le principe :

> •  Ce n'est pas un outil! Le COPIL ne veut pas d'outils, c'est seulement un modèle de données.
> •  A chacun de choisir, créer, implémenter ses outils. (p. 9)

Trop cool !! Malgré tous cela. Il y a je trouve de gros soucis dans l'implémentation technique de gracethd.

# Aucune notion de perf

Quand vous avez un réseau vous vous retrouvez vite avec plusieurs centaine de câbles, qui représente plusieurs dizaine de milliers de fibres, et donc de connexions entrent elles. Dans toute les tables, la clé primaire est un varchar, souvent de taille 255. Je suis franchement pas fan des SERIAL (voir [le blog de clever cloud](https://www.clever-cloud.com/blog/engineering/2015/05/20/why-auto-increment-is-a-terrible-idea/)) mais le `varchat(255)` m'a tué... c'est très peut maintenable. Vous pouvez tapez sur google `Can I have varchar PK` _tous_ les postes vous dirons non (ex : [mailing list pg](https://www.postgresql.org/message-id/5A03DE8B-8FF6-4CFE-95AD-D9019437462C%40decibel.org)). Il y aurait au moins pu y avoir une contrainte sur la non-présence de charactère dans ces champs `varchar`. Ça partait pas si mal...

# Très peux de contrainte

Le modèle relationnelle est un truc qui à relativement fait ses preuves. Ce schema viole à peu près toute les formes normales. Exemple : [la table position](https://github.com/GraceTHD-community/GraceTHD-MCD/blob/master/sql_postgis/gracethd_30_tables.sql#L707-L723) vous pouvez avoir une position lié à une cassette ou à un tiroir, le `ou` étant exclusif.

Je ne critique pas trop ce choix. J'aurais mis trois table pour la `position`, mais passons. On pourrait imaginer d'implémenter cette contrainte directement dans le moteur avec 
`CHECK (ps_cs_code IS NULL OR ps_ti_code IS NULL)`. Je reproduis ici la réponse qui ma été faite sur le [redmine de gracethd](https://redmine.gracethd.org/redmine/issues/431) :

> La réponse générale est assez simple. Au plus on met de contraintes sur une base au plus ça va devenir compliqué de l'utiliser, surtout dans un cadre où les utilisateurs que sont les collectivités ne sont pas les producteurs des données. Prenons le cas des dates de modif. Pour une quelconque raison technique le format date livré par l'entreprise a un soucis. Ca veut dire qu'on bloque l'import pour une erreur, certes importante mais potentiellement non bloquante. Un contrôle a posteriori est plus souple.

> GraceTHD-MCD n'est pas mis à disposition pour la production de données. Chacun a la charge de mettre en place sa chaine de production avec les technologies qu'il souhaite. Toutefois un bureau d'étude peut aisément adapter l'implémentation du modèle pour assurer une meilleure qualité de sa production en ajoutant des contraintes.

> Pour une réponse plus nuancée, dans GraceTHD-Check le catalogue de points de contrôles (`t_ct_cat`) prévoit un attribut `ct_sqlcheck` pour identifier qu'un point de contrôle serait pertinent à implémenter directement en Check. Pour l'instant aucun n'a été pointé, mais il y en a potentiellement pas mal. Les prochains ajouts de points de contrôle seront peut-être l'occasion d'en spécifier.


# Vivre au 16ème siècle

Si vous regardez les tables, vous pouvez voir que le nom des colonnes est limité à 10 caractères. Mais pourquoi donc ? Et bien pour supporter le format shapefile. Ce format créé par ESRI est dépassé.

* les attributs ne peuvent pas faire plus de 10 caractère
* une couche par fichier
* pas un fichier `.shp` mais `.shp`, `.dbf`, `.shx`, `.sbn`, `.sbx`, etc.........

Il existe même un site contre le shapefile : [Shapefile must die](http://switchfromshapefile.org).

Le vrais problème ici c'est qu'il existe une solution : GeoPackage.
 
# Consistance a géometrie variable

TODO ajouter 2 schema, position  et masque

* c'est inconsistant (positon vs masque).

* 0 utilisation du moteur de la base. Grace-Check à la rescouse (ou pas)

# Conclusion

Oui j'ai une version "jacky" de GraceTHD, mais ma version est exploitable au jour le jour pour :

* de l'étude
* de l'exploitation
* de la gestion
* de l'analytics 


Tous n'est pas si grave. Ce standard n'est pas si mauvais, mais je ne le trouve pas adapté à une utilisation dans la vie réel. 







---
title: "Pourquoi je n'aime pas GraceTHD"
date: 2018-02-20T20:46:58+01:00
draft: true
---

Fallait que ça sorte. [GraceTHD](http://www.avicca.org/content/gracethd-projets), c'est un modèle de données, à la base d'échange, entre les différents acteurs des télécoms.  Ça à été concocté par l'[avicca](http://www.avicca.org). C'est très sérieux. 

Ce modèle devrait permettre à tous les acteurs des télécoms de s'entendrent et de s'echanger des donnés cartographiques sur leurs réseaux sans peines. Use case : vous êtes un operateur (Orange, SFR, etc...), vous demandez à une boite de TP de vous poser la fibre là, en bas de chez vous. Vous demandez une étude à un sous-traitant "bureau d'étude", qui va transmettre les fichiers au sous-traitant TP, qui vous transmettrea les données "recollé" une fois fini. La promesse c'est qu'il n'y ait plus de friction. En plus vous pourriez faire votre compta dedans, GraceImmo ça le fait non ?


# Ça partait bien

Franchement ça partait bien. Des solutions OpenSources, comme ça pas de frein économique à l'adoption. Des solutions éprouvés : 

* `Qgis`
* `postgis`
* `spatialite` : pour le test

Herbergé sur Github, une belle doc. Un redmine (ça c'est moins cool mais bon...) avec une super reactivité. Franchement, ça partait bien.

# Et puis tu regarde le "MCD"

Bon alors déjà un mcd c'est ça :
![MCD le vrais](/images/MCD_levrais.jpg)

Je n'ai jamais trouvé un MCD chez GraceTHD. J'ai trouvé ça :

![MCD sauce gracethd](https://raw.githubusercontent.com/GraceTHD-community/GraceTHD-MCD/master/docs/GraceTHD-MCD/gracethd-mcd-v2.0_schema_v01.png)

C'est jolie ça decrit bien les choses. Mais ce n'est _pas_ un MCD. 

# Et puis tu regarde les sources

[les sources](https://github.com/GraceTHD-community/GraceTHD-MCD/tree/master/sql_postgis)

Franchement ça partait pas si mal. A la base GraceTHD était censé être un modèle _d'échange_ donc au final même si le schema était un peu moche on s'en foutait, si on voulait un SI différent on le faisait et puis :

> Ce n'est pas un format, c'est un modèle de données. (page 7)

> Pas seulement pour l'échange, gestion patrimoniale (stockage, analyses, contrôles, etc.).
Source: [avicca](http://www.avicca.org/document/17214/dl)  

Franchement ça partait bien, parce que je suis d'accord sur le principe :

> •  Ce n'est pas un outil! Le COPIL ne veut pas d'outils, c'est seulement un modèle de données.

> •  A chacun de choisir, créer, implémenter ses outils. (p. 9)

Trop cool !! Malgré tous cela. Il y a je trouve de gros soucis dans l'implémentation technique de gracethd.

# Aucune notion de perf

Quand vous avez un réseau vous vous retrouvez vite avec plusieurs centainew de câbles, qui représente plusieurs dizaines de milliers de fibres, et donc de connexions entrent elles. Dans toutes les tables, la clé primaire est un `varchar`
, souvent de taille 255. Je suis franchement pas fan des SERIAL (voir [le blog de clever cloud](https://www.clever-cloud.com/blog/engineering/2015/05/20/why-auto-increment-is-a-terrible-idea/)) mais le `varchat(255)` m'a tué... c'est très peut maintenable. Vous pouvez tapez sur google `Can I have varchar PK` _tous_ les postes vous dirons non (ex : [mailing list pg](https://www.postgresql.org/message-id/5A03DE8B-8FF6-4CFE-95AD-D9019437462C%40decibel.org)). Il y aurait au moins pu y avoir une contrainte sur la non-présence de charactère dans ces champs `varchar`. Dans la vrais vie, il est courant d'avoir une clé primaire "technique" (`serial`, `uuid`, etc..) et un id business (avec contrainte `UNIQUE`. Mais pas là, parce que #Yolo. C'est con ça partait pas si mal...

# Très peux de contraintes

Le modèle relationnelle est un truc qui à relativement fait ses preuves. Ce schema viole à peu près toute les formes normales. Exemple : [la table position](https://github.com/GraceTHD-community/GraceTHD-MCD/blob/master/sql_postgis/gracethd_30_tables.sql#L707-L723) vous pouvez avoir une position lié à une cassette ou à un tiroir, le `ou` étant exclusif.

Je ne critique pas trop ce choix. J'aurais mis trois table pour la `position`, mais passons. On pourrait imaginer d'implémenter cette contrainte directement dans le moteur avec 
`CHECK (ps_cs_code IS NULL OR ps_ti_code IS NULL)`. Je reproduis ici la réponse qui ma été faite sur le [redmine de gracethd](https://redmine.gracethd.org/redmine/issues/431) :

> La réponse générale est assez simple. Au plus on met de contraintes sur une base au plus ça va devenir compliqué de l'utiliser, surtout dans un cadre où les utilisateurs que sont les collectivités ne sont pas les producteurs des données. Prenons le cas des dates de modif. Pour une quelconque raison technique le format date livré par l'entreprise a un soucis. Ca veut dire qu'on bloque l'import pour une erreur, certes importante mais potentiellement non bloquante. Un contrôle a posteriori est plus souple.

> GraceTHD-MCD n'est pas mis à disposition pour la production de données. Chacun a la charge de mettre en place sa chaine de production avec les technologies qu'il souhaite. Toutefois un bureau d'étude peut aisément adapter l'implémentation du modèle pour assurer une meilleure qualité de sa production en ajoutant des contraintes.

> Pour une réponse plus nuancée, dans GraceTHD-Check le catalogue de points de contrôles (`t_ct_cat`) prévoit un attribut `ct_sqlcheck` pour identifier qu'un point de contrôle serait pertinent à implémenter directement en Check. Pour l'instant aucun n'a été pointé, mais il y en a potentiellement pas mal. Les prochains ajouts de points de contrôle seront peut-être l'occasion d'en spécifier.

## Le point débile est attribué à...

Ce monsieur qui écrit :

>Je ne pense pas que le typage soit un moyen d'éviter les saisies douteuses, je pense même que c'est un leurre dangereux puisque laissant croire que les données sont plus fiables.

>Je travaille dans le contrôle qualité des données FTTH depuis 10 ans, il est impossible de prendre le contenu d'un format pour argent comptant.

>Ce n'est pas parce qu'un champ est numérique qu'un utilisateur ne pourra pas saisir une longueur de -133, un nombre de fibres de 0. qu'il ne pourra saisir que Oui ou Non, qu'il ne se trompera pas.Forcer Oui/Non ne permet pas de gérer le "non renseigné", donc non connu.Ex : pt_secu = "N" (si on ne peut mettre que O/N) est interprété comme "non sécurisé" alors que l'on peut ne pas connaitre la réponse. Un booléen serait mieux sous forme de VARCHAR + REFERENCES vers O/N/null comme utilisé à de nombreux endroits dans le format (voire même autorisé d'autres cas imprévus...).
Source: https://redmine.gracethd.org/redmine/issues/159

\#Facepalm... ça pourrait être marrant. Je préfère en rire. Mais ce genre de personnes travaillent. Bon ok "travaillent" est peut-être pas le mot. "S'occupent" dirons nous. Et ça casse tous... 

* SI on le typage garantie que tu rentre pas de la merde
* NON on ne peut pas rentrer -133 dans un champ longueur SEULEMENT SI on défini le.... _drum roll_ 🎉 bon type !! En PG :
`CHECK (mon_champ_longueur > 0)`. Ou avec n'importe quel language typé : `unsigned int`. Ces gens ont l'esprit _fucké_ complet par les shapefiles. Mettons les à morts (les shapefiles hein !)

# Qualité ou Kalitay ?

Des vues sont généré pour plus de facilité, cool, et puis [les sources](https://github.com/GraceTHD-community/GraceTHD-MCD/blob/master/sql_postgis/gracethd_61_vues_elem.sql) :
```SQL
CREATE VIEW "vs_elem_bp_sf_nd" AS
SELECT 
  * 
FROM 
  gracethd.t_ebp,
  gracethd.t_suf, 
  gracethd.t_noeud
WHERE 
  t_ebp.bp_sf_code = t_suf.sf_code AND
  t_suf.sf_nd_code = t_noeud.nd_code;
```
Sérieux ? les jointures vous connaissez ? un bon nombre de vue matérialisé sont généré. Si le modèle avait été fait correctement, _aucune_ de ces vues matérialisé n'auraient étés nécéssaires. a vous les cronjob pourrie et autre job rundeck à maintenir. TODO, run de grace check avec un 2 PEV, 1 pour voir le plan pourrie sans join et celui avec un join

# Vivre au 16ème siècle

Si vous regardez les tables, vous pouvez voir que le nom des colonnes est limité à 10 caractères. Mais pourquoi donc ? Et bien pour supporter le format shapefile. Ce format créé par ESRI est dépassé.

* les attributs ne peuvent pas faire plus de 10 caractère
* une couche par fichier
* pas un fichier `.shp` mais `.shp`, `.dbf`, `.shx`, `.sbn`, `.sbx`, etc...

Il existe même un site contre le shapefile : [Shapefile must die](http://switchfromshapefile.org).

Le vrais problème ici c'est qu'il existe une solution : GeoPackage.
 
# Consistance a géometrie variable

TODO ajouter 2 schema, position  et masque

* c'est inconsistant (positon vs masque).

* 0 utilisation du moteur de la base. Grace-Check à la rescouse (ou pas)


# Accès libre... oui mais, enfin, non, mais c'est compliqué

J'ai demandé [ici](https://redmine.gracethd.org/redmine/issues/447) pourquoi l'accès en lecture au redmine n'était pas permis. Je suis obligé de coller la réponse ici, puisque vous n'avez pas accès :

>C'est bien évidemment une question qui s'était posée au moment de mettre cela en place. Les raisons principales qui me reviennent sont les suivantes.

>Ce n'est pas évident pour tout le monde de prendre le clavier pour s'exprimer par exemple sur des difficultés rencontrées. Un accès réservé à des professionnels inscrits qui se connaissent souvent plus ou moins limite cet effet.
>Des choses écrites ici même restent ici-même. Pas d'archive.org et autres Google qui archivent des propos tenus un moment donné pour les siècles à venir. Accessoirement on peut apporter des rectifications, l'erreur est humaine.
>Le but est de constituer un réseau de professionnels qui coopèrent autour de GraceTHD, et donc de pouvoir s'identifier entre acteurs. Nous incitons donc à utiliser votre véritable identité professionnelle, et notamment pour pouvoir gérer les groupes d'utilisateurs. Personnellement je considère que pour de l'accès public sur Internet il est fortement préférable d'utiliser un pseudo, mon identité et ma vie professionnelle sont privées, j'essaie de choisir avec qui et sur quelle plateforme de confiance je partage cela.
>On limite l'impact des machines qui indexent ou qui cherchent à caser du spam. Même en privé j'ai régulièrement des comptes bidons qui tentent de s'inscrire pour caser du spam.
>C'est aussi un moyen d'inciter à vous inscrire, un accès public limiterait cela. Le but n'est pas de gonfler le nombre de membres ou de collecter des infos persos. Le but concerne surtout les annonces. Chacun a la possibilité de régler ce qu'il souhaite recevoir comme mail de notifications (pour les demandes, etc.), mais il nous semble important de pouvoir tenir au courant les personnes concernées avec le système d'annonces lorsqu'il y a une information importante. Bref, constituer une communauté.
>Il n'est pas évident de constituer une telle communauté d'utilisateurs qui ne sont pas toujours habitués à cela, cette plateforme nous y aide. Mais rien n'est immuable. S'il y a besoin d'un espace d'échanges publics, ça peut s'envisager, mais à mon sens les échanges réalisés jusqu'à aujourd'hui dans un contexte "privé", doivent le rester.

>Pour le besoin précis d'accès aux collaborateurs, il est rapide de se créer un compte. Sinon j'ai également déjà validé des comptes "collectifs" bien identifiés (BE_machin, etc.), c'est donc une solution possible, mais ce n'est pas une solution que je recommande car les gens ne savent plus vraiment qui gère et qui a dit quoi.

>S'il y a un véritable besoin, ce que je peux peut-être également faire c'est un groupe "consultation" pour mettre des comptes d'entreprise en consultation. Si une entreprise souhaite partager un compte en consultation avec des collaborateurs tout en contrôlant qui intervient au nom de l'entreprise, vous pouvez par exemple faire un compte [entreprise]\_consult, je comprendrai.

Notez qu'on aurait pu utiliser les PR et les issues Github. Mais le neticien est nul en informatique, c'est bien connu.

# Groupe Expert 

Si vous êtes arrivé jusqu'ici (franchement bravo !) vous êtes au courant : j'aime pas trop trop gracethd. Ce qui m'énerve le plus dans ce standard ce sont les groupes d'expert. Je ne pense pas qu'ils soient nuls, mais je pense que c'est trop léger. Exemple [ici](https://redmine.gracethd.org/redmine/issues/221) ou encore [là](https://redmine.gracethd.org/redmine/issues/395). 

* Dans la version 2.0.1, un référence récurisive à été enlevée car ça induissait des problèmes "parents/enfants". 
* La notions de NULL (SQL) ne serait pas toujours la bonne
* les boolean ne serait pas assez expréssif.
* et ça continue quasi à l'infini.

Je ne suis pas expert SQL. Mais si j'avais juste un conseil : acheter vous un putain de bouquin `SQL 101` ou `Postgresql pour les nul(l)`.

* les contraintes ça se "[défer]()https://www.postgresql.org/docs/current/static/sql-set-constraints.html" ou ça se (déactive temporairement)[https://www.postgresql.org/docs/current/static/sql-altertable.html]  (à l'import). 
* NULL veut dire NULL. Point.
* _have you ever heard of [`ENUM`](https://www.postgresql.org/docs/current/static/datatype-enum.html)_ ?

# Conclusion

Oui j'ai une version "jacky" de GraceTHD, mais ma version est exploitable au jour le jour pour :

* de l'étude
* de l'exploitation
* de la gestion
* de l'analytics 


Tous n'est pas si grave. Ce standard n'est pas si mauvais, mais je ne le trouve pas adapté à une utilisation dans la vie réel. 







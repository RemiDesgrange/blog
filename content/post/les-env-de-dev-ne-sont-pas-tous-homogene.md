---
title: "Les env de dév ne sont pas tous homogène"
date: 2023-02-27T16:50:53+01:00
draft: true
tags: ["opinion"]
category: [""]
image: ""
description: "Réaction a un article de Mathieu Corbin"
lang: fr
---

J'ai récemment lu un article de Mathieu Corbin, dont j'apprécie beaucoup le blog, sur les [env de dév](https://www.mcorbin.fr/posts/2023-02-17-environnement-dev/). Je pense que Mathieu a eu une _fatigue_[^1]. Et je le comprend. Mais je voudrais néanmoins revenir sur ces propos qui paraisse, au mieux un vœux pieux, au pire mensonger.

[^1]: comme dans [javascript fatigue](https://medium.com/@ericclemmons/javascript-fatigue-48d4011b6fc4)

Je dév principalement en python, cela depuis un paquet d'année. Je pense que l'expérience de dév en python n'est pas digne de l'usage qu'il a. C'est complètement claqué au sol tellement c'est de merde. Tu as d'abord à gérer un virtualenv, ou alors ton gestionnaire de dépendances/projets va le faire pour toi, mais tous sont incompatibles, certain utilise un fichier `.txt` (généralement `requirements.txt`) ce fichier étant parfois généré via un fichier `.in` (`pip-tools`) ou alors `Pipfile` qui, malgré le fait que ce soit "standard" n'est supporté que dans `Pipenv`, qui est d'une lenteur sur un "véritable" projet. il y a `pyproject.toml` mais chaque tools utilise ses propres champs, et j'ai pas encore parlé de la [PEP-582](https://peps.python.org/pep-0582/) avec le `__pypackages__`.

Je crois que c'est en grande partie la raison pour laquel Docker est si populaire pour faire du python. La complexité d'un setup de projet (et j'ai pas parlé des extensions nécessitant un compilateur C...) est juste énorme. 

Bref revenons au cœeur du sujet: les envs de dév locaux sont trop complexe.

Certe. J'ai récemment lu sur Twitter quelqu'un qui disait qu'a partir du moment ou tu as une org de dév conséquente, avoir un dév sénior qui est là _uniquement_ pour faire en sorte de diminuer la fraction sur les env de dév/déploiement est souhaitable/rentable. Et j'abonde dans ce sens.

Néanmoins on ne fait pas tous du web. On ne fait pas tous du backend.

Imaginons que vous développiez un serveur SMTP[^2]. Moi je veux bien avoir un setup _simple_ mais comment je fais des tests e2e, voir mieux, de la simulation ? Parce que quand on déploie des services en réseau on peut ignorer les IO pour partie "métier", mais pour la plomberie on fait comment ?

[^2] j'ai dis "_imaginez_" ! j'ai pas dis que c'était réel :-)

Autre cas, vous developpez un nouveau type de clavier révolutionnaire. Vous faites donc du "dév embarqué". Franchement si vous arrivez a setup une dévboard en 1 commande, là vous avez tué le game.

Grace à Arduino (et d'autre) le dév embarqué s'est considérablement amélioré, il reste du chemin à parcourir mais a un moment donnée, il faudra brancher la board a votre ordi (ça fait 1 étape 
 ;-)

## Conclusion

Ma conviction c'est qu'éffectivement un env de dév simple est souhaitable. Un env de dév suivant les standards de votre language préféré est tout autant souhaitable. Mais tout les projets ne sont pas égaux, notamment les projets "bas niveau" et pour ceux-là, il me semble que la compléxité soit inérante au(x) problème(s) traité(s).

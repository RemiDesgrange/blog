---
title: "Nivo, histoire d'un ras Le bol"
date: 2019-07-25T21:38:30+02:00
draft: true
---

TL; DR: un nouveau site pour consulter les BRA + les balises nivoses + des couches météos diverse et
varié est dispo ici : https://nivo.desgran.ge

# Il etait une fois

Je fais pas mal de ski, et surtout du ski de rando. Et pour ceux qui pratique cette discipline, il y
a généralement un rituel : la météo. Que ce soit les prévisions, les webcams ou le BRA [^1].

On passe donc pas mal de temps a surfer sur [cette page](). Je sais pas vous mais sur PC, c'est déjà
difficile à utiliser, mais alors sur smartphone c'est carrement la guerre ouverte avec les pub, et
le site franchement mal fichu.

[^1]: Bulletin Risques Avalanches, normalement on appelle même ça un BERA pour Bulletin  Estimation
  des Risques d'Avalanches. Mais tout le monde a gardé l'ancien nom.

# Essai 1

J'ai donc essayé ce truc : [bra-scrapper](https://github.com/RemiDesgrange/bra-scrapper) (oui on dit
scraper je sais...)

Je voulais un fonctionnement sans base de données et puis ça marchais moyen. J'y ai passé du temps
pour un résultat franchement moyen. Mais j'ai appris à me servir de `scrapy`. Ce qui est toujours ça de
pris 😊.


# Essai 2

Et puis j'ai découvert le site [opendata de météofrance](https://donneespubliques.meteofrance.fr).
Et là révélation, on pouvait avoir accès au BRA depuis 2016, tout les enregistrements nivose depuis
2010, _brut_ c'était la folie.

## Enfin presque...

Ouais presque la folie. J'avais la données. Sauf que j'avais pas la doc... Par exemple pouvez vous
me dire a quoi correspond le chiffre "5" si on parle de couverture nuageuse ? Vous peut-être mais
pas moi.
{{< figure src="/images/description_by_mf.png" caption="Je clair Luc, ne pas?" >}}

Il y a quand même des docs, mais je ne félicite pas MétéoFrance pour la qualité de celle-ci.


# Introducing: Nivo

J'ai donc codé [Nivo](https://nivo.desgran.ge). Un site qui vous permet un accès simple, sans les
infames pubs, et avec plusieurs infos superposé. Mobile-friendly [^2]

[^2]: enfin on essai...


Pour les plus téméraires (ou développeur) si vous avec des remarques de suggestions, vous pouvez
faire une _issue_ sur le [github du projet](https://github.com/RemiDesgrange/nivo)

Je remercie tout ceux qui, de près où de loin on contribué a ce bout de code. TODO add pindrow,
dwarf etc...

Le BRA est ajouté tout les jours, entre 16h et 17h.

La listes principales des fonctionnalités :

* Visu du BRA par massifs
* Visu des balises nivoses par balises. Echelles de temps ajustable (oui vous pouvez remonter
  jusqu'en 2010...)
* Visu de la météo.
* Visu des pentes > 30° (IGN)

Bonus :

* Pour le BRA je reprend exactement celui de météofrance. Pour les techos, je prend le XML des BRA,sur
lequel j'applique un XSLT et ça génére le HTML (identique a celui de MF).
* Lorsque vous regardez un BRA précédent. l'appli affiche si la prévision de risque (stable, à la
  hausse/baisse) était juste ou non.







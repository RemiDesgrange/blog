---
title: "4G Et Teletravail"
date: 2020-03-17T09:26:12+01:00
draft: false
tags: ["Télétravail", "4G", "Réseaux mobile"]
categories: ["Télétravail", "4G", "Réseaux mobile", "français"]
---

Ça y est, c'est la guerre. Vous êtes confiné chez vous, et votre ADSL 
commence à faire sacrement la gueule, entre vos gamins qui passent la journée 
sur Youtube/Netflix/Whatever et vous qui devez pousser cette satanée image 
docker de 2.8Gio (don't...). Et puis il se peut aussi que votre ADSL soit tellement moisi
que même une recherche sur Google est lente (#BeenThereDoneThat).

## Mais déjà pourquoi mon ADSL est plus mauvaise que d'hab

A cause de la [paradiaphonie](https://fr.wikipedia.org/wiki/Diaphonie) sur 
la ligne, vous pouvez remercier vos voisins. Si vous avez la fibre plus de soucis, puisque 
ce qui circule dans une fibre optique, c'est de la lumière.

## Première étape : où que c'est qu'elle est mon (mes) antenne(s)

Pour connaitre les antennes près de chez vous, ~~composez le 3637~~ allez sur [cartoradio](https://www.cartoradio.fr).

Enlevez toutes les antennes non utiles (TV, Radio, Autres stations, 2G, 3G)

![Séléction dans cartoradio](/images/selection-cartoradio.png)

> *Ceci n'est _absolument_ pas une science exacte, c'est le réseau mobile qui décide sur
> qu'elle antenne vous émettez, pas votre terminal !* 
Mais quand on se trouve à la campagne, on a peu de chance de se tromper 😀. Notez aussi que votre téléphone peut dialoguer avec plusieurs antennes (ce qu'on appelle MIMO, Multiple Input Multiple Output).

Notez aussi que si votre antenne dispose de faisceaux hertziens cela veux dire que
l'adduction (le machin qui amène le débit) est fait non par une fibre mais par un lien
radio. On arrive aujourd'hui a des liens radios assez performants (notamment chez SFR)
mais ça saturera plus vite qu'une fibre.

Notez également que plus la fréquence est basse, moins il y aura de slots pour vous. Vous
aurez moins de débit à 800 MHz qu'à 1600 MHz (à charge comparable). Sachez également que
les terminaux récents agrégents les bandes. En gros si vous êtes à la campagne et que vous
avez que du 800MHz, brace yourself...

## Deuxième étape : est-ce que je la vois ?

Pour ceux d'entre nous qui habite en montagne ou en zone avec du relief [^1], encore
faut-t-il avoir la visu sur l'antenne. Vous pouvez vérifier sur carto radio la hauteur du
mat ou du pylônes (différent en fonction de votre opérateur). Notez cette valeur.

[^1]: Si vous habitez dans la Beauce, vous pouvez donc passer...

Allez ensuite sur [airlink](link.ui.com). Ce site n'a _rien_ à voir avec la 4G, il va
juste vous permettre de voir si vous avez une vue directe vers l'antenne. Le cône de
Fresnel sera faux (pas les mêmes fréquences) le débit indiqué aussi.

C'est à ce moment qu'il faudra renseigner la hauteur du pylône que vous avez vu sur
cartoradio sur un des équipements (peu importe). 

Sachez aussi, que le signal rebondit, donc ce n'est pas parce que vous n'avez pas de vu
direct que vous n'aurez pas de signal, vous avez bien la 4G en ville sans pour autant
avoir une vue direct sur les antennes.

## Troisième étape : on passe à la pratique

Là il va falloir vous balader dans la maison. Si vous utilisez un smartphone, ça va être
pénible pour plusieurs raisons :

* le téléphone change d'antenne en permanence en fonction des injonctions du réseau ;
* il change également de bande de fréquences. Même si votre téléphone ne bouge pas (déso...) ;
* il est assez complexe de bloquer son téléphone sur une bande de fréquence données
  (bande 700, 800, 1800, 2600 MHz)

Vous pouvez utiliser plusieurs produits pour tester, fast.com, speedtest, nperf. Je ne recommande pas
fast.com car c'est un produit netflix, or il se peut qu'en ces temps troublés, les
interco Netflix soit un brin saturé.

Pour ceux qui ont de vieilles bâtisses avec de gros mur n'hésitez pas à tester "au bord de
la fenêtre". Dans un logement précédent j'avais ~20Mbps derrière la fenêtre, 45Mbps 30cm
plus loin sur le balcon. C'est pas une solution pérenne, mais pour cette satanée image
docker trop lourde, pour quelques minutes ça peut le faire (et ça aérera votre pièce 😉)

## Quatrième étape : achetez vous un routeur 4G

Si ça fonctionne à peut près chez vous, et que ça vous permet de bosser, vous pouvez
commencer à vous demander si une box 4G serait pas mal. Il existe des offres des
opérateurs (Free, Bouygues notamment, je ne suis pas exhaustif) avec des tarifs assez
élevés. Perso j'ai pris un Huawei B715-23c (le même que free) Il est assez cher mais il
vous permet de coller une antenne externe (pratique pour les vieilles bâtisses).
L'avantage de l'acheter c'est aussi que vous pouvez switcher de carte sim si un opérateur
vous fait faux bond (genre SFR en rade, je switch avec la sim de mon téléphone Orange et
ça remarche). 

En ce qui concerne les antennes externes, j'ai testé pour vous [ceci](https://www.amazon.fr/gp/product/B00UBCCQOA/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1). Spoiler ça
fonctionne pas 😀

Pour les questions, direction [twitter](https://twitter.com/DesgrangeRemi).

---
title: "Decryptage des codes des données poste Nivo"
date: 2019-11-29T21:08:16+01:00
draft: false
---


Suite à l'article sur les [données neiges en open-data]({{<relref "OpenData-et-donnees-neiges.md">}}) J'ai réussi à mettre la main sur les codes que fournissent les pisteurs lors de relevés sur un poste Nivo. Données disponible en opendata [chez Météo France](https://donneespubliques.meteofrance.fr/?fond=produit&id_produit=94&id_rubrique=32)

Il y a un code que je n'ai pas réussi à retrouver, si certain savent, qu'ils se manifestent ! par exemple [sur twitter](https://twitter.com/DesgrangeRemi)


| Nom | Colonnes dans le fichier CSV | Valeurs possibles | Description |
|-----|------------------------------| ----------------- | ----------- |
|Temps Présent| `ww` | 0 | Aucun des phénomènes suivants à la station au moment de l’observation : brouillard pluie neige orage |
| | | 44 | Brouillard mais ciel visible |
| | | 45 | Brouillard ciel invisible |
| | | 48 | Brouillard déposant du givre mais viel visible |
| | | 49 | Brouillard déposant du givre ciel invisible |
| | | 16 | Chute de pluie en vue, mais pas à la station |
| | | 60 | Pluie faible intermittente |
| | | 61 | Pluie faible continue (sans interruption depuis le début) |
| | | 63 | Pluis modérée |
| | | 65 | Pluie forte |
| | | 67 | Pluie se congelant |
| | | 69 | Pluie et neige mêlées |
| | | 81 | Averse(s) de pluie |
| | | 16 | Chute de neige en vue, mais pas à la station |
| | | 36 | Chasse neige à la station |
| | | 70 | Neige faible intermittente |
| | | 71 | Neige faible continue |
| | | 73 | Neige modérée |
| | | 75 | Neige forte |
| | | 84 | Averse(s) de neige mêlée de pluie |
| | | 86 | Averse(s) de neige |
| | | 88 | Averse(s) de grésil ou de neige roulée |
| | | 17 | Orage entendu mais pas de précipitation à la station |
| | | 95 | Orage avec pluie ou neige à la station |
| | | 96 | Orage avec grésil ou grêle à la station |
| Temps passé | `w1`, `w2`| 0 |Aucun phénomène significatif |
| | | 3 | il y a eu de la chasse-neige à la station |
| | | 4 | il y a eu du brouillard |
| | | 6 | il y a eu de la pluie, de la bruine… (eau liquide) |
| | | 7 | il a neige, grêlé |
| Type de nuages à l’étage inférieur | `cl` | 0 | Pas de nuage bas |
| | | 2 | Cumulus |
| | | 5 | stratocumulus |
| | | 6 | stratus |
| | | 9 | cumulonimbus |
| | | / | station dans le brouillard, ciel invisible |	
| Type de nuages à l’étage moyen | `cm`| 0 |Pas de nuage moyen |
| | | 1 | Altostratus |
| | | 2 | Nimbostratus |
| | | 3 | Altocumulus |
| | | 4 | Altocumulus lenticulaire |
| | | 7 | Altocumulus altostratus |
| | | / | les nuages moyens sont invisibles |
| Type de nuages à l’étage supérieur | `ch` | 0 | Pas de nuages élevés |
| | | 2 | Cirrus |
| | | 7 | Cirrostratus |
| | | 9 | Cirrocumulus |
| | | / | Les nuages élevés sont invisibles |
| Nuage dans la vallée | `nuage_val` | 0 |Pas de nuage dans la vallée |
| | | 1 |Nuage isolées inférieur à 1000m |
| | | 2 |Nuage isolées entre 1000 et 1500m |
| | | 3 |Nuage isolées supérieur à 1500m |
| | | 4 |Mer de nuage partielle inférieur à 1000m |
| | | 5 |Mer de nuage partielle entre 1000 et 1500m |
| | | 6 |Mer de nuage partielle supérieur à 1500m |
| | | 7 |Mer de nuage complete inférieur à 1000m |
| | | 8 |Mer de nuage complete entre 1000 et 1500m |
| | | 9 |Mer de nuage complete supérieur à 1500m |
| | | / |Observation impossible (station dans le brouillard) |
| Chasse neige en altitude | `chasse_neige` | 0 | Pas de chasse neige |
| | | 1 | Il y a de la chasse-neige depuis la dernière observation mais pas actuellement
| | | 2 | Chasse neige modérée d’Est |
| | | 3 | Chasse neige modérée de Sud |
| | | 4 | Chasse neige modérée d’Ouest |
| | | 5 | chasse neige modérée de Nord |
| | | 6 | Chasse neige forte d’Est |
| | | 7 | Chasse neige forte de Sud |
| | | 8 | Chasse neige forte d’Ouest |
| | | 9 | Chasse neige forte de Nord |
| | | / | Observation impossible (nuage ou brouillard) |
| Description de l’avalanches observée | `aval_descr` | | Je rien trouvé dans le guide qui correspondent à ce libellé. |
|Genre d’avalanches | `aval_genre` | 0 |Rien à signaler. Pas d’avalanches, ni de coulées, ni de fissures |
| | | 1 | Aucune avalanches mais fissure(s) dans le manteau neigeux |
| | | 2 | Coulées sèches ou humides |
| | | 3 | Avalanche(s) de neige récente, sèche, départ ponctuel |
| | | 4 | Avalanche(s) de neige récente, humide, départ ponctuel |
| | | 5 | Avalanche(s) de plaque friable (départ linéaire, neige sèche, dépôt plutôt fin) |
| | | 6 | Avalanche(s) de plaque dure de surface (départ linéaire, neige sèche dépôt de blocs) |
| | | 7 | Avalanche(s) de surface de vieille neige humide ou mouillée |
| | | 8 | Avalanche(s) de plaque de fond de neige sèche (départ linéaire) |
| | | 9 | Avalanche(s) de fond de vieille neige humide ou mouillée (départ ponctuel ou  linéaire) |
| | | / | Inconnu |
| Altitude de départ de l’avalanche | `aval_depart` | 0 | Rien à signaler |
| | | 1 | Inférieur à 1500m |
| | | 2 | Entre 1500 et 1750m |
| | | 3 | Entre 1750 et 2000m |
| | | 4 | Départ à plusieurs altitudes mais la plupart inférieur à 2000m |
| | | 5 | Entre 2000 et 2250m |
| | | 6 | Entre 2250 et 2550m |
| | | 7 | Entre 2500 et 3000m |
| | | 8 | Supérieur à 3000m |
| | | 9 | Départ à plusieurs altitudes mais la plupart supérieur à 2000m |
| | | / | Inconnu |
| Exposition de l’avalanche | `aval_expo` | 0 | Rien à signaler |
| | | 1 | La plupart en versant Nord-Est |
| | | 2 | La plupart en versant Est |
| | | 3 | La plupart en versant Sud-Est |
| | | 4 | La plupart en versant Sud |
| | | 5 | La plupart en versant Sud-Ouest |
| | | 6 | La plupart en versant Ouest |
| | | 7 | La plupart en versant Nord-Ouest |
| | | 8 | La plupart en versant Nord |
| | | 9 | Aucune orientation privilégié |
| | | / | Inconnu |
| Type de grains de surface | `grain_predom`, `grain_nombre` | 1 | Neige fraiche |
| | | 2 | Particule reconnaissables |
| | | 3 | Grains fins |
| | | 4 | Faces planes |
| | | 5 | Gobelets |
| | | 6 | Grains ronds |
| | | 7 | Croûtes |
| | | 8 | Givre de surface |
| | | 9 | Neige roulée |
| Homogénéité de la couche | `homogeneite` | 0 | Il a neigé plus de 5cm depuis la dernière observation. Carottage vertical sur la planche |
| | | 1 | Il n’a pas neigé (ou moins de 5cm) et la couche de 10cm sous la surface est homogène (une seule strate). Carottage horizontal entre la surface de la neige et le niveau -10cm |
| | | 2 | Il n’a pas neigé (ou moins de 5cm) et la couche de 10cm sous la surface est constitué d’une ou plusieurs strates de nature ou de dureté différente. Pas de mesure de masse volumique |
| Phénomène Special | `phenspeN` | | Je n'ai pas trouvé d'indicateur dans le guide |
| Etendue cche nuageuse 1 | `nnuage1` | | Je n'ai pas trouvé d'indicateur dans le guide |

Personnellement je trouve que ces données "bruts" sont difficillement utilisable par le chalan. Je vais essayer de batir des visus qui permettent de mieux comprendre les indicateurs. Ce sera mon passe temps de cet hiver 😀. Je trouve regrétable que Météo France diffuse des données sans en révéler la nature des données libérées.

Bon Ski !
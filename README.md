# COG simplifié
Version simplifiée et plus accessible du Code Officiel Géographique pour les communes (source : INSEE).

Vous trouverez ici une version du COG pour les communes, publiée dans sa plus simple expression : le libellé complet de chaque commune et son code INSEE, sous la forme de fichiers simples d'usages (le format CSV dans un premier temps).

Voici les 4 premières lignes du fichier à titre d'exemple :

|DEPCOM|ARTMIN_NCCENR
| ---- | ----
|01001|L'Abergement-Clémenciat
|01002|L'Abergement-de-Varey
|01004|Ambérieu-en-Bugey

## Téléchargement
* COG simplifié 2017 : [COG_communes_INSEE.2017.csv](https://cdn.rawgit.com/CharlesNepote/COG/62e930e3/COG_communes_INSEE.2017.csv)

## Licence
Ces données Open Data sont sous Licence Ouverte : https://www.etalab.gouv.fr/licence-ouverte-open-licence

## Motivation
Quel est le référentiel officiel des communes françaises ? Le COG bien sûr !
Le Code Officiel Géographique, produit par l'INSEE, recense toutes les communes de France et identifie chacune par un code, appelé "code INSEE".

C'est une donnée extrêmement basique pour de très nombreux usages. Elle est notamment accessible par le portail de données ouvertes de l'état (http://data.gouv.fr). Cette donnée est si basique et essentielle qu'elle fait aujourd'hui parti du Service public de la donnée.
Hélas, la version produire par l'INSEE pose certains problèmes d'usage :
* il faut reconstituer le code commune “entier” (de type 01001) à partir du code département (01) et du code commune court (001)
* le nom de la commune est séparée en deux lorsqu’il y a un article (Le),Mans (deux champs) au lieu de "Le Mans"

J’ajoute, mais c’est presque secondaire, que le jeu est en ISO-8859-1, zipé, avec l’extension .txt et dans un format ésotérique : le standard CSV pour de la donnée de référence, c’est pas un minimum ?

## Source des données
Les données proviennent de la version officielle et estampillé du Code Officiel Géographique, produites par l'INSEE et éditée par la plateforme data.gouv.fr.
Ces données sont diffusées ici : https://www.data.gouv.fr/fr/datasets/code-officiel-geographique-cog/

## Process de production
Les données de l'INSEE ne font l'objet d'aucune modification de fond. C'est seulement leur mise en forme qui est ici retravaillée.
Les données sont ainsi traitées :
* téléchargement depuis le site de l'INSEE et décompression
* supression de toutes colonnes hors DEP (code département), COM (code commune), ARTMIN et NCCENR
* fusion des colonnes DEP et COM pour obtenir le code commune complet
* fusion des colonnes ARTMIN "(L')" et NCCERN "Abergement" :
  * en supprimants les parenthèses entrantes,
  * en remplaçant les parenthèses fermantes par un espace
  * en supprimant l'espace après une apostrophe

Nous documentons ci-dessous les 6 commandes représentant l'ensemble du process. Il convient prélablement d'installer [csvkit](https://github.com/wireservice/csvkit). Ces commandes fonctionnent sous Linux et devraient fonctionner sous Mac OS.
```bash
wget https://www.insee.fr/fr/statistiques/fichier/2666684/comsimp2017-txt.zip ; unzip comsimp2017-txt.zip
csvclean -t -e CP1252 comsimp2017.txt # conversion en CSV standard, UTF-8
csvcut -c DEP,COM,ARTMIN,NCCENR comsimp2017_out.csv > c3.csv # sélection des colonnes suffisantes pour obtenir le code commune et le nom de commune
perl -n -a -F, -e 'print "$F[0]$F[1],$F[2],$F[3]"' c3.csv > c4.csv # Fusion des colonnes DEP et COM pour obtenir le code INSEE complet
perl -p -e "s/ARTMIN,NCCENR/ARTMIN_NCCENR/; s/,\(/,/g; s/\),/ /g; s/\' /\'/g; s/,,/,/g" c4.csv > COG_communes_INSEE.csv # Fusion des colonnes ARTMIN et NCCERN
rm comsimp2017_out.csv c3.csv c4.csv # suppression des fichiers temporaires
```

## Contrôle qualité
Ce badge est le résultat d'un contrôle qualité indiquant si le fichier CSV est bien formé : [![goodtables.io](https://goodtables.io/badge/github/CharlesNepote/COG.svg)](https://goodtables.io/github/CharlesNepote/COG)

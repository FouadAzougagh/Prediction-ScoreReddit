Le fichier notebook se compose comme suit :

I- Bibliothèque et importation des données
II- Décomposition des colonnes pour obtenir des colonnes « atomiques »
III- Création de features à partir d'analyses textuelles
IV- Création de features à l’aide des réseaux de neurone
V- Tests et prédiction du modèle


Pour ce qui est des analyses effectuées, voici les explications :

La première étape a été de bien comprendre les features à disposition et d'en déduire de nouvelles. 

Ainsi, 

les colonnes « subreddit_id » et « subreddit » ont été supprimées, elles n'étaient composées que d’une seule classe et n’apportaient donc pas d’information pertinente à la prédiction.
la colonne « id » qui apportait les mêmes informations que la colonne « name » a été supprimée.

Ensuite, « created_utc » qui correspondait à une date a été décomposée en 3 colonnes pour voir si le message avait été écrit un week-end ou en semaine, 
un jour férié ou non en supposant qu’en jour férié et/ou un week-end plus de personnes étaient sur le réseau et donc que l’interaction était plus forte. 
On a fait la même chose avec les différents moments de la journée (tôt, matin, midi, après-midi, soir) en supposant que les messages postés tôt auraient moins de votes.


Pour les colonnes link_id et parent_id, la partie après le t1_ et t3_ est un nombre écrit en base 36 que l’on a converti en base 10, on a vu que ces valeurs étaient 
séquentiellement crées, on pouvait ainsi voir les posts et auteurs les plus anciens.

Comme certaines valeurs de link_id, parent_id et author apparaissaient plusieurs fois, on a gardé les valeurs qui apparaissaient plus de n fois et mis les autres dans la 
catégorie « autre » les jugeant comme un bruit dans nos données.
On garde ainsi les posts ayant eu le plus d’interaction et les auteurs étant les plus actifs en espérant que leurs notes soient proportionnelles.

Pour la première partie concernant les réseaux de neurones, on a construit deux graphes différents :

Le premier liant « id » et « parent_id » afin de travailler sur les liens entre les différents commentaires et leur réponses respectives. 
Les identifiants étant uniques, si un auteur répond deux fois à un autre auteur mais sur deux posts différents, on ne pourra pas le prendre en compte de cette façon.

On a donc réfléchit à un second réseau où on a lié les auteurs entre eux afin de ne pas perdre cette information.
On a choisi arbitrairement de créer une colonne similaire à Parent_id où l’id serait remplacé par le nom de l’auteur si on le connaissait, sinon on a laissé le lien tel quel.
On a créé une autre colonne similaire en mettant les id qui n’ont pas changés dans une classe « non renseigné » afin de conserver le principe de lier des auteurs.

On a alors calculé les centralités "degree", "closeness", "eigenvalue" et "katz" de ces graphes ainsi que le pagerank et les hubs et autorités. On pouvait ainsi prendre en compte la 
« popularité » des nœuds et leur attraction dans notre analyse.


Pour la deuxième partie concernant l’analyse textuelle, on a commencé par nettoyer la feature body sur 3 niveaux :

- Le premier niveau où on a enlevé la casse et les ponctuations pour ne garder que du texte uniforme et permettre l’analyse textuelle.
- Le deuxième niveau où on garde les « mots clés » en supprimant les mots vides pour ainsi garder seulement les mots pertinents.
- Le dernier niveau où l’on utilise l’algorithme de stemming : « LancasterStemming » pour ne garder que la racine des mots et ainsi considérer deux mots à terminaisons 
différentes comme un seul mot.

C’est sur ce dernier niveau que nous avons réalisé le tf-idf, on évite ainsi les mots vides et de considérer deux vecteurs pour un même mot à terminaisons différentes.
A partir de la matrice tf-idf on a utilisé la méthode NMF afin de faire ressortir les topics les plus courant de ces commentaires.
A noter que pour chacune de ces méthodes on a appliqué la méthode des k-means sur plusieurs valeurs de k et trouver le nombre de k le plus pertinent, pour NMF on a 
trouvé 15 principaux topics.

On regardait aussi la moyenne par classe en estimant que si une classe dégageait une moyenne significativement différente des autres alors elle serait pertinente pour notre prédiction, à condition bien sûr que la classe contienne beaucoup de données.
Pour chaque body on a alors sélectionné le topic dans lequel il contribuait le plus et ainsi crée une nouvelle feature.

Pour la prédiction des données, les méthodes de classification ont été moins pertinentes que la méthode de régression GradientBoost que nous avons gardé. 
Nous avons également essayé une double prédiction sur les scores supérieurs à 50 car on a remarqué que ces valeurs étaient les moins bien prédites.

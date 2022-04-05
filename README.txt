Le fichier notebook se compose comme suit�:

I- Biblioth�que et importation des donn�es
II- D�composition des colonnes pour obtenir des colonnes ��atomiques��
III- Cr�ation de features � partir d'analyses textuelles
IV- Cr�ation de features � l�aide des r�seaux de neurone
V- Tests et pr�diction du mod�le


Pour ce qui est des analyses effectu�es, voici les explications :

La premi�re �tape a �t� de bien comprendre les colonnes et en d�duire des features. 

Ainsi, 

les colonnes ��subreddit_id�� et ��subreddit�� ont �t� supprim�es, elles n'�taient compos�es que d�une seule classe et n�apportaient donc pas d�information pertinente � la pr�diction.
la colonne ��id�� qui apportait les m�mes informations que la colonne ��name�� a �t� supprim�e.

Ensuite, ��created_utc�� qui correspondait � une date a �t� d�compos�e en 3 colonnes pour voir si le message avait �t� �crit un week-end ou en semaine, 
un jour f�ri� ou non en supposant qu�en jour f�ri� et/ou un week-end plus de personnes �taient sur le r�seau et donc que l�interaction �tait plus forte. 
On a fait la m�me chose avec les diff�rents moments de la journ�e (t�t, matin, midi, apr�s-midi, soir) en supposant que les messages post�s t�t auraient moins de votes.


Pour les colonnes link_id et parent_id, la partie apr�s le t1_ et t3_ est un nombre �crit en base 36 que l�on a converti en base 10, on a vu que ces valeurs �taient 
s�quentiellement cr�es, on pouvait ainsi voir les posts et auteurs les plus anciens.

Comme certaines valeurs de link_id, parent_id et author apparaissaient plusieurs fois, on a gard� les valeurs qui apparaissaient plus de n fois et mis les autres dans la 
cat�gorie ��autre�� les jugeant comme un bruit dans nos donn�es.
On garde ainsi les posts ayant eu le plus d�interaction et les auteurs �tant les plus actifs en esp�rant que leurs notes soient proportionnelles.

Pour la premi�re partie concernant les r�seaux de neurones, on a construit deux graphes diff�rents�:

Le premier liant ��id�� et ��parent_id�� afin de travailler sur les liens entre les diff�rents commentaires et leur r�ponses respectives. 
Les identifiants �tant uniques, si un auteur r�pond deux fois � un autre auteur mais sur deux posts diff�rents, on ne pourra pas le prendre en compte de cette fa�on.

On a donc r�fl�chit � un second r�seau o� on a li� les auteurs entre eux afin de ne pas perdre cette information.
On a choisi arbitrairement de cr�er une colonne similaire � Parent_id o� l�id serait remplac� par le nom de l�auteur si on le connaissait, sinon on a laiss� le lien tel quel.
On a cr�� une autre colonne similaire en mettant les id qui n�ont pas chang�s dans une classe ��non renseign頻 afin de conserver le principe de lier des auteurs.

On a alors calcul� les centralit�s "degree", "closeness", "eigenvalue" et "katz" de ces graphes ainsi que le pagerank et les hubs et autorit�s. On pouvait ainsi prendre en compte la 
��popularit頻 des n�uds et leur attraction dans notre analyse.


Pour la deuxi�me partie concernant l�analyse textuelle, on a commenc� par nettoyer la feature body sur 3 niveaux�:

- Le premier niveau o� on a enlev� la casse et les ponctuations pour ne garder que du texte uniforme et permettre l�analyse textuelle.
- Le deuxi�me niveau o� on garde les ��mots cl�s�� en supprimant les mots vides pour ainsi garder seulement les mots pertinents.
- Le dernier niveau o� l�on utilise l�algorithme de stemming�: ��LancasterStemming�� pour ne garder que la racine des mots et ainsi consid�rer deux mots � terminaisons 
diff�rentes comme un seul mot.

C�est sur ce dernier niveau que nous avons r�alis� le tf-idf, on �vite ainsi les mots vides et de consid�rer deux vecteurs pour un m�me mot � terminaisons diff�rentes.
A partir de la matrice tf-idf on a utilis� la m�thode NMF afin de faire ressortir les topics les plus courant de ces commentaires.
A noter que pour chacune de ces m�thodes on a appliqu� la m�thode des k-means sur plusieurs valeurs de k et trouver le nombre de k le plus pertinent, pour NMF on a 
trouv� 15 principaux topics.

On regardait aussi la moyenne par classe en estimant que si une classe d�gageait une moyenne significativement diff�rente des autres alors elle serait pertinente pour notre pr�diction, � condition bien s�r que la classe contienne beaucoup de donn�es.
Pour chaque body on a alors s�lectionn� le topic dans lequel il contribuait le plus et ainsi cr�e une nouvelle feature.

Pour la pr�diction des donn�es, les m�thodes de classification ont �t� moins pertinentes que la m�thode de r�gression GradientBoost que nous avons gard�. 
Nous avons �galement essay� une double pr�diction sur les scores sup�rieurs � 50 car on a remarqu� que ces valeurs �taient les moins bien pr�dites.
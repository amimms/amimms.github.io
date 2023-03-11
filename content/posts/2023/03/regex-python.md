---
title: "Python & Regex -- application pratique"
date: 2023-03-11T11:20:25+01:00
draft: true
author: ["Alexandre"]
---

Pendant un an, mes fiches de lecture étaient organisées sous forme de tableau Markdown, comme celui-ci :

```markdown
| Pages | Citation | Commentaire |
| - | - | - |
| 12 | citation quelconque | commentaire quelconque |
```

Aujourd'hui, j'ai cessé de faire ainsi, car je me suis rendu compte que ce n'était pas le plus efficace. Pas que ça ne l'était pas de manière absolue -- je m'en suis convenue -- mais en raison du processus de travail que j'applique aujourd'hui.

Le problème, c'est que plus d'une centaine de fiches de lectures étaient formatées de cette manière, et je trouvais cela plutôt gênant pour faire des recherches et retrouver les citations efficacement.

J'ai donc décidé de coder un petit script écrit en Python pour reformater l'intégralité de la centaine ++ de fiches que j'avais prises durant la période.

Il me semble d'abord important de justifier pourquoi je fais ce court billet sur quelque chose qui, de prime abord, n'a rien à faire dans la boîte à outil d'un juriste (1). Je présenterai ensuite la manière dont j'ai conçu le script et le script lui-même (2).

## Pourquoi s'intéresser à l'informatique ?

C'est déjà le quatrième billet sur ce blog et je pense que si un thème transparaît le plus, c'est bien celui de l'informatique. Les discussions, souvent interminables, qui naissent de l'actualité technologique (comme avec l'arrivée de ChatGPT récemment) ne font que confirmer l'importance des apports de l'informatique. 

Je ne parle ici pas uniquement de l'apport *structurel* et *global* que cela a sur les infrastructures, les institutions, le traitement des données à grande échelle. Je vise plutôt l'apport que cela peut avoir sur notre quotidien, pour automatiser, faciliter, accompagner des tâches qui peuvent être rébarbatives, fatigantes, et franchement parfois inutiles.

Dans le domaine de la recherche, je pense que Zotero est un excellent exemple de ce en quoi l'informatique peut être salvatrice. Je présenterai aussi, bientôt, la puissance que nous offrent certains outils pour organiser nos notes et réflexions.

Aujourd'hui, je prendrai un autre exemple, plus spécifique que celui de Zotero, mais tout aussi démonstratif de l'intérêt d'utiliser l'informatique comme aide de travail.

## Python et les Regex

Dans mon [billet sur Markdown]({{< ref "posts/2023/03/mardown-simplicite.md" >}}), j'ai mis en avant à plusieurs reprises que l'un des avantages de MD est le format des fichiers. Ici, je vais exploiter cet intérêt pour opérer une transformation d'ampleur et complètement automatisée des notes en utilisant deux outils :
- [Python](https://www.python.org/), un langage de programmation ;
- Les expressions régulières, aussi appelées Regex (pour *regular expression*), qui sont des formules permettant de retrouver tout type de contenu et de le capturer.

Voilà le problème initial :
- une centaine de fiches dans des documents séparés contenant chacun un ou plusieurs tableaux Markdown ;
- dans ces tableaux, des données que je souhaitais récupérer pour les reformater. 

Pour simplifier, le script que je souhaitais développer devait comporter trois phases :
1. l'extraction des données ;
2. le reformatage des données ;
3. la réécriture des données dans un nouveau fichier.

### Première étape : construire une expression régulière 

La première phase implique de récupérer les données dans les fichiers. Il fallait donc que je trouver une manière de dire à l'ordinateur que je souhaitais obtenir les données qui se trouvent à tel et tel endroit un nombre indéterminé de fois.

Pour accomplir cela, rien de mieux qu'une expression régulière. Ces suites de caractères spéciaux permettent de décrire très précisément un ensemble de caractère possible. 

> :writing_hand: Ex. La chaîne `[a-z]` désigne une lettre minuscule quelconque de l'alphabet. 
 
En plus d'indiquer quels sont les caractères que l'on recherche, on peut aussi demander à l'ordinateur de *récupérer* certains caractères, mots, phrases, dans une variable pour les réutiliser par la suite -- et non pas uniquement *remplacer* des caractères !

> :fire: Pour une introduction complète aux Regex, voir [ici](https://regexone.com/). 

En utilisant une expression de ce type, il est ainsi possible de dire à l'ordinateur que je souhaitais retrouver, dans mes fiches de lecture, toutes les citations ou tous les commentaires, ainsi que les numéros de page associés dans les tableaux, de stocker ces informations dans une variable puis de les resituer dans une chaîne de caractère redéfinie à mon souhait.

Après quelques essais infructueux, je suis parvenu à écrire une expression régulière convenable : 

```
^\|*\s*[\*]*(.*?)[\*]*\s[\*]*\|\s*(.*?)\|(.*)
```


Puisque l'objet de ce billet est essentiellement une démonstration de l'intérêt des connaissances en informatique, je ne rentrerai pas dans les détails des explications. Ce qu'il faut retenir, c'est que la formule permet de retrouver toutes les chaînes de caractères qui ressemblent à cela :

```markdown
| 12 | citations quelconques ou rien | commentaire quelconque ou rien |
```

### Deuxième étape : récupérer les données et les reformater

Une fois l'expression régulière formée, il faut récupérer les données et les reformater.

Pour cela, Python, un langage de programmation à la syntaxe plutôt facile à comprendre, est un choix relativement judicieux lorsqu'un non-programmeur s'improvise sorcier...

Python dispose de nombreux modules développés par la communauté des développeurs ; il est très polyvalent, bien documenté (il est donc "facile" de résoudre des bugs pour des erreurs communes), et très en vogue. 

Avec, il est aisé de manipuler des fichiers `.txt` afin de récupérer leur contenu et d'en créer des nouveaux.

Pour récupérer les données, j'ai utilisé le module `re`, qui permet de manipuler les Regex et de retrouver ce qu'on souhaite dans une chaîne de caractères donnés.

Pour reformater les données trouvées, il m'a fallu concevoir une fonction spéciale (`convertir(match_found)`) qui prend pour paramètre un objet créé par une fonction du module `re` dans lequel se trouve des variables contenant le texte que je souhaitais récupérer. 

J'ai utilisé un test sous forme de `if - elif - else` qui permet de distinguer toutes les formes possibles de la chaîne observée et y appliquer le format que je souhaite. Une fois que la chaîne est réécrite conformément à ce que je souhaite, la fonction *renvoie* (`return`) la valeur et s'arrête. 

En Python, cette fonction ressemble à ça :

```python
def convertir(match_found):
	if "Pages" in match_found.group(1).strip() or "---" in match_found.group(2).strip():
		return "\n"
	elif (match_found.group(2) is None) or (match_found.group(2) == "" or match_found.group(2) == " "):
		return "p. "+match_found.group(1).strip()+" - "+match_found.group(3).strip()+"\n"
	elif (match_found.group(2) is not None) and (match_found.group(2) != " " or match_found.group(2) != ""):
		return "> [!cite] p. "+match_found.group(1).strip()+"\n> "+match_found.group(2).strip()+"\n> - *"+match_found.group(3).strip()+"*\n"
	else:
		return "> [!cite] p. "+match_found.group(1).strip()+"\n> "+match_found.group(4).strip()+"\n"
```


### Troisième étape : réécrire la fiche

La dernière étape, après avoir établi la bonne manière de retrouver les chaînes de caractères qui m'intéressaient et les avoir reformatées, consiste à réécrire la fiche de lecture dans un fichier.

Il y a plusieurs stratégies pour cela. Afin d'éviter tout déboire lié à la suppression d'une note de lecture par inadvertance, j'ai préféré créer un nouveau fichier dans un dossier à part et ainsi conserver l'original. 

Le script fait donc les opérations suivantes :
1. Ouverture de la note originale en mode "lecture seule" 
2. Lecture et test de chaque ligne avec la Regex pour voir si la ligne lue contient la chaîne à changer ;
3. Si le test est réussi, la fonction définie plus haut est appelée afin de modifier et reformater la chaîne ;
4. Peu importe la réussite du test, la chaîne lue est insérée dans une variable à la suite des autres. Cela permet de *reconstituer* le fichier avec les éléments d'origine et les nouveaux ;
5. Une fois que tout le fichier est lu, que le script arrive à la fin de la boucle, on crée un nouveau fichier dans lequel sera insérée la chaîne de caractère entière contenue dans la variable.

Voilà à quoi cela ressemble en Python :

```python
with open(nom_fichier, encoding="utf-8") as f:
	match = ""
	for line in f:
	    match += re.sub(r"^\|*\s*[\*]*(.*?)[\*]*\s[\*]*\|\s*(.*?)\|(.*)", convertir, line)
	
	with open('results/new'+nom_fichier, 'w', encoding="utf-8") as fichier_resultat:
	    fichier_resultat.write(match)
```


## Conclusions et remarques finales

Sans doute cet exemple d'utilisation de l'informatique n'est pas le plus universel, j'en conviens. J'espère néanmoins que cela permettra de donner quelques idées sur le sujet et motiver certains ou certaines à s'intéresser un peu plus à ces sujets.

On pourrait, par ailleurs, m'accuser de *procrastination sophistiquée*. C'est vrai ; j'aurais pu me passer de tout cela et m’accommoder de l'ancien format de mes fiches de lecture. Mais je crois que ce genre d'exercices pratiques permettent de mieux comprendre les outils avec lesquels nous travaillons et d'être *plus ou moins* autonomes. Et puis, il y a une vraie forme de défi à accomplir cela, non ?







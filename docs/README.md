# Travaux d'Héraclès #5 : la&nbsp;biche&nbsp;de&nbsp;Cérynie
 
Démarrage : cloner ce *repository*.
{: .alert-info }
Puis lancer la commande 
```bash
composer install
```

## État des lieux du projet
La prochaine mission d'Héraclès est de capturer la fantastique biche de Cérynie, mais sans lui faire aucun mal au risque de provoquer la colère d'Artémis.

Pour ce nouvel atelier, tu reprends là encore où tu t'étais arrêté à l'étape précédente. Le héros peut se déplacer et attaquer les monstres sur la carte.

> Des modifications mineures ont été apportées à la classe `Fighter` afin d'en simplifier l'instanciation. `$x` et `$y` ont été ajoutés en paramètres dans le constructeur de la classe, tandis qu'`$image`, `$dexterity` et `$strength` ont été enlevés (car il y a déjà des valeurs par défaut) et les propriétés passées en *protected* afin de pouvoir être redéfinies directement dans Hero. Ainsi dans *index.php*, pour instancier le Hero il n'y a plus qu'à lui préciser `$x `et `$y`, sans utiliser les *setters*.

## ❖ C'est la tuile

La carte justement, elle est un peu triste. Tu te trouves sur la colline de Cérynie, entourée d'herbes, d'arbustes et de cours d'eau. Il va falloir représenter tout cela.

- Commence par créer une nouvelle classe `Tile`. Une tuile (une "case" sur la carte) va avoir des coordonnées `$x` et `$y` ainsi qu'une `$image` pour la représenter (avec les *getters* et *setters* correspondants.)

Tu remarqueras que la classe `Fighter` possède également ces mêmes méthodes. C'est logique puisque les tuiles ou les combattants doivent pouvoir être affichés et positionnés sur une carte (arène). Notre arène manipule donc des objets cartographiables et qui **doivent** impérativement l'être. Pous s'en assurer, tu vas créer une interface "**Mappable**" contenant ces 6 méthodes, et faire en sorte que `Tile` et `Fighter` l'implémentent (attention pour `getImage()` n'oublie pas de concaténer le chemin complet vers l'image).

Rappel : une interface ne contient **que** des signatures de méthodes (pense au typages), tu ne dois donc mettre ni propriétés, ni "corps" pour les méthodes.
{: .alert-warning }

- Dans `Tile` ajoute un constructeur avec deux paramètres, permettant de spécifier les coordonnées de la tuile en *x* et *y* au moment de l'instanciation.  
Mets également une chaîne vide par défaut pour la propriété `$image`.

Sur ta carte, tu vas avoir plusieurs type de tuiles représentant les différent éléments du paysage (herbe, eau...), chacune ayant ses propres spécifités (traversable ou non...). Tu ne seras jamais amené à instancier directement une classe `Tile`, mais toujours quelques chose de plus précis. Tu l'auras compris, nous avons affaire ici à une classe abstaite ! Modifie `Tile` en conséquence.

Tu vas créer autant de nouvelles classes que de types de tuile. 
- Pour commencer, crée une classe `Grass` et une `Water` toutes deux héritant de `Tile`. Pour le moment, spécifie uniquement une valeur par défaut à la propriété `$image` (à passer en *protected* par la même occasion), respectivement *grass.png* et *water.jpg*. Ces classes sont un peu vides et peu différentes, mais elles vont se spécialiser par la suite 😉.

Il faut maintenant passer les tuiles à la carte. 
- Dans la classe `Arena`, ajoute une propriété `$tiles` de type *array*. 
- Ajoute également un troisième paramètre `$tiles` optionnel, dans le constructeur (et ses *getters/setters*).

Reset la partie, tu devrais voir apparaître de l'herbe et de l'eau, certaines cases restant vides.

## 💦 Méfie toi de l'eau qui dort

Si Héraclès peut se déplacer sans problème sur l'herbe (ou sur un sol sans tuile définie), il ne peut pas se déplacer sur l'eau. Or, pour le moment il n'y a aucune contrainte et notre héros peut se déplacer où bon lui semble.

- Pour régler cela, commence par ajouter une nouvelle propriété `$crossable` (booléen *true* par défaut) à `Tile`. 

Pour un booléen, il est courant de nommer le *getter* `isCrossable()` plutôt que `getCrossable()`
{: .alert-info }

- Dans la classe `Water`, passe `$crossable` à *false*;
- Maintenant, tu vas modifier légèrement le comportement de la méthode `move()` d'`Arena` pour qu'Héraclès ne puisse pas se déplacer sur une tuile qui ne l'autoriserait pas.
1. Commence par créer une méthode privée `getTile($x, $y)` permettant de récupérer une tuile en fonction de ses coordonnées (ou null si la case est vide).
2. Dans `move()`, récupère la tuile correspondant à la destination potentielle du héros (grâce à la méthode précédemment créée).
3. Vérifie que la tuile est traversable grâce notamment à la nouvelle propriété `$crossable`.
4. Si non lance une exception, si oui, le déplacement continue.

Teste sur la carte que le héros peut bien traverser l'herbe mais pas l'eau.

## 🏞️ Remplissage de la carte

Ajoute une nouvelle classe `Bush`, fille de `Tile`, non traversable et affichant *bush.png*.

Réinitialise le jeu, tu vois que la carte commence à bien se remplir et qu'il est très facile d'ajouter de nouveaux types de tuiles. Pour l'instant nous nous contenterons de ces trois tuiles mais tu seras amené à en créer de nouvelles par la suite !

## 🦌 Biche, ô ma biche

La biche est innaccessible cachée derrière tous ces arbustes. Héraclès reste à l'affut, attendant qu'elle sorte de là. Pour cela, il faudrait déjà que l'animal puisse bouger !

La biche va être un monstre avec un comportement un peu particulier, notamment car elle va pouvoir bouger, ce qui n'est pas le cas des autres monstres. 

### SOLID

Les deux sections suivantes vont te permettre d'expérimenter et de comprendre deux principes contenus dans l'acronyme S**O**L**I**D, à savoir :
- le **O**, avec principe ouvert/fermé (**O**pen/Close principle) ;
- le **I**, avec le principe de séparation des interfaces (**I**nterface segregation).

>[Voici une ressource](https://guillaume-richard.fr/principe-solid-simplifies-avec-des-exemples-en-php/) qui te donnera quelques exemples complémentaires en PHP sur l'ensemble des principes SOLID.


#### Open/Close principle  

Tu pourrais implémenter un comportement spécifique aux biches directement dans `Monster`. Cependant, notre héros à encore pas mal de bêtes à combattre et donc autant de spécificités, ce qui rendrait au final la classe `Monster` très chargée et certainement pleine de conditions spécifiques à tel ou tel sous-type de monstres.

Une autre solution est de ne pas toucher directement à `Monster`, et de créer une classe `Hind` (pour Biche), fille de `Monster`, qui contiendra le code spécifique aux biches. À chaque nouveau type de monstre ayant une spécificité propre, tu ajoutes une nouvelle classe. Et il reste toujours possible d'instancier un monstre "de base" si tu le souhaites.  

C'est cela, le "**O**pen/Close Principle" de S**O**LID. Les classes, une fois définies, devraient rester "fermées" à toute modification (qui ne concernerait pas tous les objets de cette classe). S'il t'arrive de rencontrer un tel cas, la classe doit cependant être ouverte à une extension, tu dois pouvoir créer une classe fille facilement.

- Dans ta nouvelle classe `Hind`, ajoute une propriété `$image` avec pour valeur *hind.svg*. 
- Dans le fichier *index.php*, modifie l'instanciation d'un `Monster` par un `Hind` et enlève le `setImage()` de la ligne suivante qui n'est plus nécessaire.

#### Interface segregation
Réflechissons un instant à l'interface `Mappable`. Celle-ci contient les *getters* ET les *setters* pour les coordonnées. Or, pour afficher un élément sur une carte, seuls les *getters* sont réellement utiles. Par contre, les *setters* seront nécessaires pour modifier ces coordonnées. C'est ce qui arrive si l'élément peut bouger.  
Tâchons de respecter le **I** de SOL**I**D (Interface segregation) qui a pour but d'éviter à des classes de devoir implémenter des méthodes qui lui seraient inutiles, en séparant ces méthodes dans différentes interfaces plus petites.  

C'est exactement le cas ici avec `Mappable`, car les `Tile` n'ont pas besoin de **setX()** et **setY()** mais sont obligés de l'implémenter car `Mappable` les contient.

- Crée une nouvelle interface `Movable` et, tu vas y transférer uniquement `setX()` et `setY()`. Attention cependant, un élément `Movable` est forcément `Mappable` ! Donc `Movable` doit également étendre `Mappable` (le concept d'héritage existe aussi pour les interfaces). 
- Fais en sorte que `Hero` et `Hind` l'implémentent. Cela signifie que seuls Héraclès et la biche peuvent bouger, pas les autres monstres.
- À chaque fois que le héros a fini de bouger, les monstres ayant la capacité de bouger vont le faire, en utilisant eux même la méthode `move()` qui contient déjà toute la logique de déplacement (directions, contrôle des cases...). Dans les paramètres de `move()` type donc plutôt avec `Movable` qu'avec `Fighter` (et renomme `$fighter` en `$movable`), afin de s'assurer justement que le mouvement puisse s'effectuer.

### Gameplay
Le *gameplay* va être le suivant : le héros bouge puis, si son mouvement s'est bien effectué, tous les monstres qui en ont la capacité bougent à leur tour. Héros et monstres vont utiliser la même méthode `move()` pour gérer leur mouvement, afin de bénéficier des vérifications propres à tous les `Movable`.  

Pour gérer cela, crée une methode `arenaMove(string $destination)`, celle-ci sera prise en compte automatiquement dans *index.php* si elle existe.
- Dans la méthode `arenaMove()`, déplace le héros grâce à `move()` (pour rappel dans Arena, tu le récupères via `$this->getHero()`).
- Puis toujours dans `arenaMove()` boucle sur les monstres implémentant `Movable` 

HINT : pour un objet donné, tu peux utiliser `instanceof` aussi bien sur son nom de classe, une classe parente qu'il étend ou une interface qu'il implémente.
{: .alert-info }

- Pour chacun de ces monstres déplaçables, choisis une direction aléatoirement 

HINT : appuie toi sur la fonction PHP `array_rand()` et sur la constante `DIRECTIONS`
{: .alert-info }

- Puis, bouge le monstre dans cette direction en réutilisant la méthode `move()`.

- Tu peux reset la partie, tu devrais voir la biche bouger après chaque mouvement d'Héraclès.

Si tu souhaites tester, tu peux également ajouter une seconde Hind et un Monster, les biches devraient bouger et pas le monstre car il n'implémente pas `Movable` !  
Reviens ensuite à une seule biche sur la carte.


## 🙈 Cache cache 🙉 
Dernier point, notre biche bouge mais est bloquée derrière les buissons. Modifions cela. 

Si Héraclès ne sait comment traverser les épineux buissons de la forêt, cela n'est pas un problème pour la biche. Nous allons donc faire en sorte que la tuile Bush soit non traversable, **sauf** pour une biche ! 

- Pour cela, modifie la méthode `isCrossable()` de `Tile`, pour qu'elle prenne en paramètre une instance de `Movable`. 
- Dans `Arena`, lorsque tu utilises `isCrossable()`, passe en paramètre ce qui est en train d'essayer de bouger sur cette tuile.
- Dans `Bush` maintenant, redéfinis `isCrossable()` afin que la fonction renvoie *true* si et seulement si `$movable` est une instance de `Hind`. Dans les autres cas, le *getter* renvoie la valeur de la propriété `$crossable` (qui doit être *false*). En d'autres termes, la tuile n'est pas traversable sauf si c'est une biche qui essaie ! 

Effectue plusieurs mouvements, tu vas voir que la biche devrait finir par traverser les buissons, c'est le moment tant attendu, attrape la !

> Nous ne chercherons pas à implémenter le fait qu'Héraclès capture effectivement la Biche car cela demanderait pas mal d'efforts supplémentaires, mais comme toujours, si tu souhaites essayer, n'hésite pas !

Et bravo pour ta ténacité 👏, tu es arrivé au bout de cet atelier 🙌 !
{: .text-center }
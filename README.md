# TD Egg Warehouse

L'objectif de ce TD est de valider l'utilisation des notions de Technologies Logicielles vu en cours (Python, Git, Test et Docker) et de démontrer votre capacité à organiser votre travail. Concernant Git, les critères d'évaluation sont : la création d'issues et de branches, le recours à des Pull Requests, pas de commit réalisés directement dans la branche principale, réalisation de commits atomiques.

Le thème de ce TD est l'entrepôt d'oeufs. Vous devrez créer:

        - un script python `seeder.py` permettant d'insérer des oeufs dans une base de donnée;
        - un script python `egg_warehouse.py` permettant de récupérer des informations sur les oeufs disponibles et à réaliser des traitements dessus.

Le premier traitement que l'on souhaite réaliser consiste à itérer sur une base de données et tester si les immatriculations associées aux oeufs sont valides.

# Consignes

Créez un dépôt *privé* nommé TD_EGG_WAREHOUSE sur Github.

Ajoutez moi à ce dépôt

Publiez **régulièrement** votre code et ses modifications dans le dépôt

# Exercice 1 - Créez un code Python analysant les numéros d'immatriculation

Vous allez débuter le projet en créant :

- une classe `Warehouse` qui  :

        - encapsule un nombre d'emplacement d'oeufs et une liste d'oeufs;
        - dispose de méthodes permettant d'accéder à ces informations et d'ajouter ou de retirer une instance `Egg` à la boîte;
        - possède une méthode publique `is_valid` qui vérifie que le nombre d'oeufs composant l'entrepot est inférieur au nombre d'emplacements et déclenche l'analyse des immatriculations des oeufs et retourne `True` si les immatriculations sont valides et que le nombre d'oeufs est inférieur ou égal au nombre d'emplacements, `False` sinon.

- une classe `Egg` qui :

        - encapsule un nom d'élevage d'origine, une couleur et un numéro d'immatriculation
        - possède une méthode publique `is_valid` qui retourne `True` si l'immatriculation est valide, `False` sinon.

Une immatriculation est valide si elle est composée de 12 caractères :
    
    - deux chiffres qui représentent un nombre divisible par 5 (poids de l'oeuf arrondi);
    - un tiret
    - deux lettres qui représentent les initiales de la France ou d'un des pays voisins à la Franche (Belgique, Allemagne, Luxembourg, Suisse, Italie ou Espagne)
    - trois chiffres qui ne doivent pas représenter un palindrome (représentent le code de l'élevage)
    - deux chiffres allant de O1 à 31 (jour de ponte)
    - deux lettres indiquant le mois de ponte ("JA", "FE", "MA", "AV", "MI", "JU", "JL", "AO", "SE", "OC", "NO", "DE")

Attention, les deux dernier chiffres et deux dernières lettres doivent identifier des mois différents. Par exemple "01" et "FE" est valide mais pas "02" et "FE".

Dans cet exercice, il est attendu que vous réalisiez des **tests unitaire**.

# Exercice 2 - Utilisons Docker pour peupler une base de donnée MongoDB avec un code Python (procédure interactive)

Ici, nous allons jouer avec deux images Docker qui devrons pouvoir communiquer entre elles.
Pour cela, nous allons utiliser Docker pour créer un réseau que nous nommerons `mynet`

```
docker network create mynet
```
Pour les utilisateurs de Windows, il est possible que cette appel ne fonctionne pas. Dans ce cas, passez par la commande
```
docker network create --driver nat mynet
```

Commencez par lancer un premier conteneur basé sur l'image `mongo` que vous nommerez `mongodb` et que vous connecterez au réseau `mynet` (utilisez l'option `--network mynet`).
Ensuite, lancez un second conteneur basé sur l'image `python:alpine` que vous nommerez `python-mongo` et que vous connecterez au réseau `mynet`.
Afin d'éviter que le conteneur de l'image python ne se ferme automatiquement, il faudra utiliser le mode "foreground" (-t, -i ou -it).

Dans le conteneur `python-mongo` installez la paquet `pymongo` que nous allons utiliser pour remplir la base de donnée.
Utilisez l'interpréteur python du conteneur, pour peupler la base de données avec des immatriculations d'oeuf en vous inspirant du code suivant :

```python
import pymongo
# Import de la classe MongoClient qui nous permettra de nous connecter a la base de donnees MongoDB
from pymongo import MongoClient

client = MongoClient(host="A_REMPLIR")

# Acces a la base de donnees "NOM_BASE_DE_DONNEES"
db = client["NOM_BASE_DE_DONNEES"]

# Acces a la collection "NOM_COLLECTION"
col = db["NOM_COLLECTION"]

# Ajout d'un 'fruit' dans la collection
fruit = {
        "nom": "banane",
        "couleur": "jaune"
}
res = col.insert_one(fruit)

# Verification de l'ajout
print(f'Le fruit {res.inserted_id} a bien ete cree')

# Localise et affiche le fruit
col.find_one()
```

En parallèle, vous pouvez vous assurer que la base de donnée à bien été remplie depuis le conteneur `mongo`.
Pour cela, interagissez avec celui-ci en exécutant la commande `mongosh`.
Une fois réalisé, utilisez la base de donnée que vous défini depuis le conteneur python.
`use NOM_BASE_DE_DONNEES`
Ensuite, interrogez la base pour récupérer un élément qui compose la collection que vous avec définit.
`db.NOM_COLLECTION.findOne()`

# Exercice 3 - Créons un script Python peuplant la base de donnée et faisons en une image Docker

Ecrivez un script Python `seeder.py` permettant de peupler la base de donnée.
Pour cela, vous allez utiliser les tests réalisés à l'Exerice 1 et ajouter des entrées valides et non valides.

Une fois le scrip écrit, il ne vous reste plus qu'à définir le Dockerfile associé où l'on doit retrouver:

    - l'installation des dépendances nécessaires
    - la copie des données nécessaires
    - l'exécution du script `seeder.py`

Maintenant, vous allez construire votre image en la taggant par `$DOCKERID/seed_mongo` (où DOCKERID est votre identifiant docker hub).
Puis, après vous être connecté à Docker Hub, vous allez publier votre image.

# Exercice 4 - Créons un script Python traitant la base de donnée

Ecrivez un script Python `egg_warehouse.py` basé sur vos développement à l'Exercice 1.
Lorsque celui-ci est appelé, il doit se connecter à la base de données et regarder que les immatriculations sont valides.
Dans le cas où une immatriculation n'est pas valide, il doit l'afficher à l'écran.

Note : pour itérer sur une collection obtenus via `pymongo`, vous pouvez utiliser le snippet suivant:

```python
fruits = col.find()
for fruit in fruits:
        nom = fruit["nom"]
        couleur = fruit["couleur"]
```

# Exercice 5 - Assemblons les différents conteneurs avec Docker Compose !

Votre fichier `docker-compose.yml` va se composer de trois services :

    - `mongodb` : que vous allez lancer à partir de l'image `mongo:latest`
    - `seed_mongo` :  que vous allez lancer à partir de votre image `$DOCKERID/seed_mongo` (ou de celle d'un de vos camarades) et qui dépendra de `mongodb`
    - `egg_warehouse` : que vous allez construire à partir des sources de l'Exercice 4 (il vous faudra faire un nouveau Dockerfile dédié à ce conteneur) et qui dépendra de seed_mongo

Note : dans le cas du conteneur `mongodb`, n'oubliez pas que le service `mongod` utilise le port 27017.

Vérifiez ensuite que tout cela fonctionne :
`docker-compose up --build`
et, depuis un autre terminal
`docker exec -it NOM_DU_CONTENEUR bash`
et enfin l'exécution de votre code python réalisant le traitement sur la base de donnée.

# Exercice 6 - Bonus 

Modifiez `egg_warehouse.py` pour en faire une interface en ligne de commande (CLI).
Pour commencer vous pouvez recouvrir vos développements existant, i.e. ajouter une option "--check" permettant de regarder si les immatriculations sont valides.
Ensuite vous pouvez proposer d'autres options :
    - afficher les informations relatives à un oeuf à partir de son immatriculation (poule pondeuse, taille, ...)
    - rechercher les oeufs provenant d'une certaine poule pondeuse, mois de ponte, ...
    - supprimer les immatriculations invalides dans la base de données
    - ...

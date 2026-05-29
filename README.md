<p align="left">

<img src="images/Institut-Galilee.png" alt="Logo Université" height="65" align="left">
  <img src="images/Sup-Galilee.png" alt="Logo Université" height="50" align="right">
</p>

<br> <br>
<h2 align="center">
 Accès aux données en PHP via PDO  (PHP Data Objects) 
</h2>
<br> 
Ce TP a pour objectif de vous guider pas à pas dans l’utilisation de PDO (_PHP Data Objects_) pour accéder à une base de données en PHP. Il se compose de deux parties : une première partie dédiée à la mise en place de l’environnement et à l’importation des données, et une seconde partie consacrée aux requêtes et à l’exploitation des données. Si vous avez déjà une base de données prête à l’emploi, vous pouvez commencer directement par la deuxième partie.  Quel que soit le SGBD relationnel que vous utilisez, la manière d’interroger les données reste la même et ne change pas dans les grandes lignes (c’est d’ailleurs l’un des grands atouts des SGBD : **la séparation entre la couche physique et la couche logique**, qui permet de manipuler les données sans se soucier de leur stockage réel). Dans la suite de ce TP, j’utiliserai **MAMP** pour les exemples, mais les principes présentés s’appliquent de la même façon avec d’autres environnements.   La figure suivante illustre le cycle de vie complet d’une requête HTTP dans une application web dynamique. Le client (navigateur) envoie une requête HTTP vers le serveur web, qui la transmet au moteur PHP pour exécution. Le script PHP s’appuie sur **PDO** comme couche d’abstraction d’accès aux données afin d’établir une connexion avec le **SGBD relationnel**, d’exécuter des requêtes SQL et de récupérer les ensembles de résultats. Ces données sont ensuite traitées côté serveur avant d’être encapsulées dans une réponse HTTP renvoyée au client. 



> **PDO** est une interface standard de PHP qui permet d’accéder à des bases de données de manière **sécurisée, cohérente et portable**.  
> Introduit à partir de **PHP 5.1 (2005)**, PDO fournit une couche d’abstraction pour interagir avec différents systèmes de gestion de bases de données (MySQL, PostgreSQL, SQLite, etc.).  
>  
> Son objectif principal est de **séparer le code PHP des requêtes SQL**, de **sécuriser les échanges avec la base de données** (notamment contre les injections SQL) et de proposer une **API unifiée**, indépendante du moteur de base de données.  
>  
> **PDO n’est pas un framework** : il s’agit d’une **extension native de PHP**, intégrée au langage, destinée exclusivement à la gestion des accès aux bases de données.


<p align="center">
  <img src="images/Pasted image 20251220094451.png" alt="description" >
</p>


<p align="center">
  <img src="images/Pasted image 20251219190639.png"
       alt="description"
       width="800"
       height="100">
</p>

---

##### Partie I 

Les scripts de création du schéma _livres_(**id**, titre, auteur, annee) et d’initialisation des données sont fournis ci-après et sont également accessibles dans le sous-dossier **scriptsSql**.

```sql
CREATE DATABASE bibliotheque CHARACTER SET utf8mb4;
USE bibliotheque;

CREATE TABLE livres (
  id INT AUTO_INCREMENT PRIMARY KEY,
  titre VARCHAR(255) NOT NULL,
  auteur VARCHAR(255) NOT NULL,
  annee INT NOT NULL
);
```

```sql
INSERT INTO livres (titre, auteur, annee) VALUES
('L’Étranger', 'Albert Camus', 1942),
('1984', 'George Orwell', 1949),
('Le Petit Prince', 'Antoine de Saint-Exupéry', 1943),
('La Peste', 'Albert Camus', 1947);
```

---

##### Partie II

**Connexion à la base** : le script suivant permet d'établir une connexion sécurisée entre un programme PHP et la base de données _bibliotheque_ créée précédemment.  Une bonne pratique consiste à sauvegarder ce script dans un fichier séparé pour son utilisation éventuelle par d'autres scripts. Appelons ce fichier _db.php_ 

```sql
<?php
$host = "localhost";
$dbname = "bibliotheque";
$user = "root";
$pass = "root";

try {
    $pdo = new PDO(
        "mysql:host=$host;dbname=$dbname;charset=utf8mb4",
        $user,
        $pass
    );
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Erreur de connexion à la base : " . $e->getMessage());
}
```

Les **informations nécessaires pour se connecter à la base de données** sont définies par les variables suivantes :

- `$host` : adresse du serveur de base de données.  `localhost` signifie que la base de données est sur la **même machine** que le serveur PHP.
- `$dbname` : nom de la base de données à utiliser (`bibliotheque`).
- `$user` : nom d’utilisateur MySQL.
- `$pass` : mot de passe associé à cet utilisateur.
En pratique, ces valeurs peuvent changer selon l’environnement (MAMP, serveur distant, etc.).

>  **Bonnes pratiques**
>- Ne jamais afficher les erreurs détaillées en production
>- Utiliser toujours `utf8mb4` 
>Ce n'est toujours pas clair ? Revoir le cours 😊

Une fois les paramètres sont définis, la prochaine étape est d'établir la connexion proprement dite, en créant un objet de la classe _PDO_. Le constructeur prend trois paramètres en entrée : 

- `"mysql:..."` est le **DSN (Data Source Name)**  
    Il indique :
    - le type de base de données (`mysql`)
    - le serveur (`host`)
    - la base utilisée (`dbname`)
    - le jeu de caractères (`utf8mb4`)
- `$user` et `$pass` sont utilisés pour **s’authentifier**.

Si la connexion est réussie, l’objet `$pdo` permet ensuite d’**exécuter des requêtes SQL**.

Quant la gestion des erreurs, cette instruction  `$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);` indique à _PDO_ de  lever des exceptions en cas d'erreur.  Elles sont capturées dans un bloc `try/catch`. Ce dernier contient les instructions susceptibles de générer des erreurs.  `die()` stope immédiatement l'exécution du script en cas d'erreur.  

Illustrons la manière d’envoyer une requête simple, par exemple : `SELECT * FROM livres`, et afficher le résultat dans une page HTML.  Le résultat que l'on souhaite affiché est donné par la figure suivante. 

![[Pasted image 20251221101748.png]]

Le programme complet est donné ci-après.  Appelons-le _index.php_
Le script peut être décomposé en deux parties principales. La première consiste à inclure le script de connexion afin d’établir l’accès à la base de données _bibliotheque_. La seconde partie exécute une requête SQL permettant de récupérer l’ensemble des enregistrements de la table _livres_. Les résultats de cette requête sont ensuite stockés dans une structure de données PHP afin de pouvoir être exploités et affichés par l’application. La deuxième partie  du script est dédiée à l’affichage des données côté client. Elle génère une page HTML dans laquelle la liste des livres récupérés depuis la base de données est affichée dynamiquement. Le code PHP est intégré directement dans la structure HTML afin de parcourir les résultats et de présenter chaque livre de manière lisible et sécurisée.

 **Bonnes pratiques**
>-  Notez que `<?= htmlspecialchars($livre['titre']) ?>` est équivalent à `<?php echo htmlspecialchars($livre['titre']); ?>`. Cette dernière est à privilégier car elle est plus lisible dans un document `HTML` et très utilisée dans les templates (elle fonctionne à partir de `PHP 5.4` et elle est activée par défaut).    
>- `htmlspecialchars()` permet d’afficher des données de manière sécurisée en empêchant l’exécution de code HTML ou JavaScript injecté.  Il ne faut surtout pas écrire `<?= $livre['titre'] ?>` si les données ne sont pas sûres (Cf. cours). 

```php
<?php
// Inclusion du fichier de connexion à la base de données.
// Ce fichier instancie l'objet $pdo permettant de communiquer avec le SGBD.
require "db.php";
// Définition de la requête SQL.
// Ici, on récupère l'ensemble des livres présents dans la table 'livres'.
$sql = "SELECT * FROM livres";
// Exécution directe de la requête SQL via l'objet PDO.
// La méthode query() est utilisée pour les requêtes simples sans paramètres.
$stmt = $pdo->query($sql);
// Récupération de tous les résultats sous forme de tableau associatif.
// Chaque élément du tableau correspond à un livre.
$livres = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>
<!DOCTYPE html>
<html lang="fr">
<head>
    <!-- Déclaration de l'encodage de la page -->
    <meta charset="UTF-8">

    <!-- Titre affiché dans l'onglet du navigateur -->
    <title>Liste des livres</title>
</head>
<body>

<!-- Titre principal de la page -->
<h2>Liste des livres</h2>

<!-- Début de la liste non ordonnée -->
<ul>
<?php
// Parcours du tableau $livres contenant les résultats de la requête SQL.
// Chaque itération correspond à un livre.
foreach ($livres as $livre):
?>
    <li>
        <!-- Affichage sécurisé du titre du livre -->
        <?= htmlspecialchars($livre['titre']) ?>

        <!-- Affichage de l année de publication -->
        (<?= $livre['annee'] ?>) –

        <!-- Affichage sécurisé du nom de l auteur -->
        <?= htmlspecialchars($livre['auteur']) ?>
    </li>
<?php endforeach; ?>
</ul>

</body>
</html>
```

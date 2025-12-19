<p align="left">

<img src="images/Institut-Galilee.png" alt="Logo Université" height="65" align="left">
  <img src="images/Sup-Galilee.png" alt="Logo Université" height="50" align="right">
</p>

<br> <br><br> 

<h2 align="center">
 Accès aux données en PHP via PDO  (PHP Data Objects) 
</h2>
<br> 

Ce TP a pour objectif de vous guider pas à pas dans l’utilisation de PDO pour accéder à une base de données en PHP. Il se compose de deux parties : une première partie dédiée à la mise en place de l’environnement et à l’importation des données, et une seconde partie consacrée aux requêtes et à l’exploitation des données. Si vous avez déjà une base de données prête à l’emploi, vous pouvez commencer directement par la deuxième partie.  Quel que soit le SGBD relationnel que vous utilisez, la manière d’interroger les données reste la même et ne change pas dans les grandes lignes (C’est d’ailleurs l’un des grands atouts des SGBD : **la séparation entre la couche physique et la couche logique**, qui permet de manipuler les données sans se soucier de leur stockage réel). Dans la suite de ce TP, j’utiliserai **MAMP** pour les exemples, mais les principes présentés s’appliquent de la même façon avec d’autres environnements.  


> **PDO** est une interface standard de PHP qui permet d’accéder à des bases de données de manière **sécurisée, cohérente et portable**.  
> Introduit à partir de **PHP 5.1 (2005)**, PDO fournit une couche d’abstraction pour interagir avec différents systèmes de gestion de bases de données (MySQL, PostgreSQL, SQLite, etc.).  
>  
> Son objectif principal est de **séparer le code PHP des requêtes SQL**, de **sécuriser les échanges avec la base de données** (notamment contre les injections SQL) et de proposer une **API unifiée**, indépendante du moteur de base de données.  
>  
> **PDO n’est pas un framework** : il s’agit d’une **extension native de PHP**, intégrée au langage, destinée exclusivement à la gestion des accès aux bases de données.

<p align="center">
  <img src="images/Pasted image 20251219190639.png" alt="description" >
</p>

```php
<?php
$mysqlClient = new PDO(
    'mysql:host=localhost;dbname=partage_de_recettes;charset=utf8',
    'root',
    ''
);
?>
```

```php
<?php
try {
    // On se connecte à MySQL
    $mysqlClient = new PDO('mysql:host=localhost;dbname=partage_de_recettes;charset=utf8', 'root', 'root');
} catch (Exception $e) {
    // En cas d'erreur, on affiche un message et on arrête tout
    die('Erreur : ' . $e->getMessage());
}
// Si tout va bien, on peut continuer

// On récupère tout le contenu de la table recipes
$sqlQuery = 'SELECT * FROM recipes';
$recipesStatement = $mysqlClient->prepare($sqlQuery);
$recipesStatement->execute();
$recipes = $recipesStatement->fetchAll();

// On affiche chaque recette une à une
foreach ($recipes as $recipe) {
?>
    <p><?php echo $recipe['author']; ?></p>
<?php
}
?>
```


##### Partie I 

Les scripts de création du schéma _livres_ et d’initialisation des données sont fournis ci-après et sont également accessibles dans le sous-dossier **scriptsSql**.

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


##### Partie II


Connexion à la base. 

```sql
<?php
$host = "localhost";
$dbname = "bibliotheque";
$user = "root";
$pass = "";

try {
    $pdo = new PDO(
        "mysql:host=$host;dbname=$dbname;charset=utf8mb4",
        $user,
        $pass
    );
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Erreur de connexion : " . $e->getMessage());
}
```

```php
<?php
require "db.php";

$sql = "SELECT * FROM livres";
$stmt = $pdo->query($sql);
$livres = $stmt->fetchAll(PDO::FETCH_ASSOC);
?>

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Liste des livres</title>
</head>
<body>

<h2>Liste des livres</h2>

<ul>
<?php foreach ($livres as $livre): ?>
    <li>
        <?= htmlspecialchars($livre['titre']) ?>
        (<?= $livre['annee'] ?>) –
        <?= htmlspecialchars($livre['auteur']) ?>
    </li>
<?php endforeach; ?>
</ul>

</body>
</html>
```





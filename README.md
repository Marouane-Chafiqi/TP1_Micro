TP1 : chargement dynamique de classes

Un petit projet pour comprendre comment Java peut créer des objets **sans écrire `new`** dans le code. Les classes à utiliser sont lues dans un fichier texte, puis instanciées à l'exécution grâce à la réflexion.

C'est exactement ce que font Spring ou CDI en coulisses, mais ici en version minimaliste, avec uniquement le JDK.

## L'idée en deux mots

Le code métier ne connaît que des **interfaces** (`IDao`, `IMetier`). Quelles classes concrètes utiliser ? C'est écrit dans `config.txt`. Si demain on veut changer d'implémentation, on modifie une ligne du fichier, sans toucher au code.

## Les fichiers

```
reflexion-lab/
├── pom.xml
├── config.txt                      <- les noms des classes à charger
└── src/main/java/
    ├── dao/
    │   ├── IDao.java               <- contrat : getValue()
    │   └── DaoImpl.java            <- renvoie 100.0
    ├── metier/
    │   ├── IMetier.java            <- contrat : calcul()
    │   └── MetierImpl.java         <- prend la valeur du DAO et la multiplie par 2
    └── presentation/
        └── Presentation2.java      <- le "conteneur" qui fait tout le travail
```

## Ce que fait `Presentation2`

1. Lit `config.txt` : une classe par ligne (`dao.DaoImpl`, puis `metier.MetierImpl`).
2. Charge chaque classe avec `Class.forName(...)` et crée l'objet avec son constructeur par défaut.
3. Appelle `setDao(...)` par réflexion pour brancher le DAO dans le métier (c'est l'injection de dépendances, faite à la main).
4. Lance `calcul()` et affiche le résultat.

## Comment l'exécuter

### Avec IntelliJ (le plus simple)

1. `File > Open` et choisir le dossier `reflexion-lab` (celui qui contient `pom.xml`).
2. Ouvrir `Presentation2.java`, cliquer sur le triangle vert, puis **Run**.

Si IntelliJ ne trouve pas `config.txt` : `Run > Edit Configurations`, puis mettre *Working directory* à `$MODULE_WORKING_DIR$`.

### Avec la ligne de commande

Depuis le dossier qui contient `pom.xml` (Maven doit être installé) :

```bash
mvn compile
java -cp target/classes presentation.Presentation2
```

Il faut lancer la commande depuis la racine du projet, sinon `config.txt` ne sera pas trouvé.

## Résultat attendu

```
[CONTAINER] DAO instancié : dao.DaoImpl
[CONTAINER] Métier instancié : metier.MetierImpl
[DAO] Récupération de la valeur depuis la source de données...
[METIER] Calcul effectué. Résultat : 200.0
Résultats = 200.0
```

Si les accents s'affichent mal sous Windows, ajouter `-Dfile.encoding=UTF-8` aux options de la JVM.

## Si ça plante

| Erreur | Cause probable |
|---|---|
| `FileNotFoundException: config.txt` | On n'est pas dans le bon dossier (voir ci-dessus) |
| `ClassNotFoundException` | Faute de frappe dans `config.txt` (le nom doit inclure le package) |
| `NullPointerException` à la lecture | `config.txt` a une ligne vide ou moins de 2 lignes |
| `NoSuchMethodException` | Le setter `setDao` est absent ou n'a pas le bon type |

## À retenir

<img width="1128" height="265" alt="Capture d&#39;écran 2026-10-12 101024" src="https://github.com/user-attachments/assets/9606a678-c215-4ee4-851d-3bba878e6405" />




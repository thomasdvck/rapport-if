# Ruby on Rails

**Séance du 16 novembre 2026** présentée par Maria Ifourah, Rayane Alli et Ilyas Ait Ali

## Le langage
Ce que j'ai retenu de la présentation sur Ruby (créé en 1995 par Yukihiro Matsumoto), c'est qu'il s'agit d'un langage interprété, dynamique et 100 % orienté objet. Absolument tout est un objet, même les types basiques comme les entiers, les booléens ou le `nil`.
Sa philosophie tourne autour du **"Developer Happiness"** et du principe de moindre surprise : le but est d'écrire un code lisible, intuitif et très proche de l'anglais (avec des méthodes explicites comme `empty?` ou `reverse`). J'ai aussi remarqué l'utilisation massive du module `Enumerable` et des blocs (`.map`, `.select`, `.reduce`), qui permettent de manipuler des collections très proprement en évitant les boucles traditionnelles fastidieuses que l'on écrit souvent en Java.

## Ce que le framework impose, ce qu'il laisse libre
* **Ce qu'il impose :** L'architecture **MVC** (Modèle-Vue-Contrôleur) classique, mais surtout le paradigme **"Convention over Configuration"**. C'est assez bluffant à découvrir : au lieu de faire des fichiers de configuration complexes, il suffit de bien nommer ses classes. Par exemple, si je crée un modèle `User`, Rails comprend tout seul qu'il doit taper dans la table `users` en base de données. Pareil pour le routeur : un simple `resources :wallets` génère les 7 routes RESTful d'un coup.
* **Ce qu'il laisse libre :** L'organisation de la logique interne via des modules et des *mixins* (via le mot-clé `include`). Cela permet de partager des comportements entre nos classes pour compenser l'interdiction de l'héritage multiple en Ruby.

## Persistance et accès aux données
Tout passe par **Active Record**, l'ORM de Rails. Il masque presque totalement le SQL pour les opérations CRUD habituelles et gère l'évolution de la base de données via des **migrations** historisées.
*Le point de vigilance :* La magie de l'ORM a un coût. Le groupe nous a bien alertés sur le fameux **problème des requêtes N+1**. Si on itère sur des données liées sans utiliser la commande `.includes(:association)` pour les précharger, on fait exploser le nombre de requêtes SQL en arrière-plan, ce qui détruit les performances.

## Sécurité
Rails fait beaucoup de choses par défaut pour nous protéger. Les jetons **CSRF** sont intégrés automatiquement dans les formulaires, et l'ORM empêche les injections SQL grâce aux requêtes paramétrées.
L'élément le plus marquant est le mécanisme des **Strong parameters** : le framework nous oblige à déclarer explicitement dans nos contrôleurs les champs (inputs) qu'un utilisateur a le droit de modifier. Cela évite qu'un utilisateur malveillant ne s'attribue des droits administrateur en falsifiant une requête POST. On a aussi découvert **Brakeman**, un outil natif pour scanner statiquement les vulnérabilités de notre code.

## Production et déploiement
L'écosystème m'a fait penser à Maven ou NPM, mais ici on parle de **Gems** (accessibles sur RubyGems.org). Rails lui-même n'est finalement qu'un gros assemblage de gems. L'outil **Bundler** lit notre fichier `Gemfile` et génère un `Gemfile.lock` pour verrouiller les versions, garantissant que toute l'équipe de dev (et le serveur de prod) utilise exactement le même environnement.

## Le TP : la même application, autrement
Pour ce premier atelier de 2 heures, on a travaillé sur une feature de notre fil rouge. On est partis d'une branche starter avec l'authentification et les portefeuilles déjà configurés.
* **Création du modèle :** J'ai généré le modèle `Expense` et sa migration, et j'ai pu voir comment Active Record gère les validations métier (comme vérifier qu'un montant est strictement positif) et les clés étrangères vers d'autres tables (créateur, payeur, portefeuille).
* **Le cycle complet :** J'ai imbriqué les routes des dépenses dans les portefeuilles, créé le contrôleur (en sécurisant la saisie avec les Strong Parameters) et codé les vues avec le moteur de template ERB et son helper `form_with`.
* **Optimisation :** La dernière mission m'a fait retoucher le calcul du total des dépenses. Au lieu de rapatrier toutes les dépenses en mémoire pour faire une somme en Ruby, on a utilisé une fonction d'agrégation d'Active Record (`sum(:amount)`) pour que le calcul se fasse proprement en SQL. C'était la parfaite illustration des limites de l'ORM abordées pendant la présentation.

## Limites assumées
Malgré son efficacité, j'ai bien compris que Rails n'est pas la solution miracle à tout. Il pousse fortement vers une architecture **monolithique**. C'est idéal pour démarrer vite, mais la transition vers des microservices peut s'avérer très douloureuse plus tard. De plus, côté performances pures, Ruby (langage interprété) ne fait pas le poids face à des langages compilés comme Go ou Java pour des traitements massivement parallèles ou très lourds en calcul CPU.

## Usage et pertinence de l'IA générative
Comme Rails existe depuis très longtemps (2004) et repose sur des conventions universelles strictes, les LLM excellent pour générer tout le code "boilerplate" : pondre des modèles, des migrations ou des contrôleurs basiques se fait en un clin d'œil.
Cependant, c'est à double tranchant. L'IA a souvent tendance à générer du code naïf. Si on ne comprend pas la "magie" du framework, on risque de valider des requêtes N+1 catastrophiques ou d'oublier de bien filtrer nos Strong Parameters. L'IA est un excellent assistant de productivité ici, mais elle exige une relecture humaine extrêmement critique.

## Bilan : Avantages & Inconvénients

* **Les gros avantages :**
    * Une vitesse de développement incroyable (c'est clairement l'outil idéal pour sortir un MVP rapidement ou faire une appli CRUD classique).
    * Un code très expressif, concis et pensé pour le confort du développeur.
    * Un gain de temps massif sur la configuration grâce aux conventions.
    * Une sécurité de base très solide "out of the box".
* **Les inconvénients :**
    * Le typage dynamique décale la détection de certaines erreurs à l'exécution (runtime) plutôt qu'à la compilation, contrairement à Java.
    * Le côté très "magique" et implicite des conventions est déroutant pour le debug si on ne connaît pas les règles cachées du framework.
    * L'orientation monolithe par défaut, moins adaptée aux architectures hautement distribuées.

## Mon analyse
Cette première plongée dans Ruby on Rails m'a offert un énorme contraste avec l'écosystème Java/Spring que l'on manipule habituellement. En Java, on privilégie l'explicite et le typage fort pour se rassurer dès la compilation (approche *Shift-Left*). Avec Rails, on mise tout sur la vitesse, la souplesse et l'implicite.
Je trouve ce framework redoutablement efficace pour délivrer rapidement de la valeur métier sans perdre des heures à configurer des XML ou des annotations. Mais cette magie a un prix : elle exige une grande rigueur, une vraie maîtrise de ce qui se passe sous le capot (notamment côté SQL) et une solide suite de tests pour garder une application maintenable. C'est d'autant plus vrai si on se fait assister par une IA générative, où l'esprit critique du développeur devient le seul véritable garde-fou.
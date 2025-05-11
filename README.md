# Rapport de TP - Gestion des Utilisateurs et Rôles avec Spring Boot et JPA

## 📌 Objectif du TP

L’objectif principal de ce TP est de mettre en œuvre une application Spring Boot permettant la gestion des utilisateurs et des rôles, en utilisant JPA (Java Persistence API) pour la persistance des données. Le système doit permettre de :

- Ajouter des utilisateurs.
- Ajouter des rôles.
- Associer des rôles à des utilisateurs.
- Stocker les données en mémoire via H2 Database.

---

## 🧱 Structure du Projet

Le projet est structuré selon les bonnes pratiques de Spring Boot :

- entities: contient les classes User et Role.  
- repositories: interfaces de persistance avec Spring Data JPA.  
- service: interface et implémentation des services métier.
- web : contient le contrôleur UserController, qui gère les requêtes HTTP liées aux utilisateurs.
- Classe principale : JpaFsApplication.
  
  ![img](new.JPG)
## 📄 Explication détaillée des Classes
###  1. Classe `User`:  

La classe User représente une entité JPA (Java Persistence API) correspondant à la table USERS dans la base de données. Elle utilise les annotations JPA pour la persistance, Lombok pour la génération automatique de code standard, et Jackson pour gérer la sérialisation JSON. Grâce à l’annotation @Entity, cette classe est automatiquement reconnue par JPA comme une entité persistante. L’annotation @Table(name = "USERS") permet de spécifier explicitement le nom de la table à utiliser en base de données.
 - La classe contient trois attributs principaux :  
       - userId est la clé primaire de type String, marquée avec l’annotation @Id, ce qui permet d’identifier de manière unique chaque utilisateur dans la table.  
       - userName correspond au nom de l’utilisateur. Il est annoté avec @Column(name = "USER_NAME", unique = true, length = 20) pour indiquer que cette colonne doit être unique dans la base de données et que sa taille maximale est de 20 caractères.  
       - password est le mot de passe de l’utilisateur. Il est annoté avec @JsonProperty(access = JsonProperty.Access.WRITE_ONLY), ce qui signifie que ce champ ne sera pas inclus dans les réponses JSON (lecture interdite), mais pourra être utilisé lors des envois (écriture autorisée uniquement). Cela permet de renforcer la sécurité en évitant de divulguer le mot de passe dans les échanges de données JSON.

     
La relation entre les utilisateurs et les rôles est modélisée à l’aide de l’annotation @ManyToMany. Elle indique qu’un utilisateur peut avoir plusieurs rôles et qu’un rôle peut être associé à plusieurs utilisateurs.
  - L’attribut mappedBy = "users" signifie que cette relation est gérée du côté de l’entité Role via l’attribut users.
  - fetch = FetchType.EAGER indique que la liste des rôles associés sera automatiquement chargée en même temps que l’utilisateur.
  - Lombok est utilisé avec les annotations @Data, @NoArgsConstructor et @AllArgsConstructor :
       - @Data génère automatiquement les méthodes getters, setters, toString, equals et hashCode.
       - @NoArgsConstructor génère un constructeur sans arguments.
       - @AllArgsConstructor génère un constructeur avec tous les attributs en arguments.



  ![Texte alternatif](classuser.JPG)  
  ###  2. Classe `Role`:
  Cette classe est une entité JPA qui représente la table Role  dans la base de données, utilisée pour gérer les autorisations des utilisateurs (comme "ADMIN", "USER", etc.). Annotée avec @Entity, elle est persistée automatiquement par JPA. Elle utilise Lombok (@Data, @NoArgsConstructor, @AllArgsConstructor) pour générer automatiquement les méthodes standards.   
  La classe contient trois attributs :
   - id, une clé primaire de type Long générée automatiquement.
   - desc, une description facultative du rôle.
   - roleName, le nom du rôle, unique et limité à 20 caractères.
   
La relation entre Role et User est définie par @ManyToMany(fetch = FetchType.EAGER), indiquant qu’un rôle peut être partagé entre plusieurs utilisateurs et inversement.  
L’annotation @JsonProperty(access = JsonProperty.Access.WRITE_ONLY) empêche la liste des utilisateurs d’être exposée dans les réponses JSON, et @ToString.Exclude évite les boucles infinies lors de l’affichage de l’objet.
  ![Texte alternatif](role.JPG) 
## 🗂️ Repositories
### - Interface `RoleRepository` : 
La classe `RoleRepository` est une interface de persistance qui permet de manipuler les données de l'entité Role en interagissant directement avec la base de données. Elle hérite de JpaRepository<Role, Long>, ce qui lui confère automatiquement des méthodes de base pour les opérations CRUD (comme save(), findById(), findAll(), deleteById(), etc.), sans nécessiter d’implémentation manuelle. Le type Role désigne l'entité à gérer, et Long est le type de sa clé primaire (id). Annotée avec @Repository, cette interface est reconnue par Spring comme un composant de persistance injectable. Elle contient également une méthode personnalisée findByRoleName(String roleName), qui permet de rechercher un rôle à partir de son nom.  
Spring Data JPA est capable de générer automatiquement l'implémentation de cette méthode en se basant sur son nom, ce qui évite d’écrire manuellement des requêtes SQL.

  ![Texte alternatif](repositoryrole.JPG) 

### - Interface `UserRepository` :
L’interface UserRepository permet d’accéder aux données de l’entité User en interagissant avec la base de données. Elle étend JpaRepository<User, String>, ce qui lui donne accès à toutes les méthodes CRUD de base sans avoir besoin de les implémenter. Le paramètre User désigne l’entité gérée, et String est le type de sa clé primaire (userId). Bien qu’elle ne soit pas annotée avec @Repository, Spring reconnaît automatiquement cette interface comme un composant de persistance grâce à l’extension de JpaRepository. Elle déclare également une méthode personnalisée findByUserName(String userName), qui permet de retrouver un utilisateur à partir de son nom d’utilisateur. Spring Data JPA génère automatiquement son implémentation en se basant sur le nom de la méthode.
  ![Texte alternatif](repositoryuser.JPG) 

## 🛠️ Services
### -  Interface `UserService`:
L’interface UserService définit les opérations métiers (logiques de service) liées à la gestion des utilisateurs (User) et des rôles (Role). Elle représente une couche d’abstraction entre les contrôleurs (qui gèrent les requêtes HTTP) et la couche de persistance (les repositories). Cette organisation respecte le principe de séparation des responsabilités, ce qui facilite la maintenance, les tests et l’évolutivité de l’application.  
 - Voici les méthodes déclarées dans l’interface UserService :
    - addNewUser(User user) : ajoute un nouvel utilisateur au système.
    - addNewRole(Role role) : ajoute un nouveau rôle.
    - findUserByUserName(String userName) : recherche un utilisateur selon son nom d'utilisateur.
    - findRoleByRoleName(String roleName) : recherche un rôle selon son nom.
    - addRoleToUser(String username, String roleName) : assigne un rôle donné à un utilisateur spécifique.
    - authenticate(String userName, String password) : authentifie un utilisateur en vérifiant son nom d'utilisateur et son mot de passe.

Cette interface permet d'assurer une cohérence dans les signatures des méthodes, indépendamment de leur implémentation concrète, ce qui est essentiel pour garantir une architecture solide.
  ![Texte alternatif](serviceuser.JPG) 
###  - Implémentation `UserServiceImpl`:  
La classe UserServiceImpl est l’implémentation concrète de l’interface UserService. Annotée avec @Service, elle est automatiquement détectée comme un composant métier par Spring, ce qui permet son injection dans d'autres composants. L’annotation @Transactional garantit que toutes les opérations effectuées dans ses méthodes s'exécutent dans le cadre d'une transaction unique, assurant ainsi la cohérence des données.  
La classe utilise l’annotation @AllArgsConstructor de Lombok pour injecter automatiquement les dépendances nécessaires (UserRepository et RoleRepository) via le constructeur.
 - Voici les principales méthodes de cette classe :
     - addNewUser(User user) : génère un identifiant unique (UUID) pour le nouvel utilisateur avant de l'enregistrer.
     - addNewRole(Role role) : enregistre un nouveau rôle dans la base de données.
     - findUserByUserName(String userName) / findRoleByRoleName(String roleName) : recherchent respectivement un utilisateur ou un rôle à partir de leur nom.
     - addRoleToUser(String username, String roleName) : établit une relation bidirectionnelle entre un utilisateur et un rôle en les ajoutant mutuellement dans leurs listes respectives.
     - authenticate(String userName, String password) : vérifie les identifiants d’un utilisateur. En cas d’échec, une exception est levée avec un message d’erreur "Bad credentials".

Ainsi, UserServiceImpl regroupe la logique métier de gestion des utilisateurs et des rôles, tout en déléguant les accès aux données aux repositories. Elle incarne la couche service typique dans une architecture Spring Boot bien structurée.
  ![Texte alternatif](userserviceImpl1.JPG) 
  ![Texte alternatif](userserviceImpl2.JPG) 
  ![Texte alternatif](userserviceImpl3.JPG) 

## Web:
###  - Classe `UserController`:
La classe UserController est un contrôleur REST qui permet d’exposer les services liés à l’entité User via des endpoints HTTP. Annotée avec @RestController, elle est automatiquement détectée par Spring pour gérer les requêtes web. Elle utilise l’annotation @Autowired pour injecter automatiquement une instance de UserService, qui contient la logique métier liée aux utilisateurs. La méthode user(@PathVariable String username) est mappée à l’URL /users/{username} grâce à l’annotation @GetMapping, ce qui signifie qu’elle est appelée lorsqu’une requête GET contenant un nom d’utilisateur est reçue. Cette méthode extrait le nom d’utilisateur de l’URL, utilise la méthode findUserByUserName() du service pour récupérer l’utilisateur correspondant depuis la base de données, puis retourne l’objet User en réponse, généralement sous forme de JSON. Ce contrôleur joue donc un rôle essentiel dans l’interaction entre le client et la couche service de l’application.

![Texte alternatif](usercontroller.JPG) 

## Classe Principale `JpaFsApplication`:
La classe JpaFsApplication constitue le point d’entrée principal de l’application Spring Boot. Annotée avec @SpringBootApplication, elle active la configuration automatique de Spring. La méthode main() lance l'application via SpringApplication.run(). La méthode start(), annotée avec @Bean, retourne un CommandLineRunner, ce qui permet d’exécuter automatiquement un bloc de code au démarrage de l’application. Ce bloc initialise deux utilisateurs (user1 et admin) avec un mot de passe, puis crée trois rôles (STUDENT, USER et ADMIN). Ensuite, ces rôles sont affectés aux utilisateurs : user1 reçoit les rôles STUDENT et USER, tandis que admin reçoit USER et ADMIN. Enfin, le code tente d’authentifier user1 avec son mot de passe, et en cas de succès, affiche son identifiant, nom d’utilisateur et les rôles associés.  
Ce mécanisme d’initialisation est pratique pour tester ou démarrer l’application avec des données par défaut.
  ![Texte alternatif](jpa1.JPG) 
  ![Texte alternatif](jpa2.JPG) 

## ⚙️ Configuration (`application.properties`):
Ce fichier application.properties contient les paramètres de configuration de l’application Spring Boot, en particulier pour la connexion à la base de données et le comportement de JPA/Hibernate.
  - server.port=8083 indique que l’application web sera accessible sur le port 8083.
  - La ligne spring.datasource.url=jdbc:mysql://localhost:3306/USERS_DB?createDatabaseIfNotExist=true configure la connexion à une base de données MySQL nommée USERS_DB, qui sera créée automatiquement si elle n'existe pas.
  - spring.datasource.username=root et spring.datasource.password= spécifient les identifiants de connexion à la base de données (ici, l'utilisateur root sans mot de passe).
  - spring.jpa.hibernate.ddl-auto=create indique à Hibernate de recréer les tables de la base de données à chaque démarrage de l’application, ce qui est utile en phase de développement.
  - spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MariaDBDialect informe Hibernate du dialecte SQL à utiliser, ici adapté à MariaDB (et compatible avec MySQL).
  - spring.jpa.show-sql=true permet d'afficher les requêtes SQL générées par Hibernate dans la console, pratique pour le débogage.
    ![Texte alternatif](configuration.JPG) 


  - Résultat Attendu
Au lancement de l’application :
    - Les utilisateurs et rôles sont insérés automatiquement.
    - Les relations entre utilisateurs et rôles sont établies.
    - On peut visualiser les tables via l’interface H2 (/h2-console), avec le JDBC URL jdbc:h2:mem:users_db.


 ## - Conclusion
Ce TP m'a permis de réaliser une application simple mais complète de gestion des utilisateurs et des rôles avec Spring Boot, JPA et une base de données MySQL.Les fonctionnalités essentielles de création, recherche, et association entre utilisateurs et rôles ont été implémentées avec succès.



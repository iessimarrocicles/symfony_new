---
title: "TEMA 5 — Model de dades amb Doctrine"
nav:
  - TEMA 5: 'Tema5.md'
---

# 📘 Tema 5 — Model de dades amb Doctrine

## 1. Introducció a Doctrine

### 1.1 Què és un ORM?

Un **ORM** (*Object Relational Mapping*) és un sistema que permet treballar amb bases de dades **relacionals** utilitzant **objectes**.  
Doctrine és l’ORM més utilitzat amb Symfony. Gràcies a ell:

- Podem connectar-nos a una base de dades relacional (MySQL, MariaDB, PostgreSQL, etc.).
- Les taules i columnes es mapegen automàticament amb classes i atributs PHP.
- Ens abstrau del gestor de base de dades concret, ja que tot es fa a través d’objectes.

**Avantatge principal:**  

- Aïllar l’aplicació del gestor de base de dades (MySQL, Oracle, PostgreSQL...). Symfony treballa amb objectes, i Doctrine s’encarrega de convertir-los a registres SQL o viceversa.

---

### 1.2 Configuració bàsica

Doctrine utilitza el fitxer `.env` per a definir la **connexió amb la base de dades**. 

L'estructura general serà aquesta:
```env
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
```

Exemple:

```env
DATABASE_URL="mysql://root@127.0.0.1:3306/daw_agenda"
```

Després, podem crear la base de dades amb la comanda:

```bash
php bin/console doctrine:database:create
```

Aquesta comanda llegirà la variable `DATABASE_URL` i crearà la base de dades indicada.

⚠️ **Atenció:** No s’ha de pujar l’arxiu `.env` al repositori, ja que conté dades sensibles (usuaris, contrasenyes…).

---

## 2. Entitats

Una **entitat** és una classe PHP que representa una taula de la base de dades.

Exemple d’entitat: `Contacte` amb atributs `id`, `nom`, `telefon` i `email`.

### 2.1. Crear Entitats

Per a la creació d'entitat utilitzem la següent instrucció:

```bash
php bin/console make:entity
```

L’assistent et demanarà:

- Nom de la classe
- Propietats del atributs: nom, tipus, tamany, si poden ser nuls o no

Exemple d’execució:

```
Class name of the entity to create or update:
 > Contacte
New property name:
 > nom
Field type:
 > string
Field length [255]:
 > 255
Can this field be null in the database? [no]:
 > no
```

Com a resultat, generarà el codi corresponent a la següent classe:

**Fitxer:** `src/Entity/Contacte.php`
```php
<?php

namespace App\Entity;

use App\Repository\ContacteRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ContacteRepository::class)]
class Contacte
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private ?string $nom = null;

    #[ORM\Column(length: 15)]
    private ?string $telefon = null;

    #[ORM\Column(length: 255)]
    private ?string $email = null;

    // Getters i setters
}

?>
```

El camp **codi** que abans utilitzavem com a identificador s’ha substituït per un **id** autonumèric que Doctrine genera automàticament com a clau principal. Així, només cal definir els altres camps (nom, telèfon i email) amb el seu tipus corresponent.

!!! tip "Symfony"
    L’execució de la instrucció genera automàticament les següents classes:
    
    -  El fitxer de l’entitat en `src/Entity/Contate.php`.
    -  El seu repositori en `src/Repository/ContacteRepository.php` (si no existia).

    Recorda no es modifica la base de dades. Per aplicar els canvis al esquema cal generar i executar la migració, tal com explicarem en l’apartat següent.

Els tipus de dades més habituals en Doctrine són:

- string (text amb longitud limitada)
- text (text llarg)
- integer (enter)
- boolean (cert/fals)
- float (nombre real)
- date, time, datetime (dates i hores).

---

### 2.2. Editar Entitats

Si després de crear una entitat volem canviar-ne l’estructura, podem editar la classe manualment o tornar a executar `php bin/console make:entity`, indicant el mateix nom d’entitat per afegir nous camps.

Una vegada fets els canvis, cal generar i executar una nova migració perquè Doctrine actualitze la base de dades amb la nova estructura.

---

### 2.3. Establir claus primàries

Per defecte, Doctrine crea automàticament un camp `id` autonumèric com a **clau primària**.  

Si volem utilitzar un altre camp com a identificador, hem de seguir aquests passos:

1. Eliminar l’atribut `id` i el seu *getter* de l’entitat.  
2. Afegir l’anotació següent al camp que volem usar com a clau primària:

```php

#[ORM\Id]
private $nomCamp;

```

- Si la clau és **composta** (format per diversos camps), afegirem aquesta anotació a cadascun d’ells.

Finalment, cal generar i aplicar una nova migració perquè la base de dades s’actualitze.

---

## 3. Actualitzar la base de dades

Una vegada definida l’entitat o les seues modificacions, és necessari sincronitzar els canvis amb la base de dades perquè l’estructura (taules, columnes, relacions...) coincidisca amb el model definit al projecte.

El flux de treball habitual és el següent:

Comandes:

```bash
# 1. Comprovar les diferències entre entitats i base de dades
php bin/console doctrine:schema:validate

# 2. Genera un fitxer de migració amb els canvis detectats
php bin/console make:migration

# 3. Aplica (i registra) les migracions pendents
php bin/console doctrine:migrations:migrate
```

Aquest procés crea o modifica les taules que corresponen a les entitats del projecte de manera controlada i reversible.

---

## 4. Funcionament intern

Per comprendre bé el funcionament, primer necessitem conéixer tres components bàsics que Doctrine utilitza internament.

- **ManagerRegistry:** és el punt central de connexió que sap quins gestors hi ha disponibles i com accedir (com la centraleta de Doctrine).
    - Serveix per obtindre l’EntityManager adequat. En la majoria de projectes només hi ha un, però aquest servei permetria gestionar diversos gestors de base de dades.

- **EntityManager:** és el gestor d’entitats.
    - S’encarrega de les operacions d’escriptura i gestió del cicle de vida de les entitats:
        - Sap com guardar, modificar, eliminar de la base de dades.
        - També permet accedir a buscar informació, mitjançant el repositori de cada entitat.
            - **Repositori**: és una classe especialitzada a obtenir entitats d'un tipus concret.

---

### 4.1. Com es relacionen

| **Element**       | **Rol**                                   | **Analogia**                 | **S’obté de**                   | **Exemple**                           |
|-------------------|--------------------------------------------|-------------------------------|----------------------------------|----------------------------------------|
| **ManagerRegistry** | Punt d’accés central als EntityManagers   | ☎️ Centraleta de gestors      | S’injecta automàticament        | `getManager()`, `getManagerForClass()` |
| **EntityManager** | Gestiona el cicle de vida de les entitats | 🧰 L’encarregat del magatzem | `ManagerRegistry->getManager()` | `persist()`, `flush()`, `remove()`     |
| **Repository**    | Busca i recupera entitats concretes        | 📗 Catàleg o venedor          | `EntityManager->getRepository()` | `find()`, `findBy()`, `findAll()`      |

---

### 4.2. Resum visual

```bash
ManagerRegistry
     │
     └──> EntityManager
              │
              ├── persist(), flush(), remove()
              └──> Repository (per entitat)
                        ├── find(), findBy()
                        └── mètodes personalitzats (createQueryBuilder)
```

!!! important "Symfony"
    En els nostres projectes només tenim una base de dades i un únic gestor, per això injectem directament **EntityManagerInterface**, que és més simple i net.

    No necessitem cridar `$registry->getManager()` perquè no hi ha cap dubte sobre quin manager s’ha d’utilitzar.

---

## 5. Persistir objectes

L’**EntityManager** és el component encarregat de gestionar el cicle de vida de les entitats, és a dir, les operacions que afecten directament la base de dades: inserir, modificar o eliminar registres.

Per a poder utilitzar-lo en un controlador, s’ha d’injectar la interfície **`EntityManagerInterface`**, que proporciona els mètodes necessaris per a treballar amb les entitats, com ara `persist()`, `remove()` o `flush()`.


### 5.1. Guardar

Exemple dins d'un mètode al controlador:

```php
<?php

use App\Entity\Contacte;
use Doctrine\ORM\EntityManagerInterface;


class ContacteController extends AbstractController
{
    ...

    #[Route('/contacte/afegir', name: 'afegir_contacte')]
    public function afegir(EntityManagerInterface $gestor)
    {
        $contacte = new Contacte();
        $contacte->setNom("Juan Cuesta");
        $contacte->setTelefon("659231544");
        $contacte->setEmail("juan@simarro.org");

        // Indiquem que volem guardar aquest objecte
        $gestor->persist($contacte);

        // S’executa la inserció
        $gestor->flush();

        return new Response("Contacte " . $contacte->getId() . " guardat.");
    }

    ...

?>
```

📝 Notes:

- És important col·locar el controlador d’inserció abans del de cerca, perquè si no, en accedir a la ruta `/contacte/inserir`, Symfony podria interpretar “inserir” com un paràmetre del controlador de cerca i executar-lo per error.

---

### 5.2. Modificar

```php
<?php

$contacte = $this->repositori->find($id);
$contacte->setTelefon("600000000");
$gestor->flush();

?>
```

### 5.3. Eliminar

```php
<?php

$contacte = $this->repositori->find($id);
$gestor->remove($contacte);
$gestor->flush();

?>
```

### 5.4. Captura errors

Si es produeix un error durant la inserció (si algun camp obligatori és nul o es duplica una clau primària), el mètode `flush()` llançarà una **excepció**.  

Per evitar que l’aplicació es trenque, podem **capturar l’excepció** i mostrar una resposta controlada a l’usuari:

```php
<?php

$gestor->persist($objecte);

try {
    $gestor->flush();
    return new Response("Objecte guardat");
} catch (\Exception $e) {
    return new Response("Error guardant objecte: " . $e->getMessage());
}

?>
```

Això garanteix un **tractament d’errors segur** i evita que es mostren missatges interns del servidor.

---

## 6. Consultar objectes

Doctrine crea automàticament una classe de **repositori** per a cada entitat (per exemple, ContacteRepository). Aquesta classe hereta de `ServiceEntityRepository` i permet:

- Recuperar dades.
- Fer cerques personalitzades.
- Accedir a l’EntityManager per fer canvis.

Per a poder fer **consultes** a la nostra base de dades, s’ha d’injectar al controlador la interfície `EntityManagerInterface`, que ens permet obtindre el repositori de qualsevol entitat i accedir a les seues dades.

---

En Symfony és una molt bona pràctica injectar l’EntityManagerInterface en el constructor. Això et permet tindre disponible l’entity manager en tots els mètodes del controlador.

Si tenim una entitat `Contacte`, podem obtindre el seu repositori per fer consultes així:

```php
<?php

class ContacteController extends AbstractController
{
    private $repositori;

    // Injecció de dependència al constructor
    public function __construct(private EntityManagerInterface $gestor)
    {
        // Instància única del repositori de Contacte
        $this->repositori = $this->gestor->getRepository(Contacte::class);
    }

    // A partir d'ací, qualsevol mètode pot utilitzar:
    // → $this->gestor     → per a persistir, eliminar o treballar amb entitats
    // → $this->repositori → per a fer consultes sobre Contacte

    ...

}

?>
```

Una vegada tenim el repositori, podem utilitzar diferents mètodes de consulta.

- **Mètode `find()`**
    
    Localiza un objecte per la clau primària (normalment l’id) que se li passa com a paràmetre.

    ```php
    $contacte = $this->repositori->find(1);
    ```

    ➡️ Aquesta instrucció buscarà el contacte amb id = 1.

    ---

- **Mètode `findOneBy()`**

    Localitza un únic objecte que complisca els criteris de cerca passats com a paràmetre (en forma d’array associatiu).

    ```php
    $contacte = $this->repositori->findOneBy(["telefon" => "687908070"]);
    ```

    ➡️ Aquesta consulta busca el contacte el telèfon del qual siga "687908070".

    Si volem indicar més d’un criteri, podem afegir-los a l’array:

    ```php
    $contacte = $this->repositori->findOneBy([
        "nom" => "Laura",
        "email" => "laura@gmail.com"
    ]);
    ```

    ---

- **Mètode `findBy()`**

    Localitza tots els objectes que complisquen els criteris passats com a paràmetre.
    Retorna un array d’objectes.

    ```php
    $contactes = $this->repositori->findBy(["comarca" => "La Costera"]);
    ```

    ➡️ Retorna un array amb tots els contactes que pertanyen a la comarca “La Costera”.

    També es pot afegir ordre i límit als resultats:

    ```php
    $contactes = $this->repositori->findBy(
        ["comarca" => "La Costera"],
        ["nom" => "ASC"], // ordre ascendent pel nom
        10,               // màxim 10 resultats
        0                 // des de l’índex 0
    );
    ```

    ---

- **Mètode `findAll()`**

    Recupera tots els objectes de la taula corresponent a l’entitat.

    ```php
    $contactes = $this->repositori->findAll();
    ```

    ➡️ Retorna un array amb tots els contactes existents en la base de dades.

    ---

### 6.1. Exemple complet en un controlador

```php
<?php

# En el constructor hem injectat EntityManagerInterface com a $gestor

#[Route('/contacte/{codi}', name:'fitxa_contacte', requirements: ['codi' => '\d+'])]
public function fitxa(int $codi)
{
    // 1) Buscar el contacte pel codi
    $contacte = $this->repositori->find($codi);

    if ($contacte)
        return $this->render('contacte/fitxa.html.twig', [
            'contacte' => $contacte,
            'codi' => $codi
        ]);
    else
        return $this->render('contacte/fitxa.html.twig', [
            'contacte' => null,
            'codi' => $codi
        ]);
}

?>
```

En aquest exemple, el controlador accedeix al repositori de Contacte i recupera un objecte segons el seu id. Si el troba, renderitza la plantilla fitxa.html.twig amb les dades del contacte; en cas contrari, envia el contacte buit.

---

### 6.2. Consultes avançades (QueryBuilder)

Els mètodes anteriors permeten buscar per igualtat directa.
Si volem fer cerques parcials o condicionals, podem afegir mètodes personalitzats al repositori utilitzant el QueryBuilder de Doctrine.

**Exemple 1: buscar contactes pel nom**

**Fitxer:** `src/Repository/ContacteRepository.php`

```php
<?php

public function findByName($text): array
{
    $qb = $this->createQueryBuilder('c')
        ->andWhere('c.nom LIKE :text')
        ->setParameter('text', '%' . $text . '%')
        ->getQuery();

    return $qb->execute();
}

?>
```

➡️ Aquesta consulta retornarà tots els contactes el nom dels quals continga el text indicat.

I podem cridar-la des del controlador:

```php
<?php

# En el constructor hem injectat EntityManagerInterface i asignat el repositori 

#[Route('/contacte/buscar/{text}', name:'buscar_contacte')]
public function buscar(string $text)
{
    $resultats = $this->repositori->findByName($text);

    return $this->render('llista_contactes.html.twig', [
        'contactes' => $resultats
    ]);
}

?>
```

Aquest controlador permet buscar contactes pel nom introduït en la URL, utilitzant la consulta definida al repositori. Després envia els resultats a una vista llista_contactes.html.twig per a mostrar-los.

---

**Exemple 2: buscar contactes majors d'una certa edat**

**Fitxer:** `src/Repository/ContacteRepository.php`

```php
<?php

public function findByEdatMajorQue(int $edat): array
{
    $qb = $this->createQueryBuilder('p')
        ->andWhere('p.edat > :edat')
        ->setParameter('edat', $edat)
        ->getQuery();

    return $qb->execute();
}

?>
```

!!! danger "Laboratori"
    Aquesta consulta no funcionarà al projecte actual perquè l'entitat Contacte no té definit el camp edat.

---

### 6.3. Altres formes

A més d’emprar el `QueryBuilder` o els mètodes per defecte del repositori (`find`, `findOneBy`, `findBy`, etc.), Doctrine permet realitzar consultes mitjançant dues alternatives addicionals:

- **DQL (Doctrine Query Language):** llenguatge similar a SQL però orientat a entitats en lloc de taules.
- **SQL natiu:** permet escriure consultes directament en SQL quan necessitem màxim control o optimització.

---

## 7. Relacions entre entitats

Fins ara, hem treballat amb operacions centrades en una sola entitat (taula).  

Aquesta secció explica com gestionar i operar amb més d'una entitat que estan relacionades entre si en la base de dades.

Hi ha dos tipus principals de relacions entre entitats:

| Tipus de Relació | Descripció | Implementació |
|------------------|-------------|----------------|
| **Molts a Un (Many-to-One)** | Engloba les relacions Un a Molts, Molts a Un i Un a Un. | Es reflecteix afegint una **Clau Aliena (Foreign Key)** en una de les entitats que referencia l'altra. |
| **Molts a Molts (Many-to-Many)** | Una entitat es pot relacionar amb múltiples instàncies de l'altra, i viceversa. | Requereix una **Taula Addicional** (taula pivot/d’unió) per emmagatzemar la relació entre els IDs de les dues entitats. |

Exemples:

| Tipus de relació | Exemple | Anotació |
|------------------|----------|----------|
| OneToOne         | Persona ↔ Passaport | `#[ORM\OneToOne]` |
| OneToMany / ManyToOne | Comarca ↔ Població | `#[ORM\OneToMany]`, `#[ORM\ManyToOne]` |
| ManyToMany       | Alumne ↔ Assignatura | `#[ORM\ManyToMany]` |

---

### 7.1. Relació Molts a Un (`ManyToOne`)

Com a exemple pràctic, crearem una relació *Molts a Un* entre l'entitat `Contacte` i una nova entitat `Comarca` (un contacte pertany a una comarca, i una comarca pot tenir molts contactes).

---

**Pas 1. Creació de l'Entitat `Comarca`**

Primer, generem l'entitat amb un ID autogenerat i un camp `nom`.

```bash
php bin/console make:entity

> Comarca
# ... Afegir el camp 'nom' com a string (255, no nullable)

```

Després, creem la taula `Comarca` a la base de dades mitjançant una migració:

```bash
# 1. Genera un fitxer de migració amb els canvis detectats
php bin/console make:migration

# 2. Aplica (i registra) les migracions pendents
php bin/console doctrine:migration:migrate
```

---

**Pas 2. Afegir la Relació a l'Entitat `Contacte`**

Editem l'entitat `Contacte` per afegir-li el camp de relació `comarca`:

```bash
php bin/console make:entity

> Contacte
# ...
New property name:
> comarca
Field type:
> relation
What class should this entity be related to?:
> Comarca
Relation type?:
> ManyToOne
Is the Contacte.comarca property allowed to be null (nullable)?:
> no
Do you want to add a new property to Comarca...?:
> no
```

Això afegeix un atribut a l'entitat `Contacte` com el següent:

```php
<?php

#[ORM\ManyToOne]
#[ORM\JoinColumn(nullable: false)]
private ?Comarca $comarca = null;

?>
```

---

**Pas 3. Actualització de la Base de Dades**

Després de fer els canvis, tornem a generar i executar la migració:

> ⚠️ Si hi ha registres de `Contacte` sense `Comarca`, cal eliminar-los o afegir-los abans per evitar errors de clau aliena.

```bash
# 1. Genera un fitxer de migració amb els canvis detectats
php bin/console make:migration

# 2. Aplica (i registra) les migracions pendents
php bin/console doctrine:migration:migrate
```

Això afegeix la **clau aliena** `comarca_id` a la taula `contacte`.

---

**Pas 4. Treballar amb Entitats Relacionades**

Una vegada les entitats estan relacionades, podem fer operacions d’inserció i de cerca.

---

**- Inserció d’Entitats Relacionades**

- Si la `Comarca` no existeix:

    1. Es creen els objectes `Comarca` i `Contacte`.  
    2. S’estableix la relació:
       ```php
       $contacte->setComarca($comarca);
       ```
    3. Es persisteixen ambdós objectes i es crida a `flush()`.

- Si la `Comarca` ja existeix:

    1. Es cerca l'objecte `Comarca` existent:
       ```php
       $comarca = $repositori->find(1);
       ```
    2. Es crea un nou `Contacte`.  
    3. S’estableix la relació amb l’objecte `Comarca`.  
    4. Es persisteix el `Contacte` i s’executa `flush()`.

---

**- Cerca d’Entitats Relacionades**

L’accés a les dades de l’entitat relacionada és directe des de l’entitat principal:

```php
<?php

// Obtenim l'objecte Contacte amb ID = 1
$contacte = $gestor->getRepository(Contacte::class)->find(1);

// Accedim a l'objecte Comarca relacionat i al seu nom
$nomComarca = $contacte->getComarca()->getNom();

?>
```

💡 **Nota (Lazy Loading):** Doctrine no recupera les dades de `Comarca` fins que no s’accedeix efectivament a elles (per exemple, quan s’executa `$contacte->getComarca()`).

---

## 8. Recursos i documentació

- Documentació oficial de Doctrine ORM  
👉 [https://symfony.com/doc/current/doctrine.html](https://symfony.com/doc/current/doctrine.html)

- Documentació d’anotacions i tipus de dades  
👉 [https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html](https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html)

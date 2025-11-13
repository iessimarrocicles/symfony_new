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

### 1.2 Configuració bàsica de Doctrine

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
    -  El seu repositori en `src/Repository/ContacteRepository.php` (si no existia)

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

## 3. Actualització de l'esquema de la base de dades

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
Per a una descripció més detallada de cada comanda i les seues diferències, consulteu l’**Annex 1**.

---

## 4. Repositoris

Doctrine crea automàticament una classe de **repositori** per a cada entitat (per exemple, `ContacteRepository`).

Aquesta classe hereta de `ServiceEntityRepository` i permet:

- Recuperar dades
- Fer cerques personalitzades
- Accedir a l’EntityManager per fer canvis

Exemple:

```php
<?php

$contactes = $contacteRepository->findAll();
$contacte = $contacteRepository->find($id);

?>
```

També podem definir mètodes personalitzats dins del repositori.

---

## 5. Persistir objectes

L’**EntityManager** és el component encarregat de gestionar el cicle de vida de les entitats, és a dir, les operacions que afecten directament la base de dades: inserir, modificar o eliminar registres.

Per a poder utilitzar-lo en un controlador, s’ha d’injectar la interfície **`EntityManagerInterface`**, que proporciona els mètodes necessaris per a treballar amb les entitats, com ara `persist()`, `remove()` o `flush()`.


### 5.1. Guardar

Exemple dins d’un controlador:

```php
<?php

use App\Entity\Contacte;
use Doctrine\ORM\EntityManagerInterface;


class ContacteController extends AbstractController
{
    ...

    #[Route('/contacte/afegir', name: 'afegir_contacte')]
    public function afegir(EntityManagerInterface $entityManager)
    {
        $contacte = new Contacte();
        $contacte->setNom("Frank Gallagher");
        $contacte->setTelefon("659231544");
        $contacte->setEmail("frank@simarro.org");

        // Indiquem que volem guardar aquest objecte
        $entityManager->persist($contacte);

        // S’executa la inserció
        $entityManager->flush();

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

$contacte = $contacteRepository->find($id);
$contacte->setTelefon("600000000");
$entityManager->flush();

?>
```

### 5.3. Eliminar
```php
<?php

$contacte = $contacteRepository->find($id);
$entityManager->remove($contacte);
$entityManager->flush();

?>
```

### 5.4. Captura errors

Si es produeix un error durant la inserció (si algun camp obligatori és nul o es duplica una clau primària), el mètode `flush()` llançarà una **excepció**.  

Per evitar que l’aplicació es trenque, podem **capturar l’excepció** i mostrar una resposta controlada a l’usuari:

```php
<?php

$entityManager->persist($objecte);

try {
    $entityManager->flush();
    return new Response("Objecte guardat");
} catch (\Exception $e) {
    return new Response("Error guardant objecte: " . $e->getMessage());
}

?>
```

Això garanteix un **tractament d’errors segur** i evita que es mostren missatges interns del servidor.

---

## 6. Consultar objectes

A l'hora d'obtenir objectes d'una taula, existeixen diferents mètodes que podem emprar.  
Aquests mètodes formen part del **repositori** de cada entitat, que és l'encarregat de gestionar l'accés a la base de dades per a aquesta classe.

- **Per fer consultes** (lectura de dades):  
  S’injecta el component **`ManagerRegistry`**, que ens permet obtindre el repositori de qualsevol entitat i accedir a les seues dades.

---

Per exemple, si tenim una entitat `Contacte`, podem obtindre el seu repositori per fer consultes així:

```php
<?php


// Injecció de dependència al constructor
public function __construct(ManagerRegistry $doctrine){

}

// Després on volem utilitzarla
$repositori = $doctrine->getRepository(Contacte::class);

?>
```


Una vegada tenim el repositori, podem utilitzar diferents mètodes de consulta.

- **Mètode `find()`**
    
    Localiza un objecte per la clau primària (normalment l’id) que se li passa com a paràmetre.

    ```php
    $contacte = $repositori->find(1);
    ```

    ➡️ Aquesta instrucció buscarà el contacte amb id = 1.

    ---

- **Mètode `findOneBy()`**

    Localitza un únic objecte que complisca els criteris de cerca passats com a paràmetre (en forma d’array associatiu).

    ```php
    $contacte = $repositori->findOneBy(["telefon" => "687908070"]);
    ```

    ➡️ Aquesta consulta busca el contacte el telèfon del qual siga "687908070".

    Si volem indicar més d’un criteri, podem afegir-los a l’array:

    ```php
    $contacte = $repositori->findOneBy([
        "nom" => "Laura",
        "email" => "laura@gmail.com"
    ]);
    ```

    ---

- **Mètode `findBy()`**

    Localitza tots els objectes que complisquen els criteris passats com a paràmetre.
    Retorna un array d’objectes.

    ```php
    $contactes = $repositori->findBy(["comarca" => "La Costera"]);
    ```

    ➡️ Retorna un array amb tots els contactes que pertanyen a la comarca “La Costera”.

    També es pot afegir ordre i límit als resultats:

    ```php
    $contactes = $repositori->findBy(
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
    $contactes = $repositori->findAll();
    ```

    ➡️ Retorna un array amb tots els contactes existents en la base de dades.

    ---

### 6.1. Exemple complet en un controlador

```php
<?php

#[Route('/contacte/{codi}', name:'fitxa_contacte', requirements: ['codi' => '\d+'])]
public function fitxa($codi, ManagerRegistry $doctrine)
{
    $repositori = $doctrine->getRepository(Contacte::class);
    $contacte = $repositori->find($codi);

    if ($contacte)
        return $this->render('fitxa_contacte.html.twig', ['contacte' => $contacte]);
    else
        return $this->render('fitxa_contacte.html.twig', ['contacte' => null]);
}

?>
```

En aquest exemple, el controlador accedeix al repositori de Contacte i recupera un objecte segons el seu id. Si el troba, renderitza la plantilla fitxa_contacte.html.twig amb les dades del contacte; en cas contrari, envia el contacte buit.

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

#[Route('/contacte/buscar/{text}', name:'buscar_contacte')]
public function buscar($text, ManagerRegistry $doctrine)
{
    $repositori = $doctrine->getRepository(Contacte::class);
    $resultats = $repositori->findByName($text);

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

public function findByEdatMajorQue($edat): array
{
    $qb = $this->createQueryBuilder('p')
        ->andWhere('p.edat > :edat')
        ->setParameter('edat', $edat)
        ->getQuery();

    return $qb->execute();
}

?>
```

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
$ php bin/console make:entity
> Comarca
# ... Afegir el camp 'nom' com a string (255, no nullable)
```

Després, creem la taula `Comarca` a la base de dades mitjançant una migració:

```bash
$ php bin/console make:migration
$ php bin/console doctrine:migration:migrate
```

---

**Pas 2. Afegir la Relació a l'Entitat `Contacte`**

Editem l'entitat `Contacte` per afegir-li el camp de relació `comarca`:

```bash
$ php bin/console make:entity
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
$ php bin/console make:migration
$ php bin/console doctrine:migration:migrate
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
$contacte = $doctrine->getRepository(Contacte::class)->find(1);

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

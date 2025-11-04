---
title: "TEMA 5 — Model de dades amb Doctrine"
nav:
  - TEMA 5: 'Tema5.md'
---

# 🧩 Tema 5 — Model de dades amb Doctrine

## 1. Introducció a Doctrine

### 1.1 Què és un ORM?

Un **ORM** (*Object Relational Mapping*) és un sistema que permet treballar amb bases de dades **relacionals** utilitzant **objectes**.  
Doctrine és l’ORM més utilitzat amb Symfony. Gràcies a ell:

- Podem connectar-nos a una base de dades relacional (MySQL, MariaDB, PostgreSQL, etc.).
- Les taules i columnes es mapegen automàticament amb **classes i atributs PHP**.
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
DATABASE_URL="mysql://root@127.0.0.1:3306/contactes"
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

Finalment, cal generar i aplicar una nova migració perquè la base de dades s’actualitze:

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

---

## 3. Creació de la base de dades i les taules

Una vegada definida l’entitat, hem d’actualitzar l’esquema de la base de dades.

Comandes:

```bash
# Comprovar les diferències entre entitats i base de dades
php bin/console doctrine:schema:validate

# Generar les instruccions SQL
php bin/console doctrine:schema:update --dump-sql

# Executar realment els canvis
php bin/console doctrine:schema:update --force
```

Aquest procés crea les taules corresponents a les entitats definides en el projecte.

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

## 5. Entity Manager

L’**EntityManager** és el component que gestiona la persistència dels objectes (inserir, modificar o eliminar registres).


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

        return new Response("Contacte " . $contacte->getId() . " inserit.");
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
$entityManager->persist($objecte);

try {
    $entityManager->flush();
    return new Response("Objecte inserit");
} catch (\Exception $e) {
    return new Response("Error inserint objecte");
}
```

Això garanteix un **tractament d’errors segur** i evita que es mostren missatges interns del servidor.

---

## 7. Relacions entre entitats

Doctrine permet definir diferents tipus de relacions:

| Tipus de relació | Exemple | Anotació |
|------------------|----------|----------|
| OneToOne         | Persona ↔ DNI | `#[ORM\OneToOne]` |
| OneToMany / ManyToOne | Comarca ↔ Poblacions | `#[ORM\OneToMany]`, `#[ORM\ManyToOne]` |
| ManyToMany       | Alumne ↔ Assignatura | `#[ORM\ManyToMany]` |

Exemple de relació **ManyToOne** (una comarca té molts contactes):

```php
#[ORM\ManyToOne(inversedBy: 'contactes')]
#[ORM\JoinColumn(nullable: false)]
private ?Comarca $comarca = null;
```

---

## 8. Exercicis pràctics

1. **Crea una entitat `Comarca`** amb atributs `id` i `nom`.
2. **Afegeix la relació** `ManyToOne` des de `Contacte` cap a `Comarca`.
3. **Crea la base de dades i les taules** amb les comandes de Doctrine.
4. **Inserix diversos contactes i comarques** mitjançant el controlador.
5. **Comprova** amb `phpMyAdmin` que s’han creat les claus foranes correctament.

---

## 9. Bones pràctiques

- Defineix sempre els **tipus** de dades i longituds de les columnes.
- Utilitza **noms d’entitat i propietats clars i significatius**.
- No modifiques manualment les taules en SQL; usa les comandes de Doctrine.
- Evita guardar arxius `.env` amb dades sensibles en el repositori.

---

## 10. Recursos i documentació

📘 Documentació oficial de Doctrine ORM  
👉 [https://symfony.com/doc/current/doctrine.html](https://symfony.com/doc/current/doctrine.html)

📗 Documentació d’anotacions i tipus de dades  
👉 [https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html](https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html)


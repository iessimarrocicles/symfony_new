---
title: "TEMA 5 — Pràctica"
nav:
  - Pràctica: 'Practica5.md'
---

# 💻 TEMA 5 — Pràctica

**Objectiu**

Aquest tema tracta sobre com gestionar el model de dades en Symfony utilitzant **Doctrine ORM**, incloent la creació d’entitats, relacions i operacions amb la base de dades.

**Punt de partida**

Partim de l’aplicació **`tendaNomAlumne`** creada i configurada en les pràctiques anteriors.

---

## Exercici 1

**Creació de la Base de Dades i Entitat `Seccio`**

Configurar la connexió a la base de dades i crear l’entitat dins l’aplicació.

1. Defineix la variable d’entorn `DATABASE_URL` al fitxer `.env`:
   ```env
   DATABASE_URL="mysql://root@127.0.0.1:3306/daw_tenda"
   ```
2. Crea la base de dades:
   ```bash
   php bin/console doctrine:database:create
   ```
3. Genera la nova entitat `Seccio`:
   ```bash
   php bin/console make:entity
   > Seccio
   ```
4. Afegeix els següents camps:
    - `nom` (string, 50)
    - `descripcio` (string, 255)
    - `any` (int)
    - `articles` (int)

5. Genera la migració i aplica-la:
   ```bash
   php bin/console make:migration
   php bin/console doctrine:migration:migrate
   ```

---

## Exercici 2

**Inserció de seccions des del Controlador**

1. Afegeix un nou mètode `inserir` al controlador `SeccioController` per inserir una secció de prova:
   ```php
   <?php

   // src/Controller/SeccioController.php
   #[Route('/seccions/inserir', name: 'inserir_seccio')]
   public function inserir(ManagerRegistry $doctrine): Response {
       $entityManager = $doctrine->getManager();
       $equip = new Equip();
       $equip->setNom("Simarrets");
       $equip->setCicle("DAW");
       $equip->setCurs("22/23");
       $equip->setNota(9);
       $equip->setImatge("equipPerDefecte.jpg");

       try {
           $entityManager->persist($equip);
           $entityManager->flush();
           return $this->render('equip/inserir_equip.html.twig', ['equip' => $equip]);
       } catch (\Exception $e) {
           return $this->render('equip/error.html.twig', ['error' => $e->getMessage()]);
       }
   }

   ?>
   ```

2. Crea la plantilla `templates/equip/inserir_equip.html.twig` per mostrar el resultat:
   ```twig
   <h2>Equip inserit correctament!</h2>
   <p>Nom: {{ equip.nom }}</p>
   <p>Cicle: {{ equip.cicle }}</p>
   <img src="{{ asset('images/' ~ equip.imatge) }}" alt="Imatge Equip">
   <a href="{{ path('inici') }}">Tornar a l'inici</a>
   ```

3. Afegeix un enllaç en el menú de la plantilla principal per accedir a `/seccio/inserir`.

---

## Exercici 3

**Inserció Múltiple**

1. Crea una nova ruta `/seccio/inserirmultiple` que inserisca 3 equips més.
2. Mostra un missatge de confirmació amb la plantilla `inserir_seccio_multiple.html.twig`.


---

## Exercici 4

**Entitat `Article` amb Relació ManyToOne a `Seccio`**

Crear una nova entitat `Article` relacionada amb `Seccio` (Molts a Un).

1. Genera l’entitat:
   ```bash
   php bin/console make:entity
   > Membre
   ```
2. Afegeix els camps:
    - `nom` (string, 100)
    - `preu` (float)
    - `stock` (int)
    - `imatge` (string, 255)
    - `seccio` (relació ManyToOne amb Seccio)

3. Genera i executa la migració:
   ```bash
   php bin/console make:migration
   php bin/console doctrine:migration:migrate
   ```

---

## Exercici 5

**Inserció d’un article amb Relació**

1. Afegeix un nou controlador per inserir articles:
   ```php
   <?php

   #[Route('/membre/inserir', name: 'inserir_membre')]
   public function inserirMembre(ManagerRegistry $doctrine): Response {
       $entityManager = $doctrine->getManager();

       $repositori = $doctrine->getRepository(Equip::class);
       $equip = $repositori->find(1); // Equip existent

       $membre = new Membre();
       $membre->setNom("Sarah");
       $membre->setCognoms("Connor");
       $membre->setEmail("sarahconnor@terminator.com");
       $membre->setTelefon("699888777");
       $membre->setDataNaixement(new \DateTime("1963-11-29"));
       $membre->setImatgePerfil("sarahconnor.jpg");
       $membre->setNota(9.7);
       $membre->setEquip($equip);

       $entityManager->persist($membre);
       $entityManager->flush();

       return $this->render('membre/inserir_membre.html.twig', ['membre' => $membre]);
   }

   ?>
   ```

2. Crea la vista `inserir_membre.html.twig`:
   ```twig
   <h2>Membre inserit correctament!</h2>
   <p>{{ membre.nom }} {{ membre.cognoms }}</p>
   <p>Equip: {{ membre.equip.nom }}</p>
   <img src="{{ asset('images/' ~ membre.imatgePerfil) }}" alt="Foto del membre">
   <a href="{{ path('inici') }}">Tornar a l'inici</a>
   ```

---

## Exercici 6

**Control de Versions**

1. Quan tot funcione, fes un commit amb Git:

```bash
git add --all
git commit -m "Tema 5: Doctrine"
git tag -a v5.0 -m "Versió 5.0 TendaNomAlumne Doctrine"
git push origin master --tags
```

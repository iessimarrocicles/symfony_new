---
title: "TEMA 5 — Pràctica"
nav:
  - Pràctica: 'Practica5.md'
---

# 💻 TEMA 5 — Pràctica

**Objectiu**

Aquest tema tracta sobre com gestionar el model de dades en Symfony utilitzant **Doctrine ORM**, incloent la creació d’entitats, relacions i operacions amb la base de dades.

---

## Exercici 1 

**Creació de la Base de Dades i Entitat `Equip`**

Configurar la connexió a la base de dades i crear l’entitat `Equip` dins l’aplicació **Equips**.

1. Defineix la variable d’entorn `DATABASE_URL` al fitxer `.env`:
   ```env
   DATABASE_URL="mysql://root@127.0.0.1:3306/equips"
   ```
2. Crea la base de dades:
   ```bash
   php bin/console doctrine:database:create
   ```
3. Genera la nova entitat `Equip`:
   ```bash
   php bin/console make:entity
   > Equip
   ```
4. Afegeix els següents camps:
    - `nom` (string, 50)
    - `cicle` (string, 10)
    - `curs` (string, 10)
    - `imatge` (string, 255)
    - `nota` (float o decimal amb 2 dígits, pot ser nul)

5. Genera la migració i aplica-la:
   ```bash
   php bin/console make:migration
   php bin/console doctrine:migration:migrate
   ```

---

## Exercici 2

**Inserció d’Equips des del Controlador**

1. Afegeix un nou mètode `inserir` al controlador `EquipsController` per inserir un equip de prova:
   ```php
   <?php

   // src/Controller/EquipsController.php
   #[Route('/equip/inserir', name: 'inserir_equip')]
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

3. Afegeix un enllaç en el menú de la plantilla principal per accedir a `/equip/inserir`.

---

## Exercici 3

**Inserció Múltiple**

1. Crea una nova ruta `/equip/inserirmultiple` que inserisca 3 equips més.
2. Mostra un missatge de confirmació amb la plantilla `inserir_equip_multiple.html.twig`.


---

## Exercici 4

**Entitat `Membre` amb Relació ManyToOne a `Equip`**

Crear una nova entitat `Membre` relacionada amb `Equip` (Molts a Un).

1. Genera l’entitat:
   ```bash
   php bin/console make:entity
   > Membre
   ```
2. Afegeix els camps:
    - `nom` (string, 100)
    - `cognoms` (string, 100)
    - `email` (string, 150)
    - `telefon` (string, 15)
    - `data_naixement` (date)
    - `imatge_perfil` (string, 255)
    - `nota` (float, pot ser nul)
    - `equip` (relació ManyToOne amb Equip)

3. Genera i executa la migració:
   ```bash
   php bin/console make:migration
   php bin/console doctrine:migration:migrate
   ```

---

## Exercici 5

**Inserció d’un Membre amb Relació**

1. Afegeix un nou controlador per inserir membres:
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
git commit -m "Commit Doctrine EquipsNomAlumne"
git tag -a versio4.0 -m "Versió 4.0 EquipsNomAlumne Doctrine"
git push origin master --tags
```
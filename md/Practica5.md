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

2. Crea la base de dades.

3. Genera la nova entitat `Seccio`.

4. Afegeix els següents camps:
    - `nom` (string, 50)
    - `descripcio` (string, 255)
    - `any` (integer)
    - `imatge` (string, 100)

5. Genera la migració i aplica-la.

---

## Exercici 2

**Inserció de seccions des del Controlador**

1. Afegeix un nou mètode `afegir` al controlador `SeccioController` per inserir totes les seccions.

2. Crea la plantilla `templates/equip/afegir.html.twig` per mostrar el resultat.

3. Crea la plantilla `templates/equip/error.html.twig` per mostrar el resultat en cas d'error.

4. Afegeix un enllaç en el menú de la plantilla principal `_menu.html.twig` per accedir a `/seccions/afegir`.

---

## Exercici 3

**Entitat `Article` amb Relació ManyToOne a `Seccio`**

Crear una nova entitat `Article` relacionada amb `Seccio` (Molts a Un).

1. Genera l’entitat corresponent.

2. Afegeix els camps:
    - `nom` (string, 100)
    - `preu` (float)
    - `stock` (integer)
    - `imatge` (string, 255)
    - `seccio` (relació ManyToOne amb Seccio)
        - Quan l’assistent ens pregunta si volem afegir una propietat en la classe Seccio per poder accedir als seus articles des del codi. En este cas, acceptem la proposta escrivint yes i definim el nom de la nova propietat com articles. Això ens fara una relació bidireccional. D’esta manera es genera automàticament una relació bidireccional: la classe Article tindrà la propietat seccio, i la classe Seccio disposarà d’una col·lecció articles amb els mètodes getArticles(), addArticle() i removeArticle().

3. Genera i executa la migració.

---

## Exercici 4

**Inserció d’un article amb Relació**

1. Afegeix un nou controlador `ArticleController.php` per inserir articles.

2. Crea la plantilla `templates/article/afegir.html.twig` per mostrar el resultat.

3. Crea la plantilla `templates/equip/error.html.twig` per mostrar el resultat en cas d'error.

4. Afegeix un enllaç en el menú de la plantilla principal `_menu.html.twig` per accedir a `/articles/afegir`.

---

## Exercici 5

**Modificar les vistes perquè utilitzen dades reals de la base de dades**

1. Actualitza totes les vistes i controladors que encara utilitzen el servei antic DadesSeccioServei.php (creat en el tema anterior).

2. Substitueix les crides al servei per consultes reals a Doctrine mitjançant el repositori de Seccio i Article.

3. Revisa totes les plantilles Twig per assegurar que:
    - Les dades que es mostren provenen dels objectes Seccio i Article recuperats amb Doctrine.
    - Ja no s'utilitzen arrays ni llistes simulades.

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

---

## Resultat final esperat

- Configurar Doctrine i crear la base de dades.
- Generar l’entitat Seccio i aplicar migracions.
- Inserir dades des del controlador i mostrar-les amb Twig.
- Crear l’entitat Article i establir una relació bidireccional ManyToOne amb Seccio.
- Inserir articles vinculats a seccions existents.
- Afegir enllaços al menú per facilitar la navegació.
- Totes les vistes i controladors deixen d’utilitzar el servei temporal del Tema 4, mostrant informació real guardada en MySQL.

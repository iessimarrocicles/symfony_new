---
title: "TEMA 3 — Pràtica 3"
nav:
  - Pràctica 1: 'Practica3.md'
---

# 🧩 TEMA 3 — Pràtica 1

**Objectiu**

Aprendre a crear i utilitzar **plantilles Twig** per generar vistes dinàmiques a Symfony, reutilitzant components i aplicant bones pràctiques en la construcció de la interfície.

**Punt de partida**

Partim de l’aplicació **`retailNomAlumne`** creada i configurada en les pràctiques anteriors.

---

## Exercici 1

Tria una **plantilla Bootstrap** d’una de les pàgines web recomanades en el curs d’aules.  
Preferentment, escull-ne una que tinga un **apartat del menú principal** implementat amb un **dropdown**.

1. Descomprimeix la plantilla descarregada i copia tots els seus fitxers dins de la carpeta `public` del projecte Symfony.
2. Canvia-li el nom al fitxer `index.html` per evitar conflictes.  
   Per exemple:
   ```
   index.html.old
   ```

---

## Exercici 2

1. Crea una nova **plantilla base** (`base.html.twig`) dins de la carpeta `templates`.
2. Afig-hi les seccions principals amb **blocs Twig** (`{% block ... %}`) per a:
    - `title`
    - `stylesheets`
    - `body`
    - `javascripts`
3. Importa en aquesta plantilla tots els recursos CSS i JS que necessite la plantilla Bootstrap triada.
4. A partir de la plantilla base, crea una **subplantilla d’inici** (`inici.html.twig`) que herete de la base i mostre:
    - El nom del projecte.
    - Un breu text de presentació.
    - Una imatge o element visual destacat.

---

## Exercici 3

1. Modifica la plantilla d’inici perquè, en fer **clic sobre la imatge d’una secció**, s’òbriga la pàgina de les dades d’eixa secció en concret (utilitzant el **nom de la ruta** corresponent).
2. Modifica la plantilla de les dades de la secció perquè aparega un **enllaç amb una imatge** que permeta tornar a la pàgina d’inici.
3. Comprova que totes les rutes funcionen correctament.

---

## Resultat final esperat

- L’aplicació utilitza **plantilles Twig** per a totes les vistes.
- La navegació entre pàgines funciona correctament mitjançant rutes Symfony.
- S’han aplicat estils **Bootstrap** a les vistes.
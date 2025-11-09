---
title: "TEMA 3 — Pràctica"
nav:
  - Pràctica: 'Practica3.md'
---

# 💻 TEMA 3 — Pràctica

**Objectiu**

Aprendre a crear i utilitzar **plantilles Twig** per generar vistes dinàmiques a Symfony, reutilitzant components i aplicant bones pràctiques en la construcció de la interfície.

**Punt de partida**

Partim de l’aplicació **`tendaNomAlumne`** creada i configurada en les pràctiques anteriors.

---

## Exercici 1

En aquest exercici tens dues opcions de disseny per començar a treballar amb Bootstrap:

- **Opció A:** Utilitzar una plantilla Bootstrap ja creada (descàrrega d’un portal).
    - [Bootstrap Made](https://bootstrapmade.com/)  
    - [Theme Wagon](https://themewagon.com/)  
- **Opció B:** Crear la teua pròpia plantilla base amb Bootstrap, amb menú principal i peu de pàgina.

Pots triar lliurement entre aquestes dues opcions segons el teu nivell o preferències. L’objectiu és treballar amb una base Bootstrap funcional i estructurada que servirà per a les vistes Twig posteriors.

Requisits generals:

1. Crea un fitxer `base.html.twig` amb una estructura HTML senzilla que incloga:
    - Un menú principal amb almenys un element de tipus dropdown.
    - Un cos principal per al contingut.
    - Un peu de pàgina amb informació bàsica (autor, any, enllaços legals...).
2. Importa Bootstrap des d’un CDN o mitjançant els fitxers locals.
3. Assegura’t que la teua plantilla siga responsive i que els estils es mostren correctament.

---

## Exercici 2

1. A partir del fitxer `base.html.twig`, defineix les seccions principals amb **blocs Twig** (`{% block ... %}`) per a:
    - `title`
    - `stylesheets`
    - `contingut`
    - `javascripts`
2. Importa en aquesta plantilla tots els recursos CSS i JS que necessite la plantilla Bootstrap triada (o creada per tu).
3. A partir de la plantilla base, crea una **subplantilla d’inici** (`inici.html.twig`) que herete de la base i mostre:
    - El nom del projecte.
    - Un breu text de presentació.
    - Una imatge o element visual destacat.

---

## Exercici 3

1. Crea una carpeta `templates/_fragments` i extrau com a mínim:
    1. `_menu.html.twig` -> menú de navegació.
    2. `_footer.html.twig` -> peu de pàgina.
2. Crea una carpeta `templates/seccions`:
    1. Llistat `templates/seccio/llistat.html.twig`: taula responsive amb codi, nom, descripció, any i nombre d'articles i un enllaç "Veure" en fer clic, s'obrirà la pàgina de les dades d’eixa secció en concret (utilitzant el **nom de la ruta** corresponent).
    2. Detall `templates/seccio/detal.html.twig`: capçalera amb una imatge i dades; grid de targetes d'articles. Afig un enllaç amb una imatge per tornar al llistat de seccions.
3. Les rutes mínimes: `inici`, `llistat_seccions` i `dades_seccio` (amb un parametre codi).
    1. Comprova el funcionament amb `bin/console debug:router` i el profiler.

---

## Exercici 4

**Control de Versions**

1. Quan tot funcione, fes un commit amb Git:

```bash
git add --all
git commit -m "Tema 3: Plantilles TWIG"
git tag -a v3.0 -m "Versió 3.0 TendaNomAlumne Plantilles <TWIG>"
git push origin master --tags
```

---

## Resultat final esperat

- L’aplicació utilitza **plantilles Twig** per a totes les vistes.
- La navegació entre pàgines funciona correctament mitjançant rutes Symfony.
- S’han aplicat estils **Bootstrap** a les vistes.

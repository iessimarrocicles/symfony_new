---
title: "TEMA 4 — Pràctica"
nav:
  - Pràctica: 'Practica4.md'
---

# 💻 TEMA 4 — Pràctica

**Objectiu**

Treballar amb el **contenidor de serveis** de Symfony i aplicar el concepte d’**injecció de dependències** per a gestionar dades i recursos dins de l’aplicació.

**Punt de partida**

Partim de l’aplicació **`tendaNomAlumne`** creada i configurada en les pràctiques anteriors.

---

## Exercici 1

**Revisó ràpida del contenidor i autowiring**

1. Obri `config/services.yaml` i localitza els *defaults*:
   ```yaml
   services:
     _defaults:
       autowire: true
       autoconfigure: true
       public: false
   ```
   Això habilita que Symfony **detecte i injecte** automàticament les dependències per **tipus** (classes/interfícies), i registre serveis segons el seu tipus.

---

## Exercici 2

**Ús bàsic de LoggerInterface en un controlador**

1. Edita el fitxer `src/Controller/IniciController.php` i injecta LoggerInterface al constructor de la classe. 
       - Fes que cada vegada que un usuari accedisca a la pàgina d’inici (/), es guarde un registre amb la data, l’adreça IP i l’URL visitada.
2. Entra a http://symfony.tendaNomAlumne/ i comprova que s’ha creat o actualitzat el fitxer `var/log/dev.log`. Hauries de veure una línia semblant a:
   ```bash
   [info] Nou accés d’usuari: {"ip":"127.0.0.1","moment":"2025-11-12 09:30:00","ruta":"/"}
   ```
3. Obri el fitxer de log i verifica que cada vegada que recarregues la pàgina, s’afegeix un nou registre d’accés.

---

## Exercici 3

**Servei propi `ServeiDadesSeccio`**

Crea un servei anomenat `ServeiDadesSeccio` dins de la carpeta `src/Service` (si no existeix, crea-la primer). Aquest servei s’encarregarà de gestionar totes les dades relacionades amb les seccions, incloent ara també la imatge identificadora de cadascuna.

1. **Trasllat de dades:**
     - Copia l’array de seccions que fins ara estava declarat dins de `src/Controller/SeccioController`.
     - Porta’l al nou servei `ServeiDadesSeccio`, on quedarà com una propietat privada de la classe.
     - Afig a l'array un nou camp anomenat `imatge` per cada secció:
         - Descarrega una imatge representativa o un logo per a cada secció (p. ex. roba, calçat...).
         - Guarda-les a la carpeta `asset/imgs/seccions/` del projecte.
         - Assegura’t que els noms dels fitxers siguen senzills i sense espais (per exemple: roba.jpg, calcat.jpg...).

2. **Mètodes públics:**
     - Defineix un mètode `llistarSeccions()` que retorne tot l’array de seccions.
     - Afig també un mètode `obtenirSeccioPerCodi(int $codi)` que, donat un codi, retorne la secció corresponent o `null` si no existeix.

3. **Ús del servei en els controladors:**
     - En `src/Controller/SeccioController`, elimina l’array local i injecta el nou servei `ServeiDadesSeccio` mitjançant el constructor. Substitueix totes les crides a l’array per les crides als mètodes del servei (`llistarSeccions()` i `obtenirSeccioPerCodi()`).

4. **Mostrar les imatges en les plantilles Twig**
     - A la vista de la secció `detall.html.twig`, mostra la imatge utilitzant la funció asset de Twig i l’operador de concatenació ~.
         - Exemple: `{{ asset('img/' ~ seccio.imatge) }}` (no és la solució, només una pista sintàctica).

Amb aquest canvi, el teu projecte utilitzarà un **servei centralitzat** per gestionar les dades de seccions, millorant la modularitat i mantenibilitat del codi.

---

## Exercici 4

**Control de Versions**

1. Quan tot funcione, fes un commit amb Git:

```bash
git add --all
git commit -m "Tema 4: Contenidor de Serveis"
git tag -a v4.0 -m "Versió 4.0 TendaNomAlumne Contenidor de Serveis"
git push origin master --tags
```

---

## Resultat final esperat

- L’aplicació utilitza serveis predefinits pel sistema.
- L'aplicació utilitza serveis propis.
- Mostra imatges a les plantilles de Twig.

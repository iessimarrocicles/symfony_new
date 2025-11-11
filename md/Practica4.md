---
title: "TEMA 4 — Pràctica"
nav:
  - Pràctica: 'Practica4.md'
---

# 💻 TEMA 4 — Pràctica

**Objectiu**

Treballar amb el **contenidor de serveis** de Symfony i aplicar el concepte d’**injecció de dependències** per a gestionar dades i recursos dins de l’aplicació.

---

## Exercici 1 

**Mostrar les seccions en la pàgina inicial**

- Modifica la plantilla `inici.html.twig` perquè **reba com a paràmetre** el llistat de les seccions enviat des de `IniciController::inici()`.
- Mostra aquestes seccions en la plantilla amb el **format indicat pel professorat**.

💡 *Pista:* hauràs de passar el llistat com a variable al mètode `render()` del controlador.

---

## Exercici 2 

**Afegir imatges a cada secció**

1. Descarrega’t una **imatge o logo representatiu** per a cadascuna de les seccions del teu llistat.
2. Afig un **nou camp en les dades de les seccions** amb el nom de la imatge corresponent.
3. Modifica la plantilla perquè, en mostrar les dades de cada secció, es veja també la **imatge associada**.
4. Recorda que per **concatenar valors en Twig** s’utilitza el símbol `~`.

💡 *Exemple conceptual:* `{{ asset('img/' ~ seccio.imatge) }}` (no és la solució, només una pista sintàctica).

---

## Exercici 3

**Bind de serveis i comprovació**

1. Edita l’arxiu `config/services.yaml` i crea un **bind** perquè, sempre que un servei reba un argument anomenat `$dadesSeccions`, Symfony li assigne una **instància del servei `ServeiDadesSeccions`** creat anteriorment.
   - La ruta al servei serà: `@App\Service\ServeiDadesSeccions`
2. Comprova que aquest **bind** funciona correctament en les classes `IniciController` i `RetailController`.
3. Quan tot funcione, fes un **commit amb Git** amb el missatge:
   ```bash
   git commit -m "Commit Serveis TendaNomAlumne"
   ```
4. Crea una **etiqueta (tag)** amb el nom i comentari següents:
   ```bash
   git tag -a v3.0 -m "Versió 3.0 TendaNomAlumne Serveis"
   ```

---

## Resultat final esperat

- Assegura’t que totes les modificacions funcionen correctament.
- Puja el projecte actualitzat al teu **repositori GitHub o GitLab**.
- Verifica que la versió `3.0` apareix correctament en les etiquetes del projecte.
- Esquelet de treball (estructura)
   ```bash
   projecte/
   ├─ assets/
   │  ├─ styles/, imgs/, imgs/seccio/
   │  └─ app.js
   └─ src/
   │  ├─ Controller/IniciController.php
   │  └─ Controller/SeccioController.php
   └─ templates/
      ├─ partials/
      │  ├─ _menu.html.twig
      │  ├─ _peu.html.twig
      ├─ seccio/
      │  ├─ detall.html.twig
      │  └─ llistat.html.twig
      ├─ base.html.twig
      └─ inici.html.twig
   ```

---

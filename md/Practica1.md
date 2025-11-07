---
title: "TEMA 1 — Pràctica"
nav:
  - Pràctica: 'Practica1.md'
---

# 💻 TEMA 1 — Pràctica

**Objectiu**

En aquesta pràctica crearem un projecte inicial. Per això, hauràs de tindre **operativa la màquina virtual** amb **XAMPP** i **Composer** instal·lats.

## Exercici 1

**Creació del projecte inicial**

- **Crea un nou projecte Symfony** anomenat `tendaNomAlumne`, que anirem completant posteriorment, dins de la carpeta de treball:
  ```bash
  /var/www/html/symfony
  ```
  Aquest projecte partirà d'un esquelet mínim i els components bàsics `orm`, `twig`, `twig-bundle`, `asset` i `asset-mapper`.

---

- **Modifica el fitxer** `/etc/hosts` i afegeix una entrada per a crear un **Virtual Host**, de manera que pugues accedir al projecte mitjançant la següent adreça:
  ```bash
  http://symfony.tendaNomAlumne
  ```
---

- **Inicia un repositori Git** del teu projecte.
  ```powershell
  git init
  git add .
  git commit -m "Commit Inicial Tenda Alumne"
  ```

### Explicació

Quan pugem un projecte PHP a GitHub, la carpeta vendor no es puja perquè està exclosa mitjançant .gitignore. Aquesta carpeta conté totes les dependències gestionades per Composer, i per tant, no s’ha de versionar.

!!! warning "Carpeta vendor no inclosa al repositori"

Per això, si descarreguem el projecte en un altre ordinador o entorn de treball, la carpeta vendor no existirà.

En eixe cas, és necessari reinstal·lar totes les dependències executant la següent instrucció dins de la carpeta del projecte:

```bash
composer install
```

Aquesta ordre:

- Llegix el fitxer composer.json.
- Descarrega totes les llibreries necessàries.
- Reconstrueix la carpeta vendor.

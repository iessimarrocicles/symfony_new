---
title: "TEMA 2 — Pràctica"
nav:
  - Pràctica: 'Practica2.md'
---

# 🧩 TEMA 2 — Pràctica

**Objectiu**

En aquesta pràctica treballarem amb el **patró MVC** a Symfony, creant els primers controladors i rutes personalitzades dins del projecte `retailNomAlumne`.

**Punt de partida**

Partim de l’aplicació **`retailNomAlumne`** creada en la pràctica anterior.  

---

## Exercici 1

**Creació de controladors bàsics**

Afegeix dins de la carpeta `src/Controller` els següents controladors:

---

### 1.1. Controlador **IniciController**

1. Crea una classe anomenada `IniciController`.
2. Afig un **mètode** anomenat `inici`, associat a la **ruta arrel** `/`.
3. Dona-li el **nom de ruta** `inici`.
4. Mostra com a **resposta** el missatge següent:

   ```
   Projecte Gestió Retail de nom de alumne
   ```

---

### 1.2. Controlador **SeccioController**

1. Crea una classe anomenada `SeccioController`.
2. Afig un **mètode** anomenat `seccio`, associat a la **ruta** `/seccio/{codi}`.
3. Dona-li el **nom de ruta** `dades_seccio`.
4. El paràmetre `{codi}` serà el **codi alfanumèric** d’una secció.

A la mateixa classe, declara un **array** amb quatre seccions, cadascuna amb els següents camps:

- `codi`
- `nom`
- `descripcio`
- `any`
- `articles` (array amb quatre noms d’articles)

**Exemple orientatiu de secció:**

```php
array(
    "codi" => "1",
    "nom" => "Roba",
    "descripcio" => "Secció de roba",
    "any" => "2026",
    "articles" => array("Pantalons", "Camisa", "Jersei", "Jaqueta")
)
```

El controlador ha de **buscar la secció** que coincidisca amb el codi rebut per paràmetre:

- Si la troba, mostrarà totes les seues dades en el format que preferisques (text pla o HTML).
- Si **no** la troba, mostrarà el missatge:

  ```
  No s’ha trobat la secció: CodiSeccio
  ```

---

## Resultat final esperat

- El projecte mostra correctament la pàgina d’inici i les seccions.
- El codi segueix el patró **MVC**.

---
title: "TEMA 2 — Pràctica"
nav:
  - Pràctica: 'Practica2.md'
---

# 💻 TEMA 2 — Pràctica

**Objectiu**

En aquesta pràctica treballarem amb el **patró MVC** a Symfony, creant els primers controladors i rutes personalitzades dins del projecte `tendaNomAlumne`.

**Punt de partida**

Partim de l’aplicació **`tendaNomAlumne`** creada en la pràctica anterior.  

---

## Exercici 1

**Creació de controladors bàsics**

Afegeix dins de la carpeta `src/Controller` els següents controladors:

---

### Controlador **IniciController**

1. Crea una classe anomenada `IniciController`.
2. Afig un **mètode** anomenat `inici`, associat a la **ruta arrel** `/`.
3. Dona-li el **nom de ruta** `inici`.
4. Mostra com a **resposta** el missatge següent:

   ```bash
   Projecte Gestió Tenda de [nom i cognoms de l'alumne]
   ```

---

### Controlador **SeccioController**

1. Crea una classe anomenada `SeccioController`.
2. Afig un **mètode** anomenat `seccio`, associat a la **ruta** `/seccio/{codi}`.
3. Dona-li el **nom de ruta** `dades_seccio`.
4. El paràmetre `{codi}` serà un **número identificatiu** d’una secció.

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

  ```bash
  No s’ha trobat la secció: CodiSeccio
  ```

---

## Exercici 2

**Control de Versions**

Una vegada hages finalitzat les modificacions, afegeix els fitxers canviats a l’àrea de preparació (staging area) de Git i crea un commit amb el missatge:

```bash
Commit MVC TendaNomAlumne
```

Tot seguit, crea una etiqueta (tag) amb el nom versio1.0 i el comentari:

```bash
Versió 1.0 TendaNomAlumne MVC
```

Amb això, quedaran registrats tant l’estat final del projecte com la marca de versió corresponent.

```bash
# 1. Afegir tots els canvis a l’àrea de preparació
git add --all

# 2. Crear el commit amb el missatge indicat
git commit -m "Commit MVC TendaNomAlumne"

# 3. Crear l’etiqueta (tag) versio1.0 amb comentari
git tag -a versio1.0 -m "Versió 1.0 TendaNomAlumne MVC"

# 4. Pujar tant el commit com l’etiqueta al repositori remot
git push origin master --tags
```

### Utilitat etiqueta (tag)

Una etiqueta en Git és com posar un marcador en un punt concret de la història del projecte.
Normalment s’utilitza per:

1. Identificar versions importants del programari, com ara:
    - v1.0
    - v1.1
    - v2.0

2. Poder tornar a eixa versió en qualsevol moment per a revisar el codi, restaurar-lo o comparar canvis.

3. Distribuir o entregar treball amb una versió estable i final.

### Versions anteriors

Per tornar a una versió anterior identificada per una etiqueta

**Opció 1. Només consultar o executar el codi d’eixa versió (sense modificar res)**

```bash
git checkout versio1.0
```

- Pots mirar el codi, compilar-lo, executar-lo, etc.
- Però si fas commits ací, no estaran associats a cap branca.

**Opció 2. Crear una branca nova a partir del tag (per continuar treballant des d’eixa versió)**

```bash
git checkout -b nova_versio1.0 versio1.0
```

- Això crea una branca nova anomenada `nova_versio1.0` basada en eixe tag, i ja pots fer commits amb normalitat.

**Opció 3. Reposicionar la branca actual a eixa versió (tornar enrere “de veritat”)**

```bash
git reset --hard versio1.0
```

- Això canviarà l’historial si fas push després.

---

## Resultat final esperat

- El projecte mostra correctament la pàgina d’inici i les seccions.
- El codi segueix el patró **MVC**.

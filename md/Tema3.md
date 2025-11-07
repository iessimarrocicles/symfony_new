---
title: "TEMA 3 — Plantilles amb Twig"
nav:
  - TEMA 3: 'Tema3.md'
---

# 📘 TEMA 3 — Plantilles amb Twig

## 1. Definint plantilles de vistes amb Twig

Els controladors que hem vist fins ara generaven HTML directament al codi PHP. Per a separar el disseny de la lògica, Symfony utilitza **Twig**, un motor de plantilles que permet treballar amb fitxers `.html.twig` dins la carpeta `templates/`.

Twig ens ajuda a mantindre una **estructura MVC clara**, on el controlador prepara les dades i la vista s'encarrega de mostrar-les.

---

## 1.1. La nostra primera plantilla

Veurem com renderitzar una plantilla amb Twig. 

**Controlador**

El controlador ha d’heretar d'`AbstractController`, la qual ens ofereix el mètode `render()` per mostrar vistes. 

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class IniciController extends AbstractController
{
    #[Route('/', name: 'inici')]
    public function inici(): Response
    {
        return $this->render('inici.html.twig');
    }
}

?>
```

**Vista (`templates/inici.html.twig`)**

Les plantilles (vistes) s’emmagatzemen a la carpeta `templates/` i utilitzen l’extensió `.html.twig`.

```html
<html>
  <body>
    <h1>Contactes</h1>
    <h2>Benvingut al web de contactes.</h2>
    <p>Pàgina d'inici</p>
  </body>
</html>
```

💡 Encara que aquesta vista és totalment estàtica, ja es processa a través de Twig i pot rebre dades del controlador.

---

## 1.2. Plantilles amb parts variables

Podem passar informació del controlador a la plantilla mitjançant un **array associatiu**:

**Controlador**

```php
<?php

#[Route('/contacte/{codi}', name: 'fitxa_contacte')]
public function fitxa(int $codi): Response
{
    $contacte = [
        'nom' => 'Anna Puig',
        'telefon' => '612345678',
        'email' => 'anna@simarro.org'
    ];

    return $this->render('fitxa_contacte.html.twig', [
        'contacte' => $contacte
    ]);
}

?>
```

**Vista (`fitxa_contacte.html.twig`)**

```html
<h1>Fitxa de contacte</h1>
<ul>
  <li><strong>{{ contacte.nom }}</strong></li>
  <li>Telèfon: {{ contacte.telefon }}</li>
  <li>E-mail: {{ contacte.email }}</li>
</ul>
```

💡 La notació `{{ ... }}` continua sent la forma estàndard de mostrar variables. Les dades s’escapen automàticament per evitar atacs de seguretat.

---

## 1.3. Plantilles base i herència

Twig permet **reutilitzar estructures HTML comunes** (com capçalera, menú o peu de pàgina) gràcies als **blocs i herència**.

**Plantilla base (`base.html.twig`)**

```twig
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>{% block title %}Aplicació Symfony{% endblock %}</title>
  </head>
  <body>
    <header><h1>Contactes</h1></header>

    <main>
      {% block contingut %}
        <!-- Contingut específic de cada pàgina -->
      {% endblock %}
    </main>

    <footer>
      <p>&copy; {{ "now"|date("Y") }} IES Lluís Simarro</p>
    </footer>
  </body>
</html>
```

**Plantilla filla (`inici.html.twig`)**

```twig
{% extends 'base.html.twig' %}

{% block title %}Inici{% endblock %}

{% block contingut %}
  <h1>Benvingut a la nostra aplicació</h1>
{% endblock %}
```

---

## 1.4. Comentaris, estructures de control i filtres

### 1.4.1. Comentaris

```twig
{# Aquest comentari no apareix al navegador #}
```

### 1.4.2. Condicionals

```twig
{% if contacte.email %}
  <p>Correu: {{ contacte.email }}</p>
{% else %}
  <p>Sense correu electrònic</p>
{% endif %}
```

### 1.4.3. Bucles

```twig
<ul>
  {% for contacte in contactes %}
    <li>{{ contacte.nom }} - {{ contacte.telefon }}</li>
  {% endfor %}
</ul>
```

### 1.4.4. Cicles

La funció `cycle` de Twig és molt útil quan volem **alternar valors de manera cíclica** dins d’un bucle.  
S’utilitza habitualment per canviar colors, classes CSS o estils en llistats.

Volem mostrar un llistat amb 10 files alternant dos estils de fila: `parell` i `imparell`.

```twig
{% for i in 1..10 %}
  <div class="{{ cycle(['parell', 'imparell'], i) }}">
    Fila número {{ i }}
  </div>
{% endfor %}
```

Com funciona `cycle`:

- El primer argument és una **llista de valors** que es repetiran.
- El segon argument (`i` en aquest cas) indica la **posició actual del cicle**.

Resultat generat (simplificat)

```html
<div class="parell">Fila número 1</div>
<div class="imparell">Fila número 2</div>
<div class="parell">Fila número 3</div>
...
```

### 1.4.5. Filtres

```twig
{{ contacte.nom | upper }}        {# majúscules #}
{{ contacte.nom | lower }}        {# minúscules #}
{{ "now" | date("d/m/Y") }}       {# data actual #}
{{ contactes | length }}          {# quantitat d’elements #}
```

💡 En Symfony 6.4+ o superior, es recomana usar sempre **filtres Twig** en lloc de manipular cadenes amb PHP.

---

## 1.5. Inclusió de plantilles

En Twig podem inserir altres plantilles dins d’una per **reutilitzar codi** i **millorar l’organització** de les nostres vistes.  

Aquesta tècnica s’anomena **inclusió de plantilles** i és especialment útil per a elements comuns com menús, peus de pàgina o barres laterals.

---

**Exemple pràctic**

Suposem que tenim l’estructura següent dins del directori `templates/`:

```bash
templates/
│
├── base.html.twig
├── index.html.twig
└── partials/
    ├── _menu.html.twig
    └── _footer.html.twig
```

- Desa les plantilles parcials dins d’una carpeta `partials/`.
- Utilitza un guió baix (`_`) al principi del nom per indicar que són plantilles auxiliars: `_menu.html.twig`, `_footer.html.twig`.

---

**Fitxer (`partials/_menu.html.twig`)**

Aquest fitxer conté el menú principal del nostre lloc web.

```twig
<nav>
  <ul>
    <li><a href="">Inici</a></li>
    <li><a href="">Articles</a></li>
    <li><a href="">Contacte</a></li>
  </ul>
</nav>
```

---

**Fitxer (`base.html.twig`)**

Ací definim l’estructura principal de la pàgina i **incloem** el menú i el peu de pàgina amb la directiva `include`.

```twig
<!DOCTYPE html>
<html lang="ca">
  <head>
    <meta charset="UTF-8">
    <title>{% block title %}Aplicació Symfony{% endblock %}</title>
  </head>
  <body>

    {# Incloem el menú com una plantilla parcial #}
    {% include 'partials/_menu.html.twig' %}

    <main>
        {% block contingut %}
          <!-- Contingut específic de cada pàgina -->
        {% endblock %}
    </main>

    {# Incloem el peu de pàgina #}
    {% include 'partials/_footer.html.twig' %}

  </body>
</html>
```

---

**Fitxer (`inici.html.twig`)**

Aquesta plantilla **hereta** de `base.html.twig` i defineix el bloc de contingut:

```twig
{% extends 'base.html.twig' %}

{% block title %}Pàgina principal{% endblock %}

{% block contingut %}
  <h1>Benvinguts a la pàgina principal</h1>
  <p>Aquesta pàgina utilitza el menú i el peu de pàgina amb include.</p>
{% endblock %}
```

---

**Resultat en navegador**

Quan renderitzes `inici.html.twig`, Twig combinarà totes les plantilles:

```html
<nav>
  <ul>
    <li><a href="">Inici</a></li>
    <li><a href="">Articles</a></li>
    <li><a href="">Contacte</a></li>
  </ul>
</nav>

<main>
  <h1>Benvinguts a la pàgina principal</h1>
  <p>Aquesta pàgina utilitza el menú i el peu de pàgina amb include.</p>
</main>

<footer>
  © 2025 IES Lluís Simarro
</footer>
```

---

## 1.6. Enllaçar a altres rutes de l’aplicació

Si el que volem és definir un enllaç a una altra ruta o pàgina de la nostra aplicació, utilitzem la funció `path()` per indicar el **nom** (`name`) assignat a la ruta a la qual volem anar.

Per exemple, si volem anar a la fitxa d’un contacte el codi del qual està emmagatzemat en la variable `codi`, escriurem:

```twig
<a href="{{ path('fitxa_contacte', {'codi': codi}) }}">Veure fitxa</a>
```

Aquesta instrucció genera automàticament l’enllaç segons les rutes definides al controlador.

Si la ruta està declarada així:

```php
#[Route('/contacte/{codi}', name: 'fitxa_contacte')]
```

i la variable `codi` val `12`, el resultat final serà:

```html
<a href="/contacte/12">Veure fitxa</a>
```

Quan la ruta no té paràmetres, simplement podem fer:

```twig
<a href="{{ path('inici') }}">Tornar a l’inici</a>
```

Si necessitem generar una **URL absoluta** (per a correus o enllaços externs), emprarem `url()`:

```twig
<a href="{{ url('fitxa_contacte', {'codi': codi}) }}">Fitxa completa</a>
```

| Funció Twig | Resultat | Exemple | Ús recomanat |
|--------------|-----------|----------|---------------|
| `path('nom_ruta')` | URL **relativa** | `/contacte/12` | Enllaços dins la web |
| `url('nom_ruta')` | URL **absoluta** | `https://exemple.com/contacte/12` | Enllaços externs o correus |

---

## 1.7. Afegir contingut estàtic a les plantilles

En una aplicació Symfony moderna, els recursos estàtics (fulls d’estil, imatges i JavaScript) es desen dins la carpeta `assets/` i es publiquen automàticament a `public/` mitjançant **AssetMapper**.

**Estructura recomanada de carpetes**

```bash
assets/
└── styles/
    └── app.css
└── js/
    └── app.js
└── imgs/
    ├── sony-wh1000xm5.jpg
    └── ...
```

**Instal·lació i comandes útils**

```bash
# Instal.la el component (si no el té)
composer require symfony/asset-mapper

# Mostra la llista d’actius disponibles
php bin/console debug:asset-map

# Genera còpies versionades per a producció
php bin/console asset-map:compile
```

A continuació veurem com incloure aquests **recursos estàtics** a les nostres plantilles.

---

### 1.7.1. Fulls d’estil (CSS)

Crea dins de la carpeta `assets/` una subcarpeta `styles/` i afegeix-hi un fitxer anomenat `estils.css`:

**Ruta del fitxer:**  

```bash
assets/styles/estils.css
```

**Contingut de prova:**

```css
body {
  background-color: #99ccff;
}

h1 {
  border-bottom: 1px solid black;
  background-color: white;
  color: red;
  margin: 10px auto;
  text-align: center;
  width: 50%;
}
```

Per carregar aquest full d’estil en totes les pàgines, edita la plantilla base `base.html.twig` i afegeix la línia següent dins del bloc `stylesheets`:

```twig
{% block stylesheets %}
  <link href="{{ asset('css/estils.css') }}" rel="stylesheet" />
{% endblock %}
```

💡 La funció `asset()` genera automàticament la ruta correcta del fitxer segons la configuració del projecte.

---

### 1.7.2. Imatges

Per mostrar imatges desades a la carpeta `assets/imgs/`, pots fer-ho així:

```twig
<img src="{{ asset('imgs/imatge.png') }}" alt="Exemple d’imatge">
```

Si necessites una **URL absoluta** (per exemple, per enviar-la en un correu electrònic o per compartir-la fora de la web), pots combinar `absolute_url()` amb `asset()`:

```twig
<img src="{{ absolute_url(asset('imgs/imatge.png')) }}" alt="Ex. d’imatge">
```

---

### 1.7.3. Fitxers JavaScript

Els fitxers `.js` s’acostumen a incloure al final del `body` o dins del bloc `javascripts` de la plantilla base.

Per exemple, si tenim un fitxer `assets/js/llibreria.js`, el carregaríem així:

```twig
{% block javascripts %}
  <script src="{{ asset('js/llibreria.js') }}" defer></script>
{% endblock %}
```

💡 L'opció `defer` fa que el JS s'execute després de carregar l'HTML, millorant el rendiment.

---

### 1.7.4. Bones pràctiques

- Guarda tots els recursos estàtics dins `assets/` i organitza’ls per carpetes (`styles/`, `js/`, `imgs/`...).
- La carpeta `public/` queda per a arxius **realment públics** que no passen per AssetMapper (per exemple `robots.txt`, `favicon.ico` o fitxers pujats per l’usuari).  
- Fes servir sempre `asset()` o `absolute_url()` en lloc de rutes relatives manuals.
- Si vols enllaçar una **URL externa** (p. ex. `https://cdn.jsdelivr.net/...`), posa-la directament al `src` o `href`, sense `asset()`.

---

## 1.8. Resum

| Eina Twig | Serveix per a... | Exemple pràctic | Símil programació |
|------------|------------------|-----------------|-------------------|
| `{% extends %}` | Heretar una plantilla completa | `inici.html.twig` hereta de `base.html.twig` | Classe filla ↔ classe pare |
| `{% include %}` | Afegir una peça o component comú | Inserir un menú o un peu de pàgina | Copiar/enganxar un mòdul |
| `{% block %}` | Definir zones modificables dins la base | `body`, `title`, etc. | Mètode que pots sobreescriure |

Funcionament:

- `extends` defineix **qui hereta**.
- `include` afegeix **peces reutilitzables** dins de l’estructura.
- `block` marca **què pot canviar**.

---

## 1.9. Bones pràctiques amb Twig

- Mantindre les vistes lliures de lògica de negoci.  
- Utilitzar blocs, includes i herència per evitar repeticions.  
- Passar només les dades necessàries des del controlador.  
- Organitzar les plantilles en carpetes (`templates/contacte/`, `templates/partials/`...).  
- Usar noms coherents i semàntics per a les variables.

---

## 2. Actualitzacions i parts obsoletes

| Tema original | Estat actual | Recomanació |
|----------------|---------------|-------------|
| `renderView()` i `Response` manual | Encara vàlid però poc usat | Fes servir `$this->render()` |
| Injecció de dependències en vistes (accés directe a `app.*`) | Desaconsellat | Passa les dades explícitament |
| Variables globals de Twig (`app.request`, `app.user`) | Mantingudes | Només per a casos específics |
| `{{ asset() }}` per recursos | Vigent però millorable | Usa **AssetMapper** si el projecte està en Symfony 6.4+ |

---

## 3. Exemple complet

### 3.1. Estructura de fitxers

```bash
src/
└── Controller/
    └── ContacteController.php

templates/
├── base.html.twig
└── contacte/
    ├── fitxa.html.twig
    └── cerca.html.twig
```

---

### 3.2. Controlador

**Fitxer:** `src/Controller/ContacteController.php`

```php
<?php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ContacteController extends AbstractController
{
    private array $contactes = [
        ["codi" => 1, "nom" => "Salvador Sala",
         "telefon" => "638961244", "email" => "salvasala@simarro.org"],
        ["codi" => 2, "nom" => "Anna Llopis",
         "telefon" => "669332004", "email" => "annallopis@simarro.org"],
        ["codi" => 3, "nom" => "Marc Sanchis",
         "telefon" => "962286040", "email" => "marcsanchis@simarro.org"],
        ["codi" => 4, "nom" => "Laura Palop",
         "telefon" => "663568890", "email" => "laurapalop@simarro.org"],
    ];

    // Métode 1
    #[Route('/contacte/{codi}', name: 'fitxa_contacte', requirements: ['codi' => '\\d+'])]
    public function fitxa(int $codi): Response
    {
        // 1) Buscar el contacte pel codi
        $resultat = array_filter($this->contactes, fn($c) => $c['codi'] === $codi);

        if (!$resultat) {
            // En un cas real, podríem llançar una 404 o mostrar una vista d’error
            return $this->render('contacte/fitxa.html.twig', [
                'contacte' => null,
                'codi' => $codi,
            ]);
        }

        $contacte = array_shift($resultat);

        // 2) Passar dades a la plantilla Twig
        return $this->render('contacte/fitxa.html.twig', [
            'contacte' => $contacte,
            'codi' => $codi,
        ]);
    }

    // Métode 2
    #[Route('/contacte/{text}', name: 'buscar_contacte', requirements: ['text' => '[a-zA-Z]+'])]
    public function buscar(string $text): Response
    {
        // 1) Filtrar per nom (coincidència parcial, sense tindre en compte maj./min.)
        $resultat = array_filter($this->contactes, function ($c) use ($text) {
            return stripos($c['nom'], $text) !== false;
        });

        // 2) Passar la llista (pot ser buida) a la vista
        return $this->render('contacte/cerca.html.twig', [
            'text' => $text,
            'resultats' => $resultat,
        ]);
    }
}
```

**Què hem canviat respecte a Tema 2?**  
Ara no construïm HTML al controlador: sols preparem dades i les enviem a Twig per a pintar la resposta (exactament el que recomanàvem a la secció *“Separació MVC real”* del tema 2).

---

### 3.3. Plantilla base

**Fitxer:** `templates/base.html.twig`

```html
<!DOCTYPE html>
<html lang="ca">
  <head>
    <meta charset="UTF-8">
    <title>{% block title %}Contactes{% endblock %}</title>
  </head>
  <body>
    <header>
      <h1>👇 Aplicació de contactes</h1>
      <p class="meta">Exemple MVC amb Twig</p>
    </header>

    <main>
      {% block body %}{% endblock %}
    </main>

    <footer>
      <p>&copy; {{ "now"|date("Y") }} IES Lluís Simarro</p>
    </footer>
  </body>
</html>
```

---

### 3.4. Vista de fitxa

**Fitxer:** `templates/contacte/fitxa.html.twig`

```twig
{% extends 'base.html.twig' %}

{% block title %}Fitxa de contacte{% endblock %}

{% block body %}
  <h2>Fitxa de contacte</h2>

  {% if contacte %}
    <ul class="llista">
      <li><strong>Nom:</strong> {{ contacte.nom }}</li>
      <li><strong>Telèfon:</strong> {{ contacte.telefon }}</li>
      <li><strong>Email:</strong> {{ contacte.email }}</li>
      <li class="meta">Codi intern: {{ codi }}</li>
    </ul>
  {% else %}
    <p>Contacte amb codi {{ codi }} no trobat.</p>
  {% endif %}
{% endblock %}
```

---

### 3.5. Vista de cerca

**Fitxer:** `templates/contacte/cerca.html.twig`

```twig
{% extends 'base.html.twig' %}

{% block title %}Resultats per "{{ text }}"{% endblock %}

{% block body %}
  <h2>Resultats de la cerca: “{{ text }}”</h2>

  {% if resultats is not empty %}
    {% for c in resultats %}
      <ul class="llista">
        <li><strong>Nom:</strong> {{ c.nom }}</li>
        <li><strong>Telèfon:</strong> {{ c.telefon }}</li>
        <li><strong>Email:</strong> {{ c.email }}</li>
      </ul>
      <hr>
    {% endfor %}
  {% else %}
    <p>No s’han trobat contactes amb “{{ text }}”.</p>
  {% endif %}
{% endblock %}
```

---

### 3.6. Proves ràpides

**Fitxa (numèric):**  
👉 `http://localhost/contacte/3` → mostra la fitxa amb codi 3.

**Cerca (text):**  
👉 `http://localhost/contacte/anna` → llista coincidències per “anna”.

> Ambdós camins comparteixen prefix i es diferencien pel *requirement* de la ruta (igual que l’exemple original del Tema 2).

---

### 3.7. Notes i bones pràctiques

- Mantín l’HTML fora del controlador; la vista és responsabilitat de Twig.  
- Centralitza estils i estructura en `base.html.twig` i fes servir herència (`{% extends %}`) i blocs (`{% block %}`).  
- Si reuses fragments (per ex. el llistat d’un contacte), pots crear un *partial*:  
`templates/contacte/_targeta.html.twig` i incloure’l amb `{% include %}`.

---

## 4. Recursos recomanats

- [Documentació oficial Twig 3.x](https://twig.symfony.com/doc/3.x/)  
- [Plantilles en Symfony 6.4+](https://symfony.com/doc/current/templates.html)  
- [Symfony AssetMapper (nou sistema d’actius)](https://symfony.com/doc/current/frontend/asset_mapper.html)

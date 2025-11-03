---
title: "TEMA 3 — Plantilles amb Twig"
nav:
  - TEMA 3: 'tema3.md'
---

# 🧩 TEMA 3 — Plantilles amb Twig

## 1. Definint plantilles de vistes amb Twig

Els controladors que hem vist fins ara generaven HTML directament en el codi PHP. Per a separar el disseny de la lògica, Symfony utilitza **Twig**, un motor de plantilles que permet treballar amb fitxers `.html.twig` dins la carpeta `templates/`.

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

Podem passar informació del controlador a la plantilla mitjançant un **array associatiu**:

**Controlador**

```php
<?php

#[Route('/contacte/{codi}', name: 'fitxa_contacte')]
public function fitxa($codi): Response
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
      {% block body %}{% endblock %}
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

{% block body %}
  <h2>Benvingut a la nostra aplicació</h2>
{% endblock %}
```

---

## 1.4. Comentaris, filtres i estructures de control

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

```
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
    <li><a href="{{ path('inici') }}">Inici</a></li>
    <li><a href="{{ path('articles') }}">Articles</a></li>
    <li><a href="{{ path('contacte') }}">Contacte</a></li>
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
    <title>{% block title %}Benvinguts!{% endblock %}</title>
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

**Fitxer (`index.html.twig`)**

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

Quan renderitzes `index.html.twig`, Twig combinarà totes les plantilles:

```html
<nav>
  <ul>
    <li><a href="/inici">Inici</a></li>
    <li><a href="/articles">Articles</a></li>
    <li><a href="/contacte">Contacte</a></li>
  </ul>
</nav>

<main>
  <h1>Benvinguts a la pàgina principal</h1>
  <p>Aquesta pàgina utilitza el menú i el peu de pàgina amb include.</p>
</main>

<footer>
  © 2025 - El meu lloc web
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
## 1.7. Resum

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

## 1.8. Bones pràctiques amb Twig

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

## 3. Recursos recomanats

- [Documentació oficial Twig 3.x](https://twig.symfony.com/doc/3.x/)  
- [Plantilles en Symfony 6.4+](https://symfony.com/doc/current/templates.html)  
- [Symfony AssetMapper (nou sistema d’actius)](https://symfony.com/doc/current/frontend/asset_mapper.html)


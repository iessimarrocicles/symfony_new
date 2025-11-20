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
   ```env
   DATABASE_URL="mysql://root@127.0.0.1:3306/daw_tenda"
   ```
2. Crea la base de dades:
   ```bash
   php bin/console doctrine:database:create
   ```
3. Genera la nova entitat `Seccio`:
   ```bash
   php bin/console make:entity
   > Seccio
   ```
4. Afegeix els següents camps:
    - `nom` (string, 50)
    - `descripcio` (string, 255)
    - `any` (integer)
    - `imatge` (string, 100)

5. Genera la migració i aplica-la:
   ```bash
   php bin/console make:migration
   php bin/console doctrine:migration:migrate
   ```

---

## Exercici 2

**Inserció de seccions des del Controlador**

1. Afegeix un nou mètode `afegir` al controlador `SeccioController` per inserir totes les seccions:
   ```php
   <?php

    public function __construct(private EntityManagerInterface $gestor ) {

    }

    #[Route('/seccions/afegir', name: 'afegir_seccions')]
    public function afegirSeccions(): Response 
    {
        $seccio1 = new Seccio();
        $seccio1->setNom("Roba");
        $seccio1->setDescripcio("Secció de roba");
        $seccio1->setAny(2026);
        $seccio1->setImatge("roba.webp");

        $seccio2 = new Seccio();
        $seccio2->setNom("Calçat");
        $seccio2->setDescripcio("Secció de sabates i esportives");
        $seccio2->setAny(2025);
        $seccio2->setImatge("calçat.webp");

        $seccio3 = new Seccio();
        $seccio3->setNom("Complements");
        $seccio3->setDescripcio("Secció de complements de moda");
        $seccio3->setAny(2023);
        $seccio3->setImatge("complements.webp");

        $seccio4 = new Seccio();
        $seccio4->setNom("Tecnologia");
        $seccio4->setDescripcio("Secció d'electrònica i gadgets");
        $seccio4->setAny(2027);
        $seccio4->setImatge("tecnologia.webp");

        try {
            $this->gestor->persist($seccio1);
            $this->gestor->persist($seccio2);
            $this->gestor->persist($seccio3);
            $this->gestor->persist($seccio4);

            $this->gestor->flush();
            return $this->render('seccio/afegir.html.twig');
        } catch (\Exception $e) {
            return $this->render('seccio/error.html.twig', ['error' => $e->getMessage()]);
        }
    }

   ?>
   ```

2. Crea la plantilla `templates/equip/afegir.html.twig` per mostrar el resultat:
   ```twig
   {% extends 'base.html.twig' %}

   {% block title %}Tenda Simarro{% endblock %}

   {% block contingut %}
   <section class="seccio-inici">
      
      <h2>Seccions afegides correctament!</h2>
      <br><br>
      <a href="{{ path('llistat_seccions') }}" class="btn btn-outline-primary btn-sm">Tornar a seccions</a>

   </section>
   {% endblock %}
   ```

3. Crea la plantilla `templates/equip/error.html.twig` per mostrar el resultat:
   ```twig
   {% extends 'base.html.twig' %}

   {% block title %}Tenda Simarro{% endblock %}

   {% block contingut %}
   <section class="seccio-inici">
      
      <h2>Error afegint noves seccions!</h2>
      <p class="text-danger">S'ha produït l'error següent: {{ error }}</p>
      <br><br>
      <a href="{{ path('llistat_seccions') }}" class="btn btn-outline-primary btn-sm">Tornar a seccions</a>
      
   </section>
   {% endblock %}
   ```

4. Afegeix un enllaç en el menú de la plantilla principal `_menu.html.twig` per accedir a `/seccions/afegir`.
   ```twig
   <li class="nav-item">
      <a class="btn btn-sm btn-primary ms-lg-2 mt-2 mt-lg-0" href="{{ path('afegir_seccions') }}">Afegir seccions</a>
   </li>
   ```

---

## Exercici 3

**Entitat `Article` amb Relació ManyToOne a `Seccio`**

Crear una nova entitat `Article` relacionada amb `Seccio` (Molts a Un).

1. Genera l’entitat:
   ```bash
   php bin/console make:entity
   > Article
   ```
2. Afegeix els camps:
    - `nom` (string, 100)
    - `preu` (float)
    - `stock` (integer)
    - `imatge` (string, 255)
    - `seccio` (relació ManyToOne amb Seccio)
        - Quan l’assistent ens pregunta si volem afegir una propietat en la classe Seccio per poder accedir als seus articles des del codi. En este cas, acceptem la proposta escrivint yes i definim el nom de la nova propietat com articles. Això ens fara una relació bidireccional. D’esta manera es genera automàticament una relació bidireccional: la classe Article tindrà la propietat seccio, i la classe Seccio disposarà d’una col·lecció articles amb els mètodes getArticles(), addArticle() i removeArticle().

3. Genera i executa la migració:
   ```bash
   php bin/console make:migration
   php bin/console doctrine:migration:migrate
   ```

---

## Exercici 4

**Inserció d’un article amb Relació**

1. Afegeix un nou controlador `ArticleController.php` per inserir articles:
   ```php
   <?php

   namespace App\Controller;

   use App\Entity\Article;
   use App\Entity\Seccio;

   use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
   use Symfony\Component\HttpFoundation\Response;
   use Symfony\Component\Routing\Annotation\Route;
   use Doctrine\ORM\EntityManagerInterface;


   class ArticleController extends AbstractController
   {

      private $repositoriSeccio;

      public function __construct(private EntityManagerInterface $gestor ) {
         $this->repositoriSeccio = $this->gestor->getRepository(Seccio::class);
      }

      #[Route('/articles/afegirUn', name: 'afegirUn_article')]
      public function afegirUnArticle(): Response {

         $seccio = $this->repositoriSeccio->find(1); // Secció existent


         $article = new Article();
         $article->setNom("Pantalons");
         $article->setPreu(86);
         $article->setStock(random_int(10, 100));
         $article->setImatge("https://placehold.co/300x200?text=");
         $article->setSeccio($seccio);

         $gestor->persist($article);
         $gestor->flush();

         return $this->render('article/afegir.html.twig', ['article' => $article]);
      }

      #[Route('/articles/afegir', name: 'afegir_articles')]
      public function afegirArticles(): Response {

         $seccions = $this->repositoriSeccio->findAll();

         foreach ($seccions as $seccio) {

               // Determinem els articles segons el nom de la secció
               $articlesAInsertar = [];

               switch (strtolower($seccio->getNom())) {

                  case 'roba':
                     $articlesAInsertar = ["Pantalons", "Camisa", "Jersei", "Jaqueta"];
                     break;

                  case 'calçat':
                     $articlesAInsertar = ["Sabates", "Botes", "Sandàlies", "Esportives"];
                     break;

                  case 'complements':
                     $articlesAInsertar = ["Cinturó", "Bufanda", "Ulleres", "Motxilla"];
                     break;

                  case 'tecnologia':
                     $articlesAInsertar = ["Auriculars", "Rellotge", "Teclat", "Ratolí"];
                     break;

               }

               // Generem els articles associats
               foreach ($articlesAInsertar as $nomArticle) {
                  $article = new Article();
                  $article->setNom($nomArticle);
                  $article->setPreu(random_int(10, 200));
                  $article->setStock(random_int(10, 100));
                  $article->setImatge("https://placehold.co/300x200?text=$nomArticle");
                  $article->setSeccio($seccio);

                  $this->gestor->persist($article);
               }
         }


         try {
         // Guardem tot al final

               $this->gestor->flush();
               return $this->render('seccio/afegir.html.twig');
         } catch (\Exception $e) {
               return $this->render('seccio/error.html.twig', ['error' => $e->getMessage()]);
         }
      }

   }
   ```

2. Crea la plantilla `templates/article/afegir.html.twig` per mostrar el resultat:
   ```twig
   {% extends 'base.html.twig' %}

   {% block title %}Tenda Simarro{% endblock %}

   {% block contingut %}
   <section class="seccio-inici">
      
      <h2>Articles afegits correctament!</h2>
      <br><br>
      <a href="{{ path('llistat_seccions') }}" class="btn btn-outline-primary btn-sm">Tornar a seccions</a>

   </section>
   {% endblock %}
   ```

3. Crea la plantilla `templates/equip/error.html.twig` per mostrar el resultat:
   ```twig
   {% extends 'base.html.twig' %}

   {% block title %}Tenda Simarro{% endblock %}

   {% block contingut %}
   <section class="seccio-inici">
      
      <h2>Error afegint nous articles!</h2>
      <p class="text-danger">S'ha produït l'error següent: {{ error }}</p>
      <br><br>
      <a href="{{ path('llistat_seccions') }}" class="btn btn-outline-primary btn-sm">Tornar a seccions</a>
      
   </section>
   {% endblock %}
   ```

4. Afegeix un enllaç en el menú de la plantilla principal `_menu.html.twig` per accedir a `/articles/afegir`.
   ```twig
   <li class="nav-item">
      <a class="btn btn-sm btn-primary ms-lg-2 mt-2 mt-lg-0" href="{{ path('afegir_articles') }}">Afegir articles</a>
   </li>
   ```

---

## Exercici 5

**Modificar les vistes perquè utilitzen dades reals de la base de dades**

1. Actualitza totes les vistes i controladors que encara utilitzen el servei antic DadesSeccioServei.php (creat en el tema anterior).

2. Substitueix les crides al servei per consultes reals a Doctrine mitjançant el repositori de Seccio i Article.

3. Revisa totes les plantilles Twig per assegurar que:
    - Les dades que es mostren provenen dels objectes Seccio i Article recuperats amb Doctrine.
    - Ja no s'utilitzen arrays ni llistes simulades.

**Fitxer:** `SeccioController.php`

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

use App\Entity\Seccio;
use Doctrine\ORM\EntityManagerInterface;


class SeccioController extends AbstractController
{

    private $repositoriSeccio;

    public function __construct(private EntityManagerInterface $gestor ) {
        $this->repositoriSeccio = $this->gestor->getRepository(Seccio::class);
    }

    /**
     * Llistat de totes les seccions
     */
    #[Route('/seccions', name: 'llistat_seccions', methods: ['GET'])]
    public function llistarSeccions(): Response
    {
        // Pasamos los datos a Twig
        return $this->render('seccio/llistat.html.twig', [
            'seccions' => $this->repositoriSeccio->findAll()
        ]);
    }

    /**
     * Detall d'una secció concreta (ja el tenies, el mantinc i l'adapte)
     */
    #[Route('/seccio/{codi}', name: 'dades_seccio', requirements: ['codi' => '\d+'], methods: ['GET'])]
    public function voreSeccio(int $codi): Response
    {

        $seccio = $this->repositoriSeccio->find($codi);

        if ($seccio){
            return $this->render('seccio/detall.html.twig', ['seccio' => $seccio]);
        }else{
            return new Response("No s'ha trobat la secció: $codi", Response::HTTP_NOT_FOUND);
        }
    }

    #[Route('/seccions/afegir', name: 'afegir_seccions')]
    public function afegirSeccions(): Response 
    {
        $seccio1 = new Seccio();
        $seccio1->setNom("Roba");
        $seccio1->setDescripcio("Secció de roba");
        $seccio1->setAny(2026);
        $seccio1->setImatge("roba.webp");

        $seccio2 = new Seccio();
        $seccio2->setNom("Calçat");
        $seccio2->setDescripcio("Secció de sabates i esportives");
        $seccio2->setAny(2025);
        $seccio2->setImatge("calçat.webp");

        $seccio3 = new Seccio();
        $seccio3->setNom("Complements");
        $seccio3->setDescripcio("Secció de complements de moda");
        $seccio3->setAny(2023);
        $seccio3->setImatge("complements.webp");

        $seccio4 = new Seccio();
        $seccio4->setNom("Tecnologia");
        $seccio4->setDescripcio("Secció d'electrònica i gadgets");
        $seccio4->setAny(2027);
        $seccio4->setImatge("tecnologia.webp");

        try {
            $this->gestor->persist($seccio1);
            $this->gestor->persist($seccio2);
            $this->gestor->persist($seccio3);
            $this->gestor->persist($seccio4);

            $this->gestor->flush();
            return $this->render('seccio/afegir.html.twig');
        } catch (\Exception $e) {
            return $this->render('seccio/error.html.twig', ['error' => $e->getMessage()]);
        }
    }
}

?>
```

**Fitxer:** `_menu.html.twig`

```twig
{# ==========================
   CAPÇALERA + MENÚ PRINCIPAL
   ========================== #}
<header>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark" aria-label="Menú principal">
        <div class="container">
            <a class="navbar-brand fw-bold" href="{{ path('inici') }}">Tenda</a>

            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#menuPrincipal" aria-controls="menuPrincipal" aria-expanded="false" aria-label="Canviar navegació">
                <span class="navbar-toggler-icon"></span>
            </button>

            <div class="collapse navbar-collapse" id="menuPrincipal">
                <ul class="navbar-nav me-auto mb-2 mb-lg-0">

                    {% if seccions is defined and seccions|length > 0 %}
                        {% for seccio in seccions %}
                            <li class="nav-item">
                                <a class="nav-link" href="{{ path('dades_seccio', { codi: seccio.id }) }}">
                                    {{ seccio.nom }}
                                </a>
                            </li>
                        {% endfor %}
                    {% else %}
                        {# Si no passes 'seccions', mostrem uns enllaços d’exemple #}
                        <li class="nav-item"><a class="nav-link" href="{{ path('llistat_seccions') }}">Seccions</a></li>
                    {% endif %}
                    
                </ul>

                <ul class="navbar-nav ms-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="">🛒 Carret
                            {# Opcional: mostra el nombre d’articles en el carret si 'quantitatCarret' està definit #}
                            {% if quantitatCarret is defined and quantitatCarret > 0 %}
                                <span class="badge text-bg-success ms-1">{{ quantitatCarret }}</span>
                            {% endif %}
                        </a>
                    </li>

                    <li class="nav-item">
                        <a class="btn btn-sm btn-primary ms-lg-2 mt-2 mt-lg-0" href="{{ path('afegir_seccions') }}">Afegir seccions</a>
                    </li>
                    <li class="nav-item">
                        <a class="btn btn-sm btn-primary ms-lg-2 mt-2 mt-lg-0" href="{{ path('afegir_articles') }}">Afegir articles</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</header>
```

**Fitxer:** `llistat.html.twig`

```twig
{% extends 'base.html.twig' %}

{% block title %}Llistat de seccions{% endblock %}

{% block contingut %}
  <section class="container" style="max-width: 900px; margin: 2rem auto;">
    <h1 style="margin-bottom: 1rem;">Llistat de seccions</h1>

    {% if seccions is empty %}
      <p>No hi ha seccions disponibles.</p>
    {% else %}
      <table style="width:100%; border-collapse: collapse;">
        <thead>
          <tr>
            <th style="text-align:left; border-bottom:1px solid #ccc; padding: .5rem;">Id</th>
            <th style="text-align:left; border-bottom:1px solid #ccc; padding: .5rem;">Nom</th>
            <th style="text-align:left; border-bottom:1px solid #ccc; padding: .5rem;">Descripció</th>
            <th style="text-align:left; border-bottom:1px solid #ccc; padding: .5rem;">Any</th>
            <th style="text-align:left; border-bottom:1px solid #ccc; padding: .5rem;">Articles</th>
            <th style="text-align:left; border-bottom:1px solid #ccc; padding: .5rem;">Accions</th>
          </tr>
        </thead>
        <tbody>
          {% for seccion in seccions %}
            <tr>
              <td style="padding: .5rem;">{{ seccion.id }}</td>
              <td style="padding: .5rem;">{{ seccion.nom }}</td>
              <td style="padding: .5rem;">{{ seccion.descripcio }}</td>
              <td style="padding: .5rem;">{{ seccion.any }}</td>
              <td style="padding: .5rem;">
                {% if seccion.articles is iterable %}
                  {{ seccion.articles|length }} ítems
                {% else %}
                  —
                {% endif %}
              </td>
              <td style="padding: .5rem;">
                <a href="{{ path('dades_seccio', { codi: seccion.id }) }}" class="btn btn-outline-primary btn-sm">Veure</a>
              </td>
            </tr>
          {% endfor %}
        </tbody>
      </table>
    {% endif %}
  </section>
{% endblock %}
```

**Fitxer:** `detall.html.twig`

```twig
{% extends 'base.html.twig' %}

{% block title %}{{ seccio.nom }} — Tenda{% endblock %}

{% block contingut %}
<header class="d-flex align-items-center justify-content-between mb-4">
    {# PART ESQUERRA: IMATGE + TEXT #}
    <div class="d-flex align-items-center gap-4">

      {# IMATGE DE LA SECCIÓ #}
      <img
        src="{{ asset('imgs/seccio/' ~ seccio.imatge) }}"
        alt="Imatge de {{ seccio.nom }}"
        class="rounded-circle shadow-sm object-fit-cover seccio-foto"
        width="120" height="120"
        loading="lazy" decoding="async">

      {# TEXT DESCRIPTIU #}
      <div>
          <h1 class="mb-1">{{ seccio.nom }}</h1>
          <p class="text-muted mb-0">{{ seccio.descripcio }}</p>
          <small class="text-secondary">Any {{ seccio.any }}</small>
      </div>

    </div>

    {# PART DRETA: BOTÓ TORNAR A L’INICI #}
    <a href="{{ path('llistat_seccions') }}" class="d-inline-flex align-items-center">
        <img
          src="{{ asset('imgs/home.webp') }}"
          alt="Inici"
          width="48" height="48"
          class="rounded-circle shadow-sm"
          loading="lazy" decoding="async">
    </a>
</header>


  <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 g-4">
    {% for article in seccio.articles %}
      <div class="col">
        <div class="card h-100 article-card">
          {# En un futur pots posar imatge per a cada article; ara utilitzem placehold #}
          <img
            class="card-img-top"
            src="{{ 'https://placehold.co/300x200?text=' ~ article.nom }}"
            alt="{{ article.nom }}"
            loading="lazy" decoding="async">

          <div class="card-body">
            <h5 class="card-title">{{ article.nom }}</h5>
            <p class="card-text text-muted">Disponible en diferents talles i colors.</p>
            <a href="#" class="btn btn-outline-primary btn-sm">Veure</a>
          </div>
        </div>
      </div>
    {% else %}
      <p class="text-muted">Encara no hi ha articles en aquesta secció.</p>
    {% endfor %}
  </div>
{% endblock %}
```

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

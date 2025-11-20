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

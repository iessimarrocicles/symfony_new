---
title: "TEMA 2 — El patró MVC a Symfony"
nav:
  - TEMA 2: 'tema2.md'
---

# 🧩 TEMA 2 — El patró MVC a Symfony

## 1. Què és el patró MVC?

**MVC** són les sigles de **Model - Vista - Controlador**, i és el patró d’arquitectura més utilitzat en el desenvolupament web.

L’objectiu és **separar les responsabilitats** del codi per a aconseguir aplicacions més netes, modulars i fàcils de mantenir.

| Component | Descripció | Exemple |
|------------|-------------|----------|
| **Model** | Gestiona les dades i la lògica de negoci. Representa la informació que utilitza l’aplicació. | Entitats de Doctrine com `Contacte`, `Usuari`, `Llibre`... |
| **Vista** | S’encarrega de mostrar la informació a l’usuari (interfície HTML). | Fitxers `.html.twig` dins la carpeta `templates/` |
| **Controlador** | Gestiona les peticions de l’usuari, interactua amb el model i tria quina vista mostrar. | Classes dins `src/Controller/` |

### 1.1. Avantatges del patró MVC
- Codi **més estructurat i mantenible**.  
- Facilita el **treball en equip** (programador ↔ dissenyador).  
- Permet **reutilitzar** components i millorar la seguretat.  

---

## 2. Controladors i rutes

Un **controlador** és una funció o mètode PHP que rep la petició de l’usuari, la processa i envia una resposta (una pàgina HTML, JSON, etc.).

Els controladors:

- Es guarden dins la carpeta `src/Controller/` del projecte.
- Normalment estan dins de **classes**, i aquestes solen tenir el sufix `Controller` (com `IniciController`).
- Cada mètode de la classe gestiona una **ruta** concreta.

Exemple:  

- Un controlador anomenat `IniciController` pot tindre un mètode `inici()` que mostre un missatge de benvinguda quan l’usuari accedeix a la pàgina principal del web.

Per a crear una pàgina en Symfony calen **dos elements principals**:

1. **Una ruta** → defineix l’URL que activarà el controlador.  
2. **Un controlador** → és la classe que conté el codi que es vol executar.

---

### 2.1. Rutes tradicionals vs rutes amigables

- **Rutes tradicionals**: Són les que utilitzaven els antics projectes PHP, on els paràmetres s’enviaven per l’URL amb `?`.  
  Exemple: `http://localhost/contacte.php?codi=3`  
  👉 Accedeixen directament a un fitxer concret.

- **Rutes amigables**: Són les que usa Symfony. Les dades es passen dins de la pròpia URL, sense extensions ni paràmetres visibles.  
  Exemple: `http://localhost/contacte/3`  
  👉 Són més netes, segures i fàcils de recordar.

En Symfony **totes les rutes són amigables**, ja que passen pel sistema de ruteig intern i no accedeixen directament als fitxers.

---

### 2.2. Exemple bàsic

Fitxer: `src/Controller/IniciController.php`

```php
<?php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class IniciController
{
    #[Route('/', name: 'inici')]
    public function inici(): Response
    {
        return new Response('Benvingut/da al web de contactes');
    }
}
?>
```

**Explicació:**

- `namespace App\Controller;` defineix l'espai de noms dins la carpeta `src`.
- `#[Route('/', name: 'inici')]` defineix la ruta `/` i el seu nom intern.
- El mètode `inici()` retorna un objecte `Response`, que Symfony enviarà al navegador.

> 🔸 A partir de Symfony 6, les rutes es defineixen amb **atributs `#[Route]`** (no amb anotacions `@Route`, que ja és obsolet).

---

### 2.3. Comprovar les rutes de la nostra aplicació

Utilitzant la consola de Symfony (arxiu `bin/console` del nostre projecte) podem comprovar quines rutes hi ha actualment definides en la nostra aplicació, mitjançant aquesta instrucció:

```bash
$ php bin/console debug:router
```

Aquesta ordre mostrarà el llistat de rutes, indicant per a cadascuna:

- El **nom de la ruta**
- El **camí (path)** associat
- El **controlador i mètode** que s’executarà

A més de les rutes que hàgem definit nosaltres (per exemple, la nostra ruta arrel `/`), apareixeran altres rutes creades automàticament per Symfony per a funcions internes, com les del *profiler*, utilitzades per a rastrejar i obtindre detalls de les peticions realitzades durant la depuració.

> ℹ️ En aquest curs no entrarem en detall sobre aquestes rutes internes, ja que s’utilitzen principalment per a tasques de test i depuració avançada.

---

## 3. Rutes amb paràmetres

Podem afegir valors dinàmics dins la ruta perquè el controlador els reba com a paràmetres:

```php
<?php
...

#[Route('/contacte/{codi}', name: 'fitxa_contacte')]
public function fitxa(int $codi): Response
{
    return new Response("Has consultat el contacte amb codi $codi");
}
...
?>
```

Quan accedim a `/contacte/3` el navegador mostrarà:
```
Has consultat el contacte amb codi 3
```

---

### 3.1. Rutes amb paràmetres amb requisits

En Symfony, podem **imposar restriccions** als valors dels paràmetres de la ruta utilitzant **expressions regulars**.  


Això es fa amb l’opció `requirements:` dins de l’atribut `#[Route]`.

```php
<?php

#[Route('/producte/{id}', name: 'fitxa_producte', requirements: ['id' => '\d+'])]
public function fitxa(int $id): Response
{
    return new Response("Producte amb ID $id");
}

?>
```

🧠 **Explicació:**

- El paràmetre `{id}` només serà vàlid si és un **número** (`\d+` indica un o més dígits).  
- Si l’usuari escriu `/producte/abc`, Symfony mostrarà un **error 404**, ja que no compleix el requisit.  
- Es poden definir diversos requisits per a diferents paràmetres.

Altres exemples de requisits:

| Requisit | Significat |
|-----------|-------------|
| `[a-zA-Z]+` | Només lletres |
| `[0-9]{4}` | Exactament 4 dígits |
| `[A-Z]{3}\d{2}` | 3 majúscules i 2 números (p. ex. codis) |

---

### 3.2. Rutes amb paràmetres per defecte

Podem indicar **valors per defecte** per als paràmetres d’una ruta amb l’opció `defaults:`.  
Això permet que la ruta siga vàlida encara que no es passe cap valor.

```php
<?php

#[Route('/blog/{pagina}', name: 'llista_blog', defaults: ['pagina' => 1])]
public function llista(int $pagina): Response
{
    return new Response("Estàs a la pàgina $pagina del blog");
}

?>
```

🧠 **Explicació:**

- Si accedim a `/blog/3`, mostrarà “Estàs a la pàgina 3 del blog”.  
- Si accedim a `/blog/`, Symfony assignarà automàticament `pagina = 1`.  
- Els valors per defecte eviten errors i fan les rutes més flexibles.

També es poden combinar **defaults** i **requirements**:

```php
<?php

#[Route(
    '/blog/{pagina}',
    name: 'llista_blog',
    defaults: ['pagina' => 1],
    requirements: ['pagina' => '\d+']
)]

?>
```

🧩 Així assegurem que el paràmetre `pagina` siga un número i, si no es proporciona, prenga el valor `1` per defecte.

---

## 4. Exemple del controlador

A continuació veurem un exemple complet d’un controlador amb **dues rutes amb el mateix patró** (`/contacte/{valor}`), però diferenciades per **tipus de paràmetre**:

- Si el paràmetre és **numèric**, es mostrarà la *fitxa* del contacte amb eixe codi.  
- Si el paràmetre conté **text**, es farà una *cerca* de contactes pel nom.

Abans d’utilitzar una base de dades, simularem les dades amb un **array local** i mostrarem el contacte amb el codi indicat.

Fitxer: `src/Controller/ContacteController.php`

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ContacteController
{
    // Llista de contactes de mostra
    private $contactes = [
        ["codi" => 1, "nom" => "Salvador Sala", 
         "telefon" => "638961244", "email" => "salvasala@simarro.org"],
        ["codi" => 2, "nom" => "Anna Llopis", 
         "telefon" => "669332004", "email" => "annallopis@simarro.org"],
        ["codi" => 3, "nom" => "Marc Sanchis", 
         "telefon" => "962286040", "email" => "marcsanchis@simarro.org"],
        ["codi" => 4, "nom" => "Laura Palop", 
         "telefon" => "663568890", "email" => "laurapalop@simarro.org"],
    ];

    // --- Mètode 1: Mostrar la fitxa d’un contacte pel seu codi ---
    // Ruta activa quan el paràmetre és numèric (\\d+)
    #[Route('/contacte/{codi}', name:'fitxa_contacte', requirements: ['codi' => '\\d+'])]
    public function fitxa(int $codi): Response
    {
        $resultat = array_filter($this->contactes, 
            function($contacte) use ($codi){
                return $contacte['codi'] == $codi;
            }
        );
        
        if (!$resultat) 
            return new Response('Contacte no trobat');

        // Torna 1r element
        $c = array_shift($resultat);
        $resp = "<ul>".
                    "<li>{ $c['nom'] }</li>".
                    "<li>{ $c['telefon'] }</li>".
                    "<li>{ $c['email'] }</li>".
                "</ul>";
        return new Response("<html><body>$resp</body></html>");
    }


    // --- Mètode 2: Buscar contactes pel nom ---
    // Mateixa ruta /contacte/{text}, però amb requirement diferent (només lletres)
    #[Route('/contacte/{text}', name:'buscar_contacte', requirements: ['text' => '[a-zA-Z]+'])]
    public function buscar(string $text): Response
    {
        $resultat = array_filter($this->contactes, 
            function($contacte) use ($text) {
                return stripos($contacte["nom"], $text) !== false;
            }
        );

        if (!$resultat) 
            return new Response("No s'han trobat contactes amb '$text'.");


        $resposta = "<h2>Resultats de la cerca: '$text'</h2>";
        foreach ($resultat as $contacte) {
            $resposta .= "<ul>".
                            "<li>{ $contacte['nom'] } </li>".
                            "<li>{ $contacte['telefon'] }</li>".
                            "<li>{ $contacte['email'] }</li>".
                         "</ul>";
        }
        return new Response("<html><body>$resposta</body></html>");

    }
}
?>
```

---

🧠 **Explicació del funcionament**

- Symfony analitza l’URL `/contacte/{param}` i comprova quin **requirement** compleix:
    - Si és numèric (`\\d+`), crida al mètode `fitxa()`.
    - Si conté lletres (`[a-zA-Z]+`), crida al mètode `buscar()`.
- D’aquesta manera **no hi ha conflicte** encara que la ruta siga la mateixa.
- L’avantatge és que l’usuari pot accedir amb una sola estructura d’URL:
    - `/contacte/3` → mostra el contacte amb codi 3.  
    - `/contacte/anna` → busca “anna” dins dels noms.

---

## 5. Separació MVC real

Els primers exemples encara barregen **vista i controlador**, ja que el text HTML està dins del mètode.

En un projecte real, el controlador només ha de **passar dades** a una **vista Twig** per a renderitzar el resultat. Ho veurem a tema següent.

Flux bàsic:

```
Usuari → Controlador → Model → Vista (HTML)
```

---

## 6. Parts obsoletes o a evitar

| Obsolet | Substitució actual |
|----------|--------------------|
| Versions Symfony < 5.4 | Treballar amb Symfony 6.x o superior |
| PHP 7.x | Utilitzar PHP 8.1 o superior |
| `@Route("/ruta")` | `#[Route("/ruta")]` |
| Retornar HTML directament amb `new Response()` | Fer servir `$this->render()` amb Twig |

---

## 7. Conclusió

El patró **MVC** és la base del desenvolupament amb Symfony:

- Separa clarament la **lògica**, **les dades** i **la presentació**.
- Millora l’escalabilitat i la seguretat del projecte.
- És compatible amb bones pràctiques com la injecció de dependències i l’ús de serveis.

> Symfony implementa el patró MVC de manera nativa, i cada part del framework està pensada per a respectar aquesta arquitectura.

---
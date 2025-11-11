---
title: "TEMA 4 — El contenidor de serveis i la injecció de dependències"
nav:
  - TEMA 4: 'Tema4.md'
---

# 📘 Tema 4 — El contenidor de serveis i la injecció de dependències

## 1. Introducció

En una aplicació moderna, les classes solen necessitar objectes d’altres classes per funcionar.  
Per exemple, un controlador pot necessitar un servei de registre de logs o accés a la base de dades.

Sense un sistema d’injecció de dependències, el codi seria rígid i difícil de mantenir, provocant:


```php
<?php

// ❌ Exemple antic
$logger = new Logger();
$logger->info('S’ha iniciat la pàgina');

?>
```

Amb **injecció de dependències**, és el *framework* (Symfony) qui crea i proporciona aquests objectes, anomenats **serveis**:

```php
<?php

// ✅ Exemple modern
public function __construct(private LoggerInterface $logger) {
}

public function index(): Response
{
    $this->logger->info('S’ha iniciat la pàgina');
    return new Response('Hola món');
}

?>
```

Això permet:

- Millorar l'estructura del codi:
    - Elimina la duplicitat de codi.
    - Redueix la dificultat de manteniment.
    - Desacobla les classes i les fa més independents.
- Augmnetar la flexibilitat:
    - Permet substituir fàcilment implementacions (p. ex. canviar el servei de logs).
    - Facilita la reutilització de components en diferents parts de l'aplicació.
- Afavorir el desenvolupament i les proves:
    - Simplifica les proves unitàries i la detecció d'errors.

---

## 2. El contenidor de serveis

Symfony disposa d’un **contenidor de serveis (Service Container)**:  un component central que s’encarrega de crear, configurar i oferir tots els objectes (serveis) de l’aplicació.

### 2.1. Què és un servei?

Un *servei* és qualsevol classe que realitza una tasca concreta dins de l’aplicació:

- Enviar correus (`MailerInterface`)
- Registrar esdeveniments (`LoggerInterface`)
- Accedir a la base de dades (`EntityManagerInterface`)
- Validar dades (`ValidatorInterface`)

El contenidor s’encarrega de gestionar-los, creant-los només quan cal.

### 2.2. Necessitat d’un sistema d’injecció de dependències

En una aplicació, diferents classes poden necessitar accedir a la mateixa base de dades.
Sense la **injecció de dependències**, cada classe hauria de crear la seua pròpia connexió, duplicant codi i recursos.

Amb la **injecció de dependències**, és el framework qui crea la connexió una sola vegada i la proporciona a totes les classes que la necessiten.

Symfony gestiona aquest procés mitjançant el **contenidor de serveis**, un component que crea i administra tots els objectes compartits de l’aplicació, anomenats **serveis**.

---

## 3. Configuració bàsica

Quan creem un projecte amb Symfony, ja trobem un fitxer de configuració anomenat `config/services.yaml`:

```yaml
parameters:
  locale: 'ca'          # Indica la localitazció i idioma per defecte

services:
  _defaults:
    autowire: true      # Symfony injecta automàticament el servei
    autoconfigure: true # Crea o registra el servei segons el seu tipus
    public: false       # Visibilitat: els serveis son privats per a Symfony

  App\:
    resource: '../src/'
    exclude:
      - '../src/DependencyInjection/'
      - '../src/Entity/'
      - '../src/Kernel.php'

  App\Controller\:
    resource: '../src/Controller'           # Ruta controladors HTTP
    tags: ['controller.service_arguments']  # Permet injecció dins dels mètodes
```

Symfony **escaneja automàticament** la carpeta `src/` i registra com a serveis totes les classes que troba, amb aquestes regles:

- Si estan dins d’un namespace conegut (`App\...`).
- I no són entitats (`Entity`) ni DTOs, ni configuracions internes (`DependencyInjection`), ni la classe principal (`Kernel.php`).
- Aleshores es registren com a serveis automàticament (gràcies a l’`autowire` i l’`autoconfigure` activats al services.yaml).

Això vol dir:

- Tot el que hi ha a `src/Service`, `src/Repository`, `src/Security`... serà un servei disponible.
- Symfony sabrà com crear-lo automàticament. Si alguna classe necessita un servei, l'injectarem al constructor.

---

### 3.1. Serveis públics i privats

Per defecte, els serveis són **privats** (només accessibles via injecció).  
Si cal exposar-ne algun directament, podem marcar-lo com **públic**:

```yaml
App\Service\MissatgeService:
  public: true
```

Només s’hauria de fer si és estrictament necessari (per exemple, per usar-lo en comandes o scripts externs).

---

## 4. Crear i utilitzar un servei propi

### 4.1 Definició d’un servei

Creem una classe que encapsule una funcionalitat, per exemple un servei de missatges:

**Fitxer:** `src/Service/MissatgeService.php`

```php
<?php

namespace App\Service;

class MissatgeService
{
    public function obtindreSalutacio(string $nom): string
    {
        return "Hola, $nom! Benvingut a Symfony 🚀";
    }
}

?>
```

Gràcies a l’`autowire`, no cal registrar-la. Symfony la detectarà automàticament.

### 4.2 Ús del servei en un controlador

**Fitxer:** `src/Controller/SalutacioController.php`

```php
<?php

namespace App\Controller;

use App\Service\MissatgeService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class SalutacioController extends AbstractController
{
    public function __construct(private MissatgeService $missatgeService) {

    }

    #[Route('/salutacio', name: 'salutacio')]
    public function index(): Response
    {
        $text = $this->missatgeService->obtindreSalutacio('Anna');
        return new Response($text);
    }
}

?>
```

👉 En el constructor, Symfony fa una cosa **automàtica i màgica**:

- Symfony detecta que el controlador necessita un objecte del tipus `MissatgeService`.
- Busca en el contenidor de serveis si ja existeix un servei registrat amb eixa classe.
    - Si existeix (i el normal és que sí, si està en `src/Service/MissatgeService.php`), Symfony el crea i l’injecta automàticament quan construeix el controlador.

💡 Açò s’anomena **Injecció de Dependències (DI)** i és una bona pràctica perquè el controlador no s’encarrega de crear el servei, sinó que el rep preparat per a utilitzar-lo.

---

## 5. Depuració de serveis

Symfony ofereix comandes per inspeccionar el contenidor:

- Per mostrar tots els serveis disponibles.

```bash
php bin/console debug:container

php bin/console debug:container App\

php bin/console debug:container App\Service
```

- Per buscar un servei concret:

```bash
php bin/console debug:container MissatgeService
```

- Per veure quines classes són autowirables:

```bash
php bin/console debug:autowiring
```

---

## 6. Exemple de servei predefinit

Symfony inclou molts **serveis predefinits** llestos per a utilitzar, com ara:

- `MailerInterface` → enviar correus electrònics  
- `LoggerInterface` → registrar missatges de log (errors, avisos, informació...)

Per utilitzar un servei, n’hi ha prou amb **injectar-lo al controlador** mitjançant el constructor.

---

### 6.1. Monolog - Servei de logs

**Monolog** és la biblioteca encarregada de gestionar els logs dins d’aplicacions PHP. És a dir, és l’eina que escriu els missatges en els fitxers, els envia a la consola, a correu, a serveis externs, etc.

Així, Monolog proporciona els “handlers”, que són els que decideixen on guardar el missatge:

| Handler              | Què fa                                           |
|----------------------|--------------------------------------------------|
| **StreamHandler**    | Guarda logs en fitxers (`var/log/{env}.log`)       |
| **BrowserConsoleHandler** | Envia logs a la consola del navegador       |
| **FirePHPHandler**   | Envia logs a l’extensió FirePHP                  |
| **SyslogHandler**    | Envia logs a Syslog (Linux/servidors)            |
| **SlackWebhookHandler** | Envia logs a un canal de Slack               |
| **etc.**             | Hi ha molts més handlers disponibles             |

En Symfony, el handler per defecte és `StreamHandler`:

- Si volem modificar el comportament per defecte, cal editar el fitxer de configuració `config/packages/monolog.yaml`.

Per això cal instal·lar:

```bash
composer require symfony/monolog-bundle
```

Este bundle:

- Registra Monolog en Symfony.
- Configura el sistema de logs
- Crea automàticament var/log/dev.log quan s’escriu el primer missatge

---

Veurem un exemple amb `LoggerInterface` per traure un missatge amb la data i hora de l'accés a la pàgina inicial.

**Fitxer:** `src/Controller/IniciController.php`

```php
<?php

namespace App\Controller;

use Psr\Log\LoggerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class IniciController extends AbstractController
{
    public function __construct(private LoggerInterface $logger) {

    }

    #[Route('/', name: 'inici')]
    public function inici(): Response
    {
        $dataHora = new \DateTime();
        $this->logger->info("Accés el " . $dataHora->format("d/m/Y H:i:s"));
        return $this->render('inici.html.twig');
    }
}

?>
```

📝 Notes:

- `\DateTime` → La barra invertida inicial \ vol dir “comença des de l’arrel del namespace”, i DateTime és una classe interna de PHP (forma part del nucli del llenguatge).
    - També podem importar la classe amb `use` i no utilitzar la `\` davant.  
- Els missatges es guarden en `var/log/dev.log` o `var/log/prod.log` segons l’entorn.
- Podríem haver passat directament l'objecte `LoggerInterface` al mètode `inici`. Però és recomanable injectar els serveis al constructor per a poder reutilitzar-los en diversos mètodes.

---

## 7. Exemple de servei propi

Podem crear **serveis propis** per encapsular funcionalitats o dades reutilitzables dins de l’aplicació.

En l’exemple de *Contactes*, extraurem l’array de contactes del controlador i el posarem en un **servei propi** anomenat `BDProva`.

Els serveis es poden crear dins de la carpeta `src/Service`, que cal crear si no existeix.  
Symfony detecta automàticament totes les classes d’aquesta carpeta com a serveis.

**Fitxer:** `src/Service/BDProva.php`

```php
<?php

namespace App\Service;

class BDProva
{
    private $contactes = array(
        array("codi" => 1, "nom" => "Salvador Sala",
              "telefon" => "638961244", "email" => "salvasala@simarro.org"),
        array("codi" => 2, "nom" => "Anna Llopis",
              "telefon" => "669332004", "email" => "annallopis@simarro.org"),
        array("codi" => 3, "nom" => "Marc Sanchis",
              "telefon" => "962286040", "email" => "marcsanchis@simarro.org"),
        array("codi" => 4, "nom" => "Laura Palop",
              "telefon" => "663568890", "email" => "laurapalop@simarro.org"),
        array("codi" => 5, "nom" => "Sara Sidle",
              "telefon" => "638765434", "email" => "sarasidle@simarro.org"),
    );

    public function get()
    {
        return $this->contactes;
    }
}

?>
```

**Fitxer:** `src/Controller/ContacteController.php`

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use App\Service\BDProva;

class ContacteController extends AbstractController
{
    private $contactes;

    public function __construct(BDProva $dades)
    {
        $this->contactes = $dades->get();
    }

    // La resta del controlador (fitxa, buscar...) continua igual
}

?>
```

📝 Notes:

- El servei `BDProva` s’injecta al **constructor** de `ContacteController`.
- Les dades es guarden en un atribut privat `$contactes`, accessible per qualsevol mètode del controlador.
- Recorda afegir la línia `use App\Service\BDProva;` per importar la classe.
- Symfony gestiona automàticament aquest servei gràcies a l’**autowiring** (no cal registrar-lo manualment a `services.yaml`).

---

## 8. Altres opcions

Ara que ja sabem com crear serveis propis i utilitzar-los, o bé utilitzar serveis de Symfony o de bundles de tercers, podem veure algunes opcions més avançades que afecten el contenidor de serveis i els serveis que utilitzem.

### 8.1. Combinar serveis

Quan en una classe o mètode determinat podem necessitar **més d’un servei**. 

Tenim dues alternatives:

**1. Passar diversos serveis al constructor**

```php
<?php

class UnaClasse
{
    private $contactes;
    private $logger;

    public function __construct(BDProva $dades, LoggerInterface $logger)
    {
        $this->contactes = $dades->get();
        $this->logger = $logger;
    }
}

?>
```

**2. Crear un servei combinat**

Podem encapsular els serveis dins d’una classe nova, per exemple `ServeiCombinat`, i injectar només aquesta.

```php
<?php

class ServeiCombinat
{
    private $contactes;
    private $logger;

    public function __construct(BDProva $dades, LoggerInterface $logger)
    {
        $this->contactes = $dades->get();
        $this->logger = $logger;
    }

    // Getters o mètodes per accedir als serveis interns
}

?>
```

Aleshores, la classe que els necessite només rebria un objecte d’aquest tipus:

```php
<?php

class UnaClasse
{
    private $servei;

    public function __construct(ServeiCombinat $servei)
    {
        $this->servei = $servei;
    }
}

?>
```

Aquesta segona opció millora la llegibilitat i facilita la reutilització de conjunts de serveis.

---

### 8.2. Arguments sense "autowiring"

L’opció `autowire` de `config/services.yaml` permet que Symfony cree automàticament els serveis necessaris en funció del tipus del paràmetre. No obstant això, **hi ha casos en què Symfony no pot autoinjectar valors** (p. ex. variables sense tipus o literals).

**🧩 Exemple**

Suposem que volem que el format de data siga personalitzable en el `IniciController`:

```php
<?php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Psr\Log\LoggerInterface;

class IniciController extends AbstractController
{
    private $logger;
    private $formatData;

    public function __construct(LoggerInterface $logger, $formatData)
    {
        $this->logger = $logger;
        $this->formatData = $formatData;
    }

    #[Route('/', name: 'inici')]
    public function inici()
    {
        $data_hora = new \DateTime();
        $this->logger->info("Accés: ". $data_hora->format($this->formatData));
        return $this->render('inici.html.twig');
    }
}

?>
```

Si intentem executar aquesta classe sense definir `$formatData`, Symfony llançarà un error, ja que **no sap d’on obtindre aquest valor**.

🔧 **Solució:**

Definir l’argument al final del fitxer `services.yaml`:

```yaml
App\Controller\IniciController:
    arguments:
        $formatData: 'd/m/y H:i:s'
```

Amb això, indiquem que el paràmetre `$formatData` del constructor de `IniciController` tindrà per defecte el valor `'d/m/y H:i:s'`.

📝 **Notes:**

- No utilitzes el tabulador per indentar (usa quatre espais).  
- Aquesta tècnica permet definir **paràmetres personalitzats** per a serveis que Symfony no pot autowirejar automàticament.

---

### 8.3. Paràmetres globals

Es poden definir **paràmetres de configuració globals** per a tots els serveis dins de la secció `parameters` de `config/services.yaml`. Aquests valors poden ser reutilitzats en qualsevol servei o part de l’aplicació.  

```yaml
parameters:
  locale: 'ca'
  app.nom_projecte: 'Gestor de Contactes'
  app.suport_email: '%env(SUPPORT_EMAIL)%'
  format_data_defecte: 'd/m/y H:i:s'

...

App\Controller\IniciController:
  arguments:
      $formatData: '%format_data_defecte%'
      $suportEmail: '%app.suport_email%'

```

I en `.env` podem definir les variables:

```
SUPPORT_EMAIL=suport@exemple.com
```

Al codi PHP:

```php
<?php

public function __construct(private string $suportEmail)
{
    // Symfony injecta automàticament el valor de l'entorn
}

?>
```

També pots accedir-hi via el contenidor:

```php
$suport = $this->getParameter('app.suport_email');
```

---

## 9. Resum final

| Conceptes clau | Descripció |
|----------------|------------|
| **Servei** | Classe que realitza una tasca concreta. Un servei = una tasca. |
| **Contenidor de serveis** | Sistema que crea i gestiona tots els serveis. |
| **Autowiring** | Symfony detecta i injecta serveis automàticament. |
| **Autoconfigure** | Afegeix configuració automàtica segons el tipus. |
| **Paràmetres i entorn** | Permeten definir configuracions globals o secretes. |
| **Bones pràctiques** | Injecció per constructor, serveis privats, i codi desacoblat. |

---

**Referències útils**

- [Documentació oficial de Symfony 7](https://symfony.com/doc/7.0/index.html)
- [Dependency Injection](https://symfony.com/doc/current/service_container.html)
- [Best Practices: Service Design](https://symfony.com/doc/current/best_practices.html#services)

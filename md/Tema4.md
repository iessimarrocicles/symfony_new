---
title: "TEMA 4 — El contenidor de serveis i la injecció de dependències (Symfony 7)"
nav:
  - TEMA 4: 'tema4.md'
---

# 🧩 Tema 4 — El contenidor de serveis i la injecció de dependències (Symfony 7)

## 1️. Introducció

En una aplicació moderna, les classes solen necessitar objectes d’altres classes per funcionar.  
Per exemple, un controlador pot necessitar un servei de registre de logs o accés a la base de dades.

Sense un sistema d’injecció de dependències, el codi seria rígid i difícil de mantenir:

```php
// ❌ Exemple antic
$logger = new Logger();
$logger->info('S’ha iniciat la pàgina');
```

Amb **injecció de dependències**, és el *framework* (Symfony) qui crea i proporciona aquests objectes, anomenats **serveis**:

```php
// ✅ Exemple modern
public function __construct(private LoggerInterface $logger) {}

public function index(): Response
{
    $this->logger->info('S’ha iniciat la pàgina');
    return new Response('Hola món');
}
```

Això permet:
- Desacoblar el codi.
- Substituir fàcilment implementacions (p. ex. canviar el servei de logs).
- Fer proves unitàries més senzilles.
- Reutilitzar components.

---

## 2️⃣ El contenidor de serveis

Symfony disposa d’un **contenidor de serveis (Service Container)**:  
una estructura central que s’encarrega de crear, configurar i oferir tots els objectes (serveis) de l’aplicació.

### Què és un servei?

Un *servei* és qualsevol classe que realitza una tasca concreta dins de l’aplicació:  
- Enviar correus (`MailerInterface`)
- Registrar esdeveniments (`LoggerInterface`)
- Accedir a la base de dades (`EntityManagerInterface`)
- Validar dades (`ValidatorInterface`)

El contenidor s’encarrega de gestionar-los, creant-los només quan cal.

---

## 3️⃣ Configuració bàsica (`config/services.yaml`)

Quan creem un projecte amb Symfony, ja trobem un fitxer de configuració:

```yaml
# config/services.yaml
parameters:
  locale: 'ca'

services:
  _defaults:
    autowire: true      # Symfony injecta automàticament les dependències
    autoconfigure: true # Registra el servei segons el seu tipus
    public: false       # Els serveis no són públics per defecte

  App\:
    resource: '../src/'
    exclude:
      - '../src/DependencyInjection/'
      - '../src/Entity/'
      - '../src/Kernel.php'

  App\Controller\:
    resource: '../src/Controller'
    tags: ['controller.service_arguments']
```

Gràcies a `autowire` i `autoconfigure`, Symfony detecta automàticament la majoria de serveis.  
Ja **no cal registrar-los manualment**.

---

## 4️⃣ Crear i utilitzar un servei propi

### 4.1 Definició d’un servei

Creem una classe que encapsule una funcionalitat, per exemple un servei de missatges:

```php
// src/Service/MissatgeService.php
namespace App\Service;

class MissatgeService
{
    public function obtindreSalutacio(string $nom): string
    {
        return "Hola, $nom! Benvingut a Symfony 🚀";
    }
}
```

Gràcies a l’autowiring, no cal registrar-la. Symfony la detectarà automàticament.

### 4.2 Ús del servei en un controlador

```php
// src/Controller/IniciController.php
namespace App\Controller;

use App\Service\MissatgeService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class IniciController extends AbstractController
{
    public function __construct(private MissatgeService $missatgeService) {}

    #[Route('/', name: 'inici')]
    public function index(): Response
    {
        $text = $this->missatgeService->obtindreSalutacio('Anna');
        return new Response($text);
    }
}
```

> ✅ Symfony crearà automàticament l’objecte `MissatgeService` i l’injectarà en el controlador.

---

## 5️⃣ Paràmetres i variables d’entorn

Els **paràmetres** s’utilitzen per a valors constants o configuracions globals.

```yaml
parameters:
  app.nom_projecte: 'Gestor de Contactes'
  app.suport_email: '%env(SUPPORT_EMAIL)%'
```

I en `.env` podem definir les variables:

```
SUPPORT_EMAIL=suport@exemple.com
```

Al codi PHP:

```php
public function __construct(private string $suportEmail)
{
    // Symfony injecta automàticament el valor de l'entorn
}
```

També pots accedir-hi via el contenidor:

```php
$suport = $this->getParameter('app.suport_email');
```

---

## 6️⃣ Serveis públics i privats

Per defecte, els serveis són **privats** (només accessibles via injecció).  
Si cal exposar-ne algun directament, podem marcar-lo com **públic**:

```yaml
App\Service\MissatgeService:
  public: true
```

Només s’hauria de fer si és estrictament necessari (per exemple, per usar-lo en comandes o scripts externs).

---

## 7️⃣ Bones pràctiques

| Recomanació | Descripció |
|--------------|------------|
| ✅ **Injecció per constructor** | És més clara i testejable que per mètode. |
| ✅ **Evita `get()`** | No accedisques al contenidor manualment (`$this->get()`) dins dels controladors. |
| ✅ **Fes els serveis immutables** | Declara propietats `readonly` o `private`. |
| ✅ **Divideix serveis per responsabilitat** | Un servei = una tasca. |
| ✅ **Aprofita els `ServiceSubscriber`** | Per injectar només els serveis necessaris dinàmicament. |

---

## 8️⃣ Depuració de serveis

Symfony ofereix comandes per inspeccionar el contenidor:

```bash
php bin/console debug:container
```

Mostrar tots els serveis disponibles.

Per buscar un servei concret:

```bash
php bin/console debug:container MissatgeService
```

O per veure quines classes són autowirables:

```bash
php bin/console debug:autowiring
```

---

## 9️⃣ Exemple complet

```php
// src/Service/CalculadoraService.php
namespace App\Service;

class CalculadoraService
{
    public function sumar(int $a, int $b): int
    {
        return $a + $b;
    }
}

// src/Controller/CalculadoraController.php
namespace App\Controller;

use App\Service\CalculadoraService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class CalculadoraController extends AbstractController
{
    public function __construct(private CalculadoraService $calculadora) {}

    #[Route('/suma/{a}/{b}', name: 'suma')]
    public function suma(int $a, int $b): Response
    {
        $resultat = $this->calculadora->sumar($a, $b);
        return new Response("El resultat de la suma és: $resultat");
    }
}
```

---

## 🔍 Resum final

| Conceptes clau | Descripció |
|----------------|------------|
| **Servei** | Classe que realitza una tasca concreta. |
| **Contenidor de serveis** | Sistema que crea i gestiona tots els serveis. |
| **Autowiring** | Symfony detecta i injecta serveis automàticament. |
| **Autoconfigure** | Afegeix configuració automàtica segons el tipus. |
| **Paràmetres i entorn** | Permeten definir configuracions globals o secretes. |
| **Bones pràctiques** | Injecció per constructor, serveis privats, i codi desacoblat. |

---

📘 **Referències útils**
- [Documentació oficial de Symfony 7 – Dependency Injection](https://symfony.com/doc/current/service_container.html)
- [Best Practices: Service Design](https://symfony.com/doc/current/best_practices.html#services)


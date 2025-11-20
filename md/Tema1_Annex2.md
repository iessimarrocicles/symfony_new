# Instal·lació pas a pas de Symfony en Windows

Aquesta guia explica què cal instal·lar en un ordinador **Windows** per poder treballar amb Symfony, amb un enfocament docent. Tots els passos estan pensats perquè l’alumnat entenga per a què serveix cadascun dels components.

> 💡 *Recomanació prèvia*: si el teu equip ho permet, la millor experiència amb Symfony en Windows sol obtindre’s usant **WSL2 + Ubuntu** (Linux dins de Windows). Tot i això, ací també tens la instal·lació **nativa en Windows** sense WSL.

---

## 0. Opció recomanada (resum): WSL2 + Ubuntu

Si vols treballar com en un servidor Linux, pots:

1. Activar **Subsistema de Windows per a Linux (WSL)**.
2. Instal·lar **Ubuntu** des de Microsoft Store.
3. Dins d’Ubuntu, seguir exactament la **guia d’Ubuntu** que ja tens preparada.

Això et dóna un entorn molt semblant a producció. Per a ús docent, però, sovint és suficient la instal·lació nativa que detallem a continuació.

---

## 1. Instal·lar un gestor de paquets per a Windows (Chocolatey) *(opcional però recomanat)*

**Chocolatey** és un gestor de paquets per a Windows semblant a `apt` en Ubuntu. Facilita molt instal·lar PHP, Git, etc.

### 1.1. Obrir PowerShell com a administrador

1. Cerca **PowerShell** en el menú d’inici.
2. Clica amb botó dret → *Executar com a administrador*.

### 1.2. Activar l’execució de scripts (només una vegada)

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
```

### 1.3. Instal·lar Chocolatey

```powershell
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
Set-ExecutionPolicy Bypass -Scope Process -Force;
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Quan acabe, tanca i torna a obrir PowerShell per assegurar que la comanda `choco` ja funciona.

---

## 2. Instal·lar PHP i extensions necessàries

Symfony necessita una versió moderna de PHP (8.1 o superior). Ho farem amb Chocolatey.

### 2.1. Instal·lar PHP

En una **PowerShell amb permisos d’usuari normal** (no cal administrador ara):

```powershell
choco install php --version=8.2.0 -y
```

> Si no especifica versió, instal·larà la més recent. L’important és que siga 8.1 o superior.

### 2.2. Afegir PHP al PATH (si no es fa automàticament)

En molts casos, Chocolatey ja ho configura. Comprova:

```powershell
php -v
```

Si mostra la versió de PHP, ja està correcte. Si diu que no es reconeix la comanda, caldrà afegir manualment la carpeta de PHP a la variable d’entorn **PATH**.

### 2.3. Extensions de PHP per Symfony

En Windows, les extensions es gestionen al fitxer `php.ini`. En instal·lacions via Chocolatey solen vindre moltes activables.

1. Localitza el directori de PHP, per exemple:
   - `C:\ProgramData\chocolatey\bin\php.exe` → normalment el PHP real és a `C:\tools\php82` o similar.
2. Obri el fitxer `php.ini` (si hi ha un `php.ini-development` o `php.ini-production`, copia i renombra un d’ells a `php.ini`).
3. Assegura’t que tens actives (sense `;` al principi) línies semblants a:

```ini
extension=mysqli
extension=pdo_mysql
extension=intl
extension=mbstring
extension=xml
extension=curl
extension=gd
extension=zip
```

**Per a què serveixen?**
- **mysqli / pdo_mysql**: connexió a MySQL/MariaDB.
- **intl**: internacionalització i formats.
- **mbstring**: cadenes multibyte (necessari per a Symfony).
- **xml**: maneig de XML (algunes eines internes de Symfony).
- **curl**: peticions HTTP.
- **gd**: manipulació d’imatges.
- **zip**: descompressió/creació de fitxers ZIP.

Després de modificar `php.ini`, guarda i tanca.

---

## 3. Instal·lar Git

**Git** és imprescindible per treballar amb repositoris i amb Symfony en equip.

Amb Chocolatey:

```powershell
choco install git -y
```

Comprova:

```powershell
git --version
```

---

## 4. Instal·lar Composer (gestor de dependències de PHP)

### 4.1. Descarregar l’instal·lador oficial

1. Ves a la pàgina oficial de Composer (des del navegador):
   - "Download Composer" (instal·lador per a Windows).
2. Executa l’instal·lador `.exe`.

### 4.2. Passos de l’instal·lador

- Selecciona la ruta de `php.exe` (normalment la detecta sola).
- Deixa marcada l’opció **Add to PATH** perquè pugues usar `composer` des de qualsevol terminal.

Quan acabe la instal·lació, comprova a PowerShell o CMD:

```powershell
composer -V
```

Això ha de mostrar la versió de Composer.

**Què és Composer?**
- És el gestor de dependències de PHP.
- Llig el fitxer `composer.json` del projecte i instal·la/actualitza llibreries.
- Symfony (framework i bundles) es distribueix a través de Composer.

---

## 5. Instal·lar Symfony CLI (eina oficial)

La **Symfony CLI** facilita molt el treball: crear projectes, arrencar servidor local, comprovar requisits, etc.

### 5.1. Descarregar l’instal·lador

1. Ves a la pàgina oficial de Symfony CLI.
2. Descarrega l’instal·lador per a Windows (`symfony-cli_windows_amd64.exe` o similar).
3. Executa l’instal·lador i segueix els passos (normalment afegirà `symfony` al PATH).

### 5.2. Comprovar instal·lació

Obri una nova finestra de PowerShell i executa:

```powershell
symfony -v
```

Si apareix la versió, ja està llest.

**Per a què serveix Symfony CLI?**
- Crear nous projectes amb esquelet complet.
- Arrencar un servidor de desenvolupament ràpid.
- Comprovar requisits (`check:requirements`).
- Analitzar seguretat de dependències (`security:check`).

---

## 6. Instal·lar una base de dades (MySQL / MariaDB / PostgreSQL)

Per a un entorn docent i per compatibilitat amb molts exemples, és habitual usar **MySQL** o **MariaDB**.

### 6.1. MySQL Server

Opció A – instal·lació amb Chocolatey:

```powershell
choco install mysql -y
```

Opció B – instal·lador gràfic:

1. Ves a la pàgina oficial de MySQL.
2. Descarrega **MySQL Installer for Windows**.
3. Tria `Developer Default` o `Server only` segons preferències.
4. Durant la instal·lació, configura:
   - Contrasenya de l’usuari `root`.
   - Mode d’autenticació (pots deixar el per defecte).

### 6.2. Verificar i provar

- Comprova que el servei MySQL s’ha iniciat.
- Pots usar **MySQL Workbench** o línia de comandes per a crear una base de dades de prova.

Exemple:

```sql
CREATE DATABASE symfony_demo CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

## 7. (Opcional) Instal·lar un servidor web: Apache o Nginx

Per a **desenvolupament**, normalment és suficient el servidor intern de Symfony CLI. Però si vols simular producció o fer pràctiques de configuració, pots instal·lar:

### 7.1. Apache amb Chocolatey

```powershell
choco install apache-httpd -y
```

Haureu de configurar després el `DocumentRoot` i els `VirtualHost` perquè apunten al directori `public/` del projecte Symfony.

### 7.2. Nginx amb Chocolatey

```powershell
choco install nginx -y
```

També s’haurà de configurar el `server` block perquè servisca el directori `public/`.

> Per a classe, moltes vegades és més pràctic usar únicament el servidor de Symfony CLI i explicar Apache/Nginx de forma teòrica o amb una pràctica específica.

---

## 8. Crear un projecte Symfony des de Windows

Un cop tens PHP, Composer i Symfony CLI, ja pots crear un projecte.

En una carpeta de treball (per exemple `C:\Users\alumne\Documents\symfony`):

```powershell
cd C:\Users\alumne\Documents
symfony new mi_projecte --full
```

Això crearà una carpeta `mi_projecte` amb:
- Estructura de directoris de Symfony (`config/`, `src/`, `templates/`, `public/`, etc.).
- Fitxers `composer.json`, `.env`, etc.

Si vols un projecte més lleuger:

```powershell
symfony new mi_projecte
```

---

## 9. Configurar la connexió a la base de dades

Dins de la carpeta del projecte hi ha un fitxer `.env` (i opcionalment `.env.local`). Allí es configura la connexió.

Exemple amb MySQL local:

```env
DATABASE_URL="mysql://root:contrasenya@127.0.0.1:3306/symfony_demo?serverVersion=8.0"
```

- **root**: usuari de la base de dades.
- **contrasenya**: la que hages configurat.
- **symfony_demo**: nom de la base de dades.

Després, pots crear l’esquema amb Doctrine (si el projecte el fa servir):

```powershell
php bin/console doctrine:database:create
php bin/console doctrine:schema:update --force
```

---

## 10. Iniciar el servidor de desenvolupament de Symfony

Des de la carpeta del projecte:

```powershell
cd C:\Users\alumne\Documents\mi_projecte
symfony serve -d
```

- L’opció `-d` fa que el servidor s’execute en segon pla (detached).
- Per veure l’URL del servidor, pots fer:

```powershell
symfony local:server:status
```

Normalment serà `https://127.0.0.1:8000` o semblant. Obri’l al navegador.

Per parar el servidor:

```powershell
symfony server:stop
```

---

## 11. Comprovar requisits i seguretat

### 11.1. Requisits de Symfony

```powershell
symfony check:requirements
```

Mostra si falta alguna extensió de PHP o si hi ha alguna configuració incorrecta.

### 11.2. Vulnerabilitats de seguretat en dependències

```powershell
symfony security:check
```

Comprova si les llibreries instal·lades via Composer tenen vulnerabilitats conegudes.

---

## 12. Resum per a l’alumnat

Per poder treballar amb Symfony en Windows, en essència necessitem:

1. **PHP** actualitzat (8.1+), amb extensions necessàries.
2. **Composer** per a gestionar les llibreries.
3. **Symfony CLI** per crear i gestionar projectes i servidor de desenvolupament.
4. **Git** per controlar versions del codi.
5. **Base de dades** (MySQL/MariaDB) per a la capa de persistència.
6. (Opcional) **Apache o Nginx** si volem treballar escenaris més propers a producció.

Amb aquests elements instal·lats i configurats, l’alumne/a ja pot crear projectes, generar controladors, entitats, formularis i repositoris, i començar a desenvolupar aplicacions web modernes amb Symfony des de Windows.


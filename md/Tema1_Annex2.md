# Instal·lació en Windows

Aquesta guia explica què cal instal·lar en un ordinador **Windows** per poder treballar amb Symfony.

> 💡 *Recomanació prèvia*: si el teu equip ho permet, la millor experiència amb Symfony en Windows sol obtindre’s usant **WSL2 + Ubuntu** (Linux dins de Windows). Tot i això, ací també tens la instal·lació **nativa en Windows** sense WSL.

---

## 1. Instal·lar PHP i extensions

Symfony requereix PHP 8.1 o superior. Per instal·lar-lo manualment:

1. Descarrega PHP per a Windows des de la pàgina oficial.
2. Descomprimeix-lo en una carpeta, per exemple C:/php.
3. Afig C:/php al PATH de Windows.
4. Copia el fitxer php.ini-development a php.ini.
5. Assegura't que tens actives (senese ; al principi) les extensions necessàries:
   ```bash
   extension=mysqli
   extension=pdo_mysql
   extension=intl
   extension=mbstring
   extension=xml
   extension=curl
   extension=gd
   extension=zip
   ```

Per a què serveixen?

- mysqli / pdo_mysql: connexió a MySQL/MariaDB.
- intl: internacionalització i formats.
- mbstring: cadenes multibyte (necessari per a Symfony).
- xml: maneig de XML (algunes eines internes de Symfony).
- curl: peticions HTTP.
- gd: manipulació d’imatges.
- zip: descompressió/creació de fitxers ZIP.

Després de modificar php.ini, guarda i tanca.

---

## 2. Instal·lar Git

1. Descarrega Git des de la pàgina oficial.
2. Instal·la’l deixant l’opció d’afegir al PATH.
3. Comprova amb:
   ```bash
   git --version.
   ```

---

## 3. Instal·lar Composer

Què és Composer?

- És el gestor de dependències de PHP.
- Llig el fitxer composer.json del projecte i instal·la/actualitza llibreries.
- Symfony (framework i bundles) es distribueix a través de Composer.

1. Descarrega l’instal·lador de Composer per a Windows o executa'l.
    - Indica la ruta del php.exe manual instal·lat (normalment la detecta sola).
    - Deixa marcada l’opció *Add to PATH* perquè pugues usar `composer` des de qualsevol terminal.
2. Comprova amb:
   ```bash
   composer -V
   ```

---

## 4. Instal·lar Symfony CLI

La **Symfony CLI** (eina oficial) facilita molt el treball: crear projectes, arrencar servidor local, comprovar requisits, etc.

### 4.1. Descarregar l’instal·lador

1. Ves a la pàgina oficial de Symfony CLI.
2. Descarrega l’instal·lador per a Windows (`symfony-cli_windows_amd64.exe` o similar).
3. Executa l’instal·lador i segueix els passos (normalment afegirà `symfony` al PATH).

### 4.2. Comprovar instal·lació

Obri una nova finestra de PowerShell i executa:

```powershell
symfony -v
```

Si apareix la versió, ja està funcionant.

---

## 5. Instal·lar una base de dades (MySQL / MariaDB / PostgreSQL)

Per a un entorn docent i per compatibilitat amb molts exemples, és habitual usar **MySQL** o **MariaDB**.

### 5.1. MySQL Server

1. Ves a la pàgina oficial de MySQL.
2. Descarrega MySQL Installer for Windows.
3. Tria `Developer Default` o `Server only` segons preferències.
4. Durant la instal·lació, configura:
   - Contrasenya de l’usuari `root`.
   - Mode d’autenticació (pots deixar el per defecte).

### 5.2. Verificar i provar

- Comprova que el servei MySQL s’ha iniciat.
- Pots usar **MySQL Workbench** o línia de comandes per a crear una base de dades de prova.

---

## 6. Instal·lar un servidor web

Per a desenvolupament, normalment és suficient el servidor intern de Symfony CLI. Però si vols simular producció o fer pràctiques de configuració, pots instal·lar:

### 6.1. Instal.lar Apache

1. Descarrega Apache per a Windows des de la pàgina d'Apache Lounge.
2. Descomprimeix el contingut en una carpeta, per exemple C:/Apache24.
3. Obri PowerShell com a administrador.
4. Executa: C:/Apache24/bin/httpd.exe -k install
5. Inicia el servei amb: net start apache2.4

Haureu de configurar després el `DocumentRoot` i els `VirtualHost` perquè apunten al directori `public/` del projecte Symfony.

> Per a classe, moltes vegades és més pràctic usar únicament el servidor de Symfony CLI i explicar Apache/Nginx de forma teòrica o amb una pràctica específica.

---

## 7. Comprovar requisits i seguretat

### 7.1. Requisits de Symfony

Mostra si falta alguna extensió de PHP o si hi ha alguna configuració incorrecta.

```powershell
symfony check:requirements
```

### 7.2. Vulnerabilitats de seguretat en dependències

Comprova si les llibreries instal·lades via Composer tenen vulnerabilitats conegudes.

```powershell
symfony security:check
```

---

## 8. Resum

Per poder treballar amb Symfony en Windows, en essència necessitem:

1. **PHP** actualitzat (8.1+), amb extensions necessàries.
2. **Composer** per a gestionar les llibreries.
3. **Symfony CLI** per crear i gestionar projectes i servidor de desenvolupament.
4. **Git** per controlar versions del codi.
5. **Base de dades** (MySQL/MariaDB) per a la capa de persistència.
6. (Opcional) **Apache o Nginx** si volem treballar escenaris més propers a producció.

Amb aquests elements instal·lats i configurats, l’alumne/a ja pot crear projectes, generar controladors, entitats, formularis i repositoris, i començar a desenvolupar aplicacions web modernes amb Symfony des de Windows.

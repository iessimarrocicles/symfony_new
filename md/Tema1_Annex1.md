# Instal·lació en Ubuntu

A continuació tens una guia detallada sobre tot allò que cal instal·lar per poder treballar amb Symfony en una màquina Ubuntu.

## 1. Actualitzar el sistema

```bash
sudo apt update && sudo apt upgrade -y
```

Actualitza l’índex de paquets i instal·la les actualitzacions pendents. És recomanable fer-ho abans d'instal·lar cap eina.

---

## 2. Instal·lar PHP i extensions

Symfony requereix una versió moderna de PHP (8.1 o superior). A Ubuntu sol estar disponible a través del repositori `ondrej/php`.

Afegir repositori PHP modern:

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```

Això permet accedir a versions de PHP que no venen per defecte a Ubuntu.

Instal·lar PHP i extensions recomanades per Symfony:

```bash
sudo apt install -y php8.2 php8.2-cli php8.2-common php8.2-mysql php8.2-xml \
php8.2-mbstring php8.2-zip php8.2-intl php8.2-curl php8.2-gd php8.2-sqlite3
```

Explicació de les extensions principals:

- **php-cli**: permet executar PHP des de terminal.
- **php-mysql / php-sqlite3**: connexions a bases de dades.
- **php-xml**: necessari per a components interns de Symfony.
- **php-mbstring**: gestiona cadenes multibyte.
- **php-intl**: internacionalització (traduccions, formats…).
- **php-curl**: peticions HTTP.
- **php-zip**: gestió de fitxers ZIP.
- **php-gd**: manipulació d’imatges.

---

## 3. Instal·lar Composer

Symfony depén de Composer per a instal·lar paquets i eines. És el nostre gestor de dependències de PHP.

Descarregar Composer:

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
sudo mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer
```

Ara tens composer disponible globalment.

---

## 4. Instal·lar Symfony CLI

La Symfony CLI (eina oficial) permet crear projectes, arrancar servidors locals i comprovar compatibilitat.

Instal·lació:

```bash
wget https://get.symfony.com/cli/installer -O - | bash
sudo mv /home/$USER/.symfony*/bin/symfony /usr/local/bin/symfony
```

Comprovació:

```bash
symfony -v
```

---

## 5. Instal·lar un servidor web

Tot i que el servidor integrat funciona bé en desenvolupament, pots instal·lar:

Apache:

```bash
sudo apt install apache2 libapache2-mod-php8.2
```

O bé Nginx:

```bash
sudo apt install nginx
```

Normalment per a Symfony en producció es recomana **Nginx** per rendiment, però Apache és més senzill per a principiants.

---

## 6. Instal·lar una base de dades

Pots triar entre *MySQL*, *MariaDB* o *PostgreSQL*.

Nosaltres utilitzarem el SGBD de MySQL:

```bash
sudo apt install mysql-server -y
```

Simples passos interactius milloren la seguretat: contrasenya root, eliminar usuaris anònims, etc.

```bash
sudo mysql_secure_installation
```

---

## 7. Comprovar compatibilitat

Podem comprovar si falta alguna extensió de PHP o configuració.

```bash
symfony check:requirements
```

---

## 8. Validador de seguretat de dependències

Podem comprovar vulnerabilitats conegudes en paquets de Composer.

```bash
symfony security:check
```

---

## 🎉 Symfony funcionant

Aquesta configuració és la recomanada per a entorns docents i de desenvolupament professional.

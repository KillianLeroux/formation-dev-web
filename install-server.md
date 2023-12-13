# Installer un serveur web

## Installation de logiciels

### Apache

```sh
sudo apt-get update
sudo apt-get install apache2
```

### PHP

On installe la dernière version de PHP à date, à savoir la 8.3.

#### Installation des pré-requis

```sh
sudo apt-get install ca-certificates apt-transport-https software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
```

### Installation de PHP 8.3

```sh
sudo apt-get install php8.3
```

### Installation des modules nécessaires pour nos applis

```sh
sudo apt install libapache2-mod-php8.3
sudo apt install php8.3-mysql php8.3-imap php8.3-ldap php8.3-xml php8.3-curl php8.3-mbstring php8.3-zip
sudo systemctl restart apache2
php -m
```

### Installation de MariaDB (fork de MySQL)

On installe les packages qui vont bien :

```sh
sudo apt-get install mariadb-server
```

On vérifie que MariaDB est bien installé et lancé :

```sh
sudo mysql
```

#### Création d'un super utilisateur pour gérer nos BDD

On exécute les deux requetes SQL ci-dessous après avoir faire le `sudo mysql` :

```sql
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
```

### Installation de Adminer

```sh
sudo apt-get install adminer
sudo a2enconf adminer
sudo systemctl reload apache2
```

### Installation d'outils utiles

#### Git

```sh
sudo apt-get install git
```

#### Zip

```sh
sudo apt-get install zip
```

#### Composer

```sh
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

## Configuration

### DocumentRoot Apache

DocumentRoot, c’est le dossier contenant les fichiers du site livré par Apache.
Par exemple, dans la VM (comme dans ce serveur), c’est /var/www/html

Suppression du fichier index.html par défaut d’Apache 2
Et modification des droits d’accès du DocumentRoot

```sh
sudo chown -Rf student:www-data /var/www/html
sudo rm -f /var/www/html/index.html
```

### URL Rewriting

```sh
sudo a2enmod rewrite
sudo nano /etc/apache2/apache2.conf
```

Dans le fichier `/etc/apache2/apache2.conf` on modifie la configuration du répertoire `/var/www` comme suit :

```apache
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
```

On redémarre Apache pour que ça prenne en compte notre nouvelle configuration :

```sh
sudo systemctl restart apache2
```

### Config Git

On va créer une clé SSH pour notre compte GitHub afin que le serveur se connecte à nos dépôts GitHub facilement.
Ainsi, on pourra cloner nos dépôts et mettre en production nos sites internet !

#### Création de la paire de clés privée/publique

On commence par créer une clé sur notre machine qui devra accéder à GitHub :

```sh
ssh-keygen -t ed25519 -C "adresse_mail_de_ton_compte_github@mail.com"
```

La commande est interactive, voici un exemple :
(on a laissé volontairement toutes les réponses vides)

```sh
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_ed25519.
Your public key has been saved in /home/ubuntu/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:ozOgoiITeANJlGWTcxsvmT5Su0t3G+3XJFywMfMmWzo your_github_email@example.com
The key's randomart image is:
+--[ED25519 256]--+
|..o+.            |
| o.o.o       =   |
|..  o *       B  |
|o    * .     o = |
|..  + o S   . B  |
|o oo = . ..  E . |
|.o....*. o .  =  |
|=.  ...o. +  . . |
|+.   ..  . ..    |
+----[SHA256]-----+
```

Vérifier la création de la clé publique et copier le contenu du fichier qui va être affiché :

```sh
cat ~/.ssh/id_ed25519.pub
```

#### Ajouter la clé SSH à son compte GitHub

Aller à l’URL [https://github.com/settings/profile] puis :

- `SSH & GPG keys` dans le menu de gauche
- On clique sur le bouton `New SSH Key` à droite en haut
- On nomme la clé comme on le souhaite (exemple : "VM Cloud O'clock")
- On colle dans la zone de texte le contenu du fichier récupéré avec la commande "cat"
- On valide l'ajout de la clé

#### Configuration de base de git

On exécute ces commandes pour appliquer la configuration de base de git :

```sh
git config --global user.name "github_username"
git config --global user.email "github_email@my.com"
git config --global push.default simple
git config --global color.ui true
```

On peut alors se rendre dans le répertoire `/var/www/html` et récupérer un projet avec `git clone` suivi de l'URL SSH.

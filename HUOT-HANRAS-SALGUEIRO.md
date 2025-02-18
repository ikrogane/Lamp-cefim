HUOT romain
HANRAS nathant
SALGUEIRO RAPHAEL
### **Dépendance :** 
installation des dépendances requises / éventuellement requises lors de l'utilisation du glpi pour évité tout conflits possibles
 
```
apt update && apt upgrade -y
apt install apache2 php mariadb-server -y
apt install php-intl
```


il y a aussi besoin de tar qui est normalement présent dans les distribution linux, il faudra tout de même l'installer si il n'est pas présent sur votre distribution 

#### **installation de GLPI (pour nous, la version 10.0.17) :**
installation a l'aide de wget 

```
wget https://github.com/glpi-project/glpi/releases/download/10.0.17/glpi-10.0.17.tgz
```


adaptez la commande pour choisir une autre version si besoin en modifiant les "x", 10.0.00 par exemple.
```
wget https://github.com/glpi-project/glpi/releases/download/xx.x.xx/glpi-xx.x.xx.tgz
```


après avoir mis le fichier .tgz dans le répertoire souhaité tel que "media/" par exemple, décompresser l'archive .tgz a l'aide de la commande suivante : 
```
tar -xvf glpi-10.0.17.tgz glpi/
```


##### **Création et configuration du VHOST pour GLPI  

ouvrir le fichier suivant :
```
nano /etc/apache2/sites-available/glpi.conf
```


le fichier glpi.conf contient le 'code' suivant :
```
<VirtualHost *:80>
    ServerName glpi.localhost

    DocumentRoot /var/www/glpi/public

    # If you want to place GLPI in a subfolder of your site (e.g. your virtual host is serving multiple applications),
    # you can use an Alias directive. If you do this, the DocumentRoot directive MUST NOT target the GLPI directory itself.
    # Alias "/glpi" "/var/www/glpi/public"

    <Directory /var/www/glpi/public>
        Require all granted

        RewriteEngine On

        # Ensure authorization headers are passed to PHP.
        # Some Apache configurations may filter them and break usage of API, CalDAV, ...
        RewriteCond %{HTTP:Authorization} ^(.+)$
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

        # Redirect all requests to GLPI router, unless file exists.
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
</VirtualHost>
```

le "80" signifie que le protocole utilisé pour la connexion sera http et non pas https qui lui utilise principalement le port 443.



puis on active le VHOST avec les commande suivante :
```
a2ensite glpi.conf
a2enmod rewrite
systemctl restart apache2
```


il est préconisé de copier le fichier décompressé du GLPI dans /var/www/glpi :
```
cp -r /la_position_du_glpi/gpli /var/www/glpi
```

##### **Debug :**

il y a parfois des dépendances non installées, ce qui peut généralement être arranger avec un peut de debug, on peut connaitre les paquets et dépendances restantes avec la commande suivante : 
```
php bin/console glpi:system:check_requirements
```
avec la sortie de cette commande on peut en déduire des dépendances manquantes étant donnée qu'une liste de toutes les dépendances nous sera affiché


##### **Vérification du bon fonctionnement de l'interface web :**

Maintenant que le site est mis en ligne, il est possible d'y accéder :

depuis un ordinateur tiers qui a une interface, que ce soit linux ou windows, depuis un navigateur web accédez a la page web de GLPI grâce a l'adresse IP ou au nom du serveur web si un serveur DNS est présent dans le réseau et qu'il a été configuré en conséquence 

# Deploiement d'une app accessible sur le réseau à partir du nom de domain et nom de l'@ip

Ce guide décrit les étapes pour rendre une application Django accessible sur un réseau local via un nom de domaine personnalisé, sans utiliser l'adresse IP. Nous utilisons Apache comme serveur HTTP et `dnsmasq` pour la résolution DNS locale.

---

## Prérequis

- Une application Django fonctionnelle.
- Apache et `libapache2-mod-wsgi-py3` installés.
- `dnsmasq` installé sur la machine hôte.
- Accès root pour effectuer les configurations.

---

## Étape 1 : Installer les outils requis

### Installer Apache et mod_wsgi
```bash
sudo apt update
sudo apt install apache2 libapache2-mod-wsgi-py3
```

### Installer dnsmasq
```bash
sudo apt install dnsmasq
```

---

## Étape 2 : Configurer dnsmasq

1. **Modifier la configuration de `dnsmasq`** :
   Ouvre le fichier de configuration :
   ```bash
   sudo nano /etc/dnsmasq.conf
   ```

   Ajoute les lignes suivantes :
   ```
   listen-address=127.0.0.1
   address=/monapp.local/127.0.0.1
   ```
   - Remplace `monapp.local` par le nom de domaine souhaité.
   - Si tu veux gérer plusieurs sous-domaines, utilise une wildcard :
     ```
     address=/.monapp.local/127.0.0.1
     ```

2. **Redémarrer `dnsmasq`** :
   ```bash
   sudo systemctl restart dnsmasq
   ```

3. **Configurer le résolveur DNS local** :
   - Modifie le fichier `/etc/resolv.conf` pour utiliser `dnsmasq` :
     ```bash
     sudo nano /etc/resolv.conf
     ```
     Ajoute cette ligne au début :
     ```
     nameserver 127.0.0.1
     ```

   - Rends cette configuration persistante en modifiant `/etc/systemd/resolved.conf` :
     ```bash
     sudo nano /etc/systemd/resolved.conf
     ```
     Ajoute ou modifie cette ligne :
     ```
     DNS=127.0.0.1
     ```
     Redémarre le service :
     ```bash
     sudo systemctl restart systemd-resolved
     ```

---

## Étape 3 : Configurer Apache

1. **Créer un fichier de configuration Apache pour le domaine** :
   Crée un fichier de configuration dans `/etc/apache2/sites-available/monapp.local.conf` :
   ```bash
   sudo nano /etc/apache2/sites-available/monapp.local.conf
   ```

   Ajoute le contenu suivant :
   ```apache
   <VirtualHost *:80>
       ServerName monapp.local
       DocumentRoot /path/to/your/django/project

       <Directory /path/to/your/django/project>
           Require all granted
           AllowOverride All
       </Directory>

       WSGIDaemonProcess django_app python-path=/path/to/your/django/project python-home=/path/to/your/venv
       WSGIProcessGroup django_app
       WSGIScriptAlias / /path/to/your/django/project/project_name/wsgi.py

       ErrorLog ${APACHE_LOG_DIR}/monapp.local-error.log
       CustomLog ${APACHE_LOG_DIR}/monapp.local-access.log combined
   </VirtualHost>
   ```
   - Remplace `/path/to/your/django/project` par le chemin vers ton projet.
   - Remplace `/path/to/your/venv` par le chemin vers ton environnement virtuel Python.

2. **Activer le site et les modules nécessaires** :
   ```bash
   sudo a2ensite monapp.local
   sudo a2enmod wsgi
   sudo a2enmod rewrite
   sudo systemctl restart apache2
   ```

---

## Étape 4 : Configurer les permissions

1. **Accorder les permissions à Apache** :
   ```bash
   sudo chown -R www-data:www-data /path/to/your/django/project
   sudo chmod -R 755 /path/to/your/django/project
   ```

2. **Autoriser Apache à accéder au dossier utilisateur (si nécessaire)** :
   Si ton projet est dans `/home/username`, donne l'accès à Apache :
   ```bash
   sudo chmod +x /home/username
   ```

---

## Étape 5 : Tester l'application

1. Accède à l'application depuis le même PC en visitant :
   ```
   http://monapp.local
   ```

2. Si tout fonctionne, l'application est accessible via le domaine local configuré.

---

## Dépannage

1. **Erreur 403 (Forbidden)** :
   - Assure-toi que les permissions sur le dossier de ton projet sont correctes.
   - Vérifie la configuration de la directive `<Directory>` dans le fichier Apache.

2. **Erreur 500 (Internal Server Error)** :
   - Consulte les journaux Apache :
     ```bash
     sudo tail -f /var/log/apache2/error.log
     ```

3. **Le domaine ne résout pas** :
   - Vérifie que `dnsmasq` est en cours d'exécution :
     ```bash
     sudo systemctl status dnsmasq
     ```
   - Teste la résolution DNS :
     ```bash
     nslookup monapp.local
     ```

---

Avec ces étapes, tu peux rendre une application Django accessible via un nom de domaine local personnalisé à l'aide d'Apache et `dnsmasq`.

NB: la supression d'une informations dans les fichiers de configuration peut entrainer des problèmes.


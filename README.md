# Ansible Role: WordPress Installer

Ce rôle installe une pile LAMP (Apache, MariaDB, PHP) complète et déploie la dernière version de WordPress.

Il est conçu pour être idempotent et fonctionner sur les familles de systèmes d'exploitation Debian (Ubuntu) et RedHat (Rocky Linux, CentOS).

## Prérequis

Les collections Ansible suivantes doivent être installées sur votre machine de contrôle :

```bash
ansible-galaxy collection install community.mysql
ansible-galaxy collection install community.general
```

## Variables du Rôle

Les variables suivantes peuvent être surchargées pour personnaliser l'installation. Elles se trouvent dans `defaults/main.yml`.

| Variable           | Valeur par défaut           | Description                               |
| ------------------ | --------------------------- | ----------------------------------------- |
| `db_name`          | `"wordpress"`               | Le nom de la base de données WordPress.   |
| `db_user`          | `"wp_user"`                 | L'utilisateur pour la base de données.    |
| `db_password`      | `"SecurePassword123"`       | Le mot de passe pour l'utilisateur de la BDD. |
| `db_root_password` | `"SuperSecureRootPassword"` | Le mot de passe root pour MariaDB.        |
| `http_host`        | `"localhost"`               | Le nom d'hôte utilisé dans le vhost Apache. |
| `document_root`    | `"/var/www/wordpress"`      | Le chemin où les fichiers WordPress seront installés. |

## Exemple de Playbook

1.  Créez un fichier d'inventaire, par exemple `inventory.ini`:

    ```ini
    [webservers]
    ubuntu_server ansible_host=192.168.1.10
    rocky_server ansible_host=192.168.1.11
    ```

2.  Créez un playbook, par exemple `deploy_wordpress.yml`:

    ```yaml
    ---
    - name: Deploy WordPress on all webservers
      hosts: webservers
      become: yes
      roles:
        - ansible-role-wordpress
    ```

## Exécution

Lancez le playbook avec la commande suivante :

```bash
ansible-playbook -i inventory.ini deploy_wordpress.yml
```

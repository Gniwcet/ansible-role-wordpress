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

## Exemple de Déploiement

1.  **Créez un fichier d'inventaire**, par exemple `inventaire`, en structurant vos serveurs par OS.

    ```ini
    [ubuntu]
    client1 ansible_user=root ansible_ssh_pass=P@ssw0rd ansible_ssh_common_args='-o StrictHostKeyChecking=no'
    client2 ansible_user=root ansible_ssh_pass=P@ssw0rd ansible_ssh_common_args='-o StrictHostKeyChecking=no'

    [rocky]
    client3 ansible_user=root ansible_ssh_pass=P@ssw0rd ansible_ssh_common_args='-o StrictHostKeyChecking=no'
    client4 ansible_user=root ansible_ssh_pass=P@ssw0rd ansible_ssh_common_args='-o StrictHostKeyChecking=no'

    [clients:children]
    ubuntu
    rocky

    [clients:vars]
    ansible_python_interpreter=/usr/bin/python3
    ```
    *   `ansible_ssh_pass`: Spécifie le mot de passe de connexion SSH.
    *   `ansible_ssh_common_args`: Permet de désactiver la vérification de la clé d'hôte SSH, utile en environnement de test.

2.  **Créez un playbook**, par exemple `playbooks/deploy_wordpress.yml`, pour appliquer le rôle au groupe parent `clients`.

    ```yaml
    ---
    - name: Deploy WordPress
      hosts: clients
      become: yes
      roles:
        - roles/ansible-role-wordpress
    ```

## Exécution

Lancez le playbook en spécifiant votre fichier d'inventaire.

```bash
# Placez-vous dans le dossier de vos playbooks
cd playbooks/

# Lancez le playbook en utilisant l'inventaire situé dans le dossier parent
ansible-playbook -i ../inventaire deploy_wordpress.yml
```

### Note sur la Sécurité
Stocker les mots de passe en clair dans l'inventaire (`ansible_ssh_pass`) est pratique pour les tests et les laboratoires, mais est fortement déconseillé pour les environnements de production. Pour la production, l'utilisation de clés SSH ou d'Ansible Vault pour chiffrer les secrets est la méthode recommandée.

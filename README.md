# Ansible Role: WordPress Installer

Ce rôle installe une pile LAMP (Apache, MariaDB, PHP) complète et déploie la dernière version de WordPress.

Il a été spécifiquement conçu et renforcé pour être **extrêmement robuste** et fonctionner de manière fiable sur les familles de systèmes d'exploitation Debian (Ubuntu) et RedHat (Rocky Linux, CentOS), y compris dans des **environnements conteneurisés minimaux (comme Docker) qui ne disposent pas d'un système d'initialisation `systemd` complet.**

## Prérequis

Les collections Ansible suivantes doivent être installées sur votre machine de contrôle. Si vous rencontrez des erreurs de modules non trouvés, forcez la réinstallation pour réparer un éventuel cache corrompu (`--force`).

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

## Notes sur l'Implémentation

Ce rôle utilise des méthodes directes pour gérer les services afin de garantir son fonctionnement dans des conteneurs sans `systemd`.

*   **Gestion de MariaDB :** Le processus `mariadb` est géré directement via les commandes `mariadb-install-db` et `mysqld_safe`. Le rôle n'utilise **pas** le module `ansible.builtin.service` pour la base de données.
*   **Nettoyage :** Pour garantir l'idempotence et la propreté, le rôle supprime et recrée le répertoire de données de MariaDB à chaque exécution. C'est un comportement voulu pour les environnements de test. Pour un usage en production, cette étape (`(RESET) Remove previous MariaDB...`) devrait être retirée ou conditionnée.

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

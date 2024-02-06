# DevOps TP3

## Ansible

### Inventories 

```yml

all:
        vars:
                ansible_user: centos
                ansible_ssh_private_key_file: /mnt/c/Users/yoann/id_rsa

        children:
                prod:
                        hosts: centos@yoann.cathelain.takima.cloud
```

`ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"` Installation de la distribution de l'OS


## Roles

On a créer un role Docker pour découper les responsabilités suivant les roles, ce role devra installer Docker.

On met dans le répertoire tasks toutes les task qui étaient précedemment dans le playbook.yml

Pour exécuter ce role on rajoute dans le playbook.yml

```yml

roles:
    - docker
```
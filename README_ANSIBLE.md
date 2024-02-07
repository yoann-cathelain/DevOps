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
    - network
    - db
    - app
    - proxy
```

## Task type pour la création d'un conteneur Docker par Ansible

```yml


  - name: Install python3
    yum:
      name: python3
      state: present

  - name: Install docker with Python 3
    pip:
      name: docker
      executable: pip3
    vars:
      ansible_python_interpreter: /usr/bin/python3


  - name: Create DB container
    vars:
      ansible_python_interpreter: /usr/bin/python3
    docker_container:
      name: database
      networks: 
        - name: app-network
      image: jaykabonk/database-tp1

```

On va chercher les images des conteneurs déployés sur le DockerHub

On installe python pour que l'interpréteur d'ansible aille chercher dans le répertoire python3


## Mise en place d'un Ansible Vault

Le fichier var.yml dans le répertoire vars correspond aux variables encryptés présentes dans le ansible-vault

`ansible-playbook -i inventories/setup.yml -e @vars/var.yml playbook.yml --ask-vault-pass` pour exécuter le playbook avec le vault

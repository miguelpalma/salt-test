# Salt *state tree*

This repository is a test [Salt](http://www.saltstack.com/) *state tree* designed for basic admin
tasks.

## Install a development/test environment

1. Clone this repository **with the recursive flag** (`git clone --recursive …`) so that submodules
    in directory `formulas` are also cloned.

2. Start a virtual environment en SSH into it (next items will assume you are logged into this box).
    This Git repository contains a `Vagrantfile` that you can use by installing vagrant and then
    running:

    ```
    # Downloads a image from Vagrant servers (or the one you like: https://vagrantcloud.com/discover/featured
    # event though we will refer to a Fedora 20 base image in the rest of this document)
    vagrant box add chef/fedora-20
    vagrant up
    vagrant ssh
    # You should update all installed packages before you do anything else
    yum update
    ```

3. Install *salt minion*: `yum install salt-minion`

4. Edit `/etc/salt/minion` to set the following values (or just copy `/vagrant/conf/etc_salt_minion`
    to `/etc/salt/minion`)

    ```
    # …
    file_client: local    # we are going to run masterless so the minion needs to know salt states are local
    # …
    file_roots:           # places where the minion can find salt states
      base:
        - /vagrant
    # …                   # 'pillars' are salt files with custom values
    pillar_roots:
      base:
        - /vagrant/pillar
    # …
    ```

5. Optional: if you plan on wiping your VM to test your Salt tree, it would be wise to create a
    snapshot of your VM now so that you will not need to go through these steps anymore.

6. Place your own pillar files in directory `pillar` with the following hierarchy:

  - `pillar/users.sls`: see `formulas/apache-formula/pillar.example`

7. To run the Salt tree:

    ```
    # This directory is a mount of the Git directory on your local machine
    cd /vagrant
    salt-call state.highstate
    ```

## Adding a formula to use

```
cd formulas
git submodule add ${GIT_REPO}
# Edit /etc/salt/minion to add the following:
file_roots:
  - /vagrant/formulas/${FORMULA_REPO_NAME}
```

## TODO

- See how to automatically include formulas into top.sls
  https://git.forgeservicelab.fi/jrodrigu/masterless-formulas/blob/e70e4e11d4c4ec1f66b1374d2335e745f4827bce/ml-wordpress/init.sls

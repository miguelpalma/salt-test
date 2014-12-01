# Salt *state tree*

This repository is a test [Salt](http://www.saltstack.com/) *state tree* designed for basic admin
tasks.

## Install a development/test environment

1. Start a virtual environment en SSH into it (next items will assume you are logged into this box).
    This Git repository contains a `Vagrantfile` that you can use by installing vagrant and then
    running:

    ```
    # Downloads a image from Vagrant servers (or the one you like: https://vagrantcloud.com/discover/featured)
    vagrant box add chef/fedora-20
    vagrant up
    vagrant ssh
    # You should update all installed packages before you do anything else
    yum update
    ```

2. Install *salt minion*:

3. Edit `/etc/salt/minion` to set the following values

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

4. To run the Salt tree:

    ```
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

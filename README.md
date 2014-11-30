# Salt *state tree*

This repository is a test [Salt](http://www.saltstack.com/) *state tree* designed for basic admin
tasks.

## Install a development/test environment

1. Start a virtual environment en SSH into it (next items will assume you are logged into this box).

2. Install *salt minion*:

3. Edit `/etc/salt/minion` to set the following values

    ```
    # …
    file_client: local
    # …
    file_roots:
      base:
        - /vagrant
    # …
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
  - /vagrant/formulas/users-formula
```

## TODO

- See how to automatically include formulas into top.sls
  https://git.forgeservicelab.fi/jrodrigu/masterless-formulas/blob/e70e4e11d4c4ec1f66b1374d2335e745f4827bce/ml-wordpress/init.sls

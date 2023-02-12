# PoC k8s secrets management with [SOPS](https://github.com/mozilla/sops)

PoC using [SOPS](https://github.com/mozilla/sops) to manage secrets for k8s with [age](https://github.com/FiloSottile/age) encryption.

- [PoC k8s secrets management with SOPS](#poc-k8s-secrets-management-with-sops)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Examples](#examples)
  - [VScode extension](#vscode-extension)
    - [Example file](#example-file)
    - [Generate age key](#generate-age-key)
    - [Configure](#configure)

## Introduction

## Prerequisites

- Install SOPS

````shell
brew install sops
````

- Install age

````shell
brew install age
````

*Note: SOPS handles AWS KMS, GCP KMS, Azure Key Vault, age, and PGP. We will use `age` along this document.*

- Optionally, install miniKube to run the PoC locally:

```shell
brew install minikube
```

*Note: we use brew, but you can install the tools on Linux.*

## Examples

- [First example](./first-example/README.md): generate age key, encrypt with SOPS, decrypt with SOPS, use `.sops.yaml` config file.
- [Multi-users](./multi-users/README.md): handle multiple users with SOPS and age, remove a user.
- [Multi-environments](./multi-environments/README.md): handle multiple environments with SOPS and age.
- [Kubernetes](./kubernetes/README.md): use SOPS and age to manage secrets for k8s.

## VScode extension

### Example file

We create a file `example.yaml` with the following content:

```yaml
user: admin
password: mySuperSecretPassword
```

### Generate age key

We want to encrypt this file with SOPS and age. First, we need to create a key pair with age:

```shell
# create directory to store the keys
mkdir -p secrets/keys
# generate a key pair
age-keygen -o secrets/keys/age.txt
```

### Configure

Download the [SOPS extension](https://marketplace.visualstudio.com/items?itemName=signageos.signageos-vscode-sops) for VScode.

Add in `.vscode/settings.json`:

```json
{
    "sops.defaults.ageKeyFile": "./secrets/keys/age.txt"
}
```

It will automatically encrypt/decrypt the file when you save it.

The extension is not perfect, but it works. Double check that it does what you want!

You can use the same `.vscode/settings.json` across your different project and follow always the same structure for your secrets and put it in `secrets/keys/` and add this directory to your `.gitignore`.

# Kubernetes secrets and SOPS

The use-case here is that you have a repository with a kubernetes secret file, and you want to encrypt it with SOPS.

We assume you will execute the commands in this document in the `kubernetes` directory.

```shell
cd kubernetes
```

We'll suppose that you have already installed SOPS, age and minikube (if you don't have a kubernetes instance).

## Example file

First, we create a kubernetes secret file `secrets.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:
  username: admin
  password: mySuperSecretPassword
```

## Generate age key

We want to encrypt this file with SOPS and age. First, we need to create a key pair with age:

```shell
# create directory to store the keys
mkdir -p secrets/keys
# generate a key pair
age-keygen -o secrets/keys/age.txt
```

## Create SOPS config file

We create a SOPS config file `.sops.yaml`:

```yaml
creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: ^(data|stringData)$
    encryption_method: age
    age: "age1j0e4xycce25j08xcswuzh60qculvk70s8av9vd4q05wxvt9app0q96jlc2"

sops:
  encryption_method: age
```

Note that we encrypt fields `data` and `stringData` of the kubernetes secret file.

## Encrypt the file

```bash
sops -e secrets.yaml > secrets.enc.yaml
```

## Create the secret

```shell
kubectl create -f secrets.enc.yaml
```

## Deploy secrets

Launch minikube:

```shell
minikube start
```

Deploy secrets:

```shell
export SOPS_AGE_KEY_FILE="secrets/keys/age.txt"
sops --decrypt secret.encrypted.yml | kubectl apply -f -
```

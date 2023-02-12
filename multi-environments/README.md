# Use SOPS with multiple environments

The use-case here is that you have a repository with multiple file containing sensitive information for different environments (dev, staging, prod). You want to encrypt this file with SOPS and age. But we also want to keep everything in the same repository.

We want to have a different key pair for each environment. We will see how to use SOPS with multiple environments.

We assume you will execute the commands in this document in the `multi-environments` directory.

```shell
cd multi-environments
```

We'll suppose that you have already installed SOPS and age.

- [Use SOPS with multiple environments](#use-sops-with-multiple-environments)
  - [Example files](#example-files)
  - [Generate age keys](#generate-age-keys)
  - [Encrypt the file with SOPS](#encrypt-the-file-with-sops)
  - [Decrypt the file with SOPS](#decrypt-the-file-with-sops)
  - [Conclusion](#conclusion)

## Example files

We have three files:

- `dev.yaml`
- `staging.yaml`
- `prod.yaml`

## Generate age keys

We will simulate multiple users by creating multiple key pairs. We will create two key pairs:

```shell
# create directory to store the keys
mkdir -p secrets/keys
# generate dev key pair
age-keygen -o secrets/keys/dev.txt
# generate staging key pair
age-keygen -o secrets/keys/staging.txt
# generate prod key pair
age-keygen -o secrets/keys/prod.txt
```

## Encrypt the file with SOPS

Let's start with the `dev.yaml` file.

Create a file `.sops.yaml` with the following content:

```yaml
creation_rules:
  - path_regex: .*dev.*
    path_suffix: .enc
    encryption_method: age
    key_groups:
      - age:
          # dev public key
          - age1s5lx8prca0rd2ky2enaqqa7fek68xgg7fp9jmuz9zpppsr5czpgqjgf2q6

sops:
  encryption_method: age
```

You can see, we added a `path_regex` to match the `dev.yaml` file. We also added the `dev` public key in the `key_groups`.

```shell
sops --encrypt dev.yaml > dev.enc.yaml
```

Let's try to encrypt the `staging.yaml` file:

```shell
sops --encrypt staging.yaml > staging.enc.yaml
```

You will get an errror as we didn't add the `staging` public key in the `.sops.yaml` file.

Let's add rules for the `staging` and `prod` environments:

```yaml
creation_rules:
  - path_regex: .*dev.*
    path_suffix: .enc
    encryption_method: age
    key_groups:
      - age:
          # dev public key
          - age1s5lx8prca0rd2ky2enaqqa7fek68xgg7fp9jmuz9zpppsr5czpgqjgf2q6

  - path_regex: .*staging.*
    path_suffix: .enc
    encryption_method: age
    key_groups:
      - age:
          # staging public key
          - age198ce76jtw2hxw8j0z6v4h4as4kergha884xde3eaw27jlyd8x9vshu6zan

  - path_regex: .*prod.*
    path_suffix: .enc
    encryption_method: age
    key_groups:
      - age:
          # prod public key
          - age1jcr6svzuztxy0arhy82hn5du0z25wku7ytxlemxj8xz6nxkdr96qx2uec5

sops:
  encryption_method: age
```

Now we can encrypt all the files:

```shell
sops --encrypt dev.yaml > dev.enc.yaml
sops --encrypt staging.yaml > staging.enc.yaml
sops --encrypt prod.yaml > prod.enc.yaml
```

## Decrypt the file with SOPS

You can process as previously to decrypt the files.

## Conclusion

We saw how to use SOPS with multiple environments. We used the `path_regex` to match the files and the `key_groups` to add the public keys.

You can easily imagine that you can combine this with a multi-users environment by providing multiple public keys for each environment.

[⬅️ Previous: Use SOPS with multiple users](../multi-users/README.md)

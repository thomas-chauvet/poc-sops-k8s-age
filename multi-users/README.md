# Use SOPS with multiple users

The use-case here is that you have a repository with a file that contains sensitive information. You want to encrypt this file with SOPS and age. But we also want to keep everything in the same repository.

Here, we add the constraint to have multiple users with its own key pair. We will see how to use SOPS with multiple users.

We assume you will execute the commands in this document in the `multi-users` directory.

```shell
cd multi-users
```

We'll suppose that you have already installed SOPS and age.

- [Use SOPS with multiple users](#use-sops-with-multiple-users)
  - [Example file](#example-file)
  - [Generate age keys](#generate-age-keys)
  - [Encrypt the file with SOPS](#encrypt-the-file-with-sops)
  - [Decrypt the file with SOPS](#decrypt-the-file-with-sops)
  - [Remove a user](#remove-a-user)

## Example file

As previously, we create a file `example.yaml` with the following content:

```yaml
user: admin
password: mySuperSecretPassword
```

## Generate age keys

We will simulate multiple users by creating multiple key pairs. We will create two key pairs:

```shell
# create directory to store the keys
mkdir -p secrets/keys
# generate Alice key pair
age-keygen -o secrets/keys/alice.txt
# generate Bob key pair
age-keygen -o secrets/keys/bob.txt
```

## Encrypt the file with SOPS

Create a file `.sops.yaml` with the following content:

```yaml
creation_rules:
  - type: encrypted_file
    path_suffix: .enc
    encryption_method: age
    key_groups:
      - age:
          # Bob public key
          - age1d5mlv264rn0ds93s8jep5a6xpfcppg2jh5l8lnl4wt5dtpdlduds07mwa9
          # Alice public key
          - age14m7y2gxqtq3r72xfj9z0vrfywuvpt9cj6dfg93nktufsxwe0pf0sjtc25r

sops:
  encryption_method: age
```

We add the two public keys of Alice and Bob in the `key_groups` section.

Now, you can encrypt the file with SOPS:

```shell
sops --encrypt example.yaml > example.enc.yaml
```

In the encrypted file you can see that the `sops` section has been added and contains two recipients for Alice and Bob age public keys.

## Decrypt the file with SOPS

To decrypt the file, you need to have the private key of Alice or Bob. Let's try with Alice:

```shell
export SOPS_AGE_KEY_FILE="secrets/keys/alice.txt"
sops --decrypt example.enc.yaml
```

You can see that the file has been decrypted.

Now, let's try with Bob:

```shell
export SOPS_AGE_KEY_FILE="secrets/keys/bob.txt"
sops --decrypt example.enc.yaml
```

You can see that the file has been decrypted.

## Remove a user

Let's suppose that Alice has left the company. We want to remove her from the encrypted file.

First, we remove her public key from the `.sops.yaml` file:

```yaml
creation_rules:
  - type: encrypted_file
    path_suffix: .enc
    encryption_method: age
    key_groups:
      - age:
          # Bob public key
          - age14m7y2gxqtq3r72xfj9z0vrfywuvpt9cj6dfg93nktufsxwe0pf0sjtc25r
```

Then, we re-encrypt the file:

```shell
sops --encrypt example.yaml > example.enc.yaml
```

Now, we can see that the `sops` section contains only one recipient for Bob.

If we try to decrypt the file with Alice private key, we will have an error:

```shell
export SOPS_AGE_KEY_FILE="secrets/keys/alice.txt"
sops --decrypt example.enc.yaml
```

But if we try to decrypt the file with Bob private key, we will have no error:

```shell
export SOPS_AGE_KEY_FILE="secrets/keys/bob.txt"
sops --decrypt example.enc.yaml
```

[⬅️ Previous: first example](../first-example/README.md) | [🏠 Home](../../README.md) | [➡️ Next: Use SOPS with multiple environments](../multi-environments/README.md)

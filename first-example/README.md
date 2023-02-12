
# First example

The use-case here is that you have a repository with a file that contains sensitive information. You want to encrypt this file with SOPS and age. But we also want to keep everything in the same repository.

We assume you will execute the commands in this document in the `first-example` directory.

```shell
cd first-example
```

We'll suppose that you have already installed SOPS and age.

- [First example](#first-example)
  - [Example file](#example-file)
  - [Generate age key](#generate-age-key)
  - [Encrypt the file with SOPS](#encrypt-the-file-with-sops)
    - [Pass public key in command line](#pass-public-key-in-command-line)
    - [Pass public key in environment variable](#pass-public-key-in-environment-variable)
    - [Use a configuration file](#use-a-configuration-file)
  - [Decrypt the file with SOPS](#decrypt-the-file-with-sops)
  - [Edit file with SOPS](#edit-file-with-sops)

## Example file

We create a file `example.yaml` with the following content:

```yaml
user: admin
password: mySuperSecretPassword
```

## Generate age key

We want to encrypt this file with SOPS and age. First, we need to create a key pair with age:

```shell
# create directory to store the keys
mkdir -p secrets/keys
# generate a key pair
age-keygen -o secrets/keys/my-first-age-key.txt
```

You can inspect the content of the file `keys/my-first-age-key.txt`:

```shell
cat secrets/keys/my-first-age-key.txt
```

It will look like this:

```txt
# created: YYYY-MM-DDTHH:MM:SS+XX:YY
# public key: ageyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
AGE-SECRET-KEY-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

We are interested in the public key, which is the first line of the file. We will use it to encrypt the file with SOPS.

## Encrypt the file with SOPS

### Pass public key in command line

Now, we can encrypt the file with SOPS:

```shell
sops --age <public-key> --encrypt example.yaml > example.enc.yaml
```

*Note: we add `.enc` to filename because it is the encrypted version of the file. You should keep the same file extension to be able to open the file in your favorite editor.*

You can inspect the content of the file `example.enc.yaml`:

```shell
cat example.enc.yaml
```

You can note that SOPS adds a key `sops` to the file. This key contains information about the encryption method used and other information about age.

One the really BIG advantage of SOPS is that the **file's structure is kept**, we can see what are the keys in the original file while keeping the value encrypted and unreadable.

### Pass public key in environment variable

It is not really handy to copy/paste the public key in the command line. You can also use an environment variable:

```shell
SOPS_AGE_RECIPIENTS=<age-public-key>
# OR simply get from the file
SOPS_AGE_RECIPIENTS=$(cat secrets/keys/my-first-age-key.txt | grep public | cut -d: -f2 | tr -d ' ')
# encrypt the file
sops --age $SOPS_AGE_RECIPIENTS --encrypt example.yaml > example.enc.yaml
```

### Use a configuration file

You can also add a configuration file that you can store in your repository.

Create a file `.sops.yaml` with the following content:

```yaml
creation_rules:
  - type: encrypted_file
    path_suffix: .enc
    encryption_method: age
    age: age1q4nrgfr96alwml6ph7eywn9rma987f7patastz7p0wtwl05ctcxsm9mfl9

sops:
  encryption_method: age
```

You can note that we add a `creation_rules` key. This key allows you to define rules to encrypt files. In this example, we want to encrypt all files with the extension `.enc` with the age encryption method.

You can also note that we add a `sops` key. This key allows you to define the default encryption method. In this example, we want to encrypt all files with the age encryption method.

Now, you can encrypt the file with SOPS:

```shell
sops --encrypt example.yaml > example.enc.yaml
```

## Decrypt the file with SOPS

Now, we can decrypt the file with SOPS:

```shell
export SOPS_AGE_KEY_FILE="secrets/keys/my-first-age-key.txt"
sops --decrypt example.enc.yaml
```

## Edit file with SOPS

You can edit the file with SOPS:

```shell
export SOPS_AGE_KEY_FILE="secrets/keys/my-first-age-key.txt"
sops example.enc.yaml
```

It will open the file in your favorite editor. You can edit the file and save it. SOPS will encrypt the file automatically.

[⬅️ Next: Use SOPS with multiple users](../multi-users/README.md)

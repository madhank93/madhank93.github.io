+++
title = "Securely storing secrets — Part I"
description = "Client-side secrets management using SOPS"
date = 2024-06-16T00:00:00+00:00

[taxonomies]
tags = ["security", "encryption", "secrets-management"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/2000/1*xT5ZGS9vgmDehqKTuNEzRQ.png)

Secrets (passwords, tokens, keys, …etc) are critical components in software and infrastructure that provide access to sensitive data and services. So the next crucial part is securing it, secrets can be stored on the ***Client*** — on the device running the application (e.g. hardcoded passwords in your codebase) or on the ***Server*** — on the centralized application retrieving over an API call (e.g. fetching from [AWS secrets manager](https://aws.amazon.com/secrets-manager/) or [Infisical](https://infisical.com/) or [Hashicorp vault](https://www.vaultproject.io/)) or using [OpenID Connect](https://www.microsoft.com/en-us/security/business/security-101/what-is-openid-connect-oidc) (OIDC) to issue short-lived access tokens instead of managing passwords. All approaches have their pros and cons.

In this blog post, we will take a look at how to store secrets on the client side securely — git using [SOPS](https://getsops.io/) (***S***ecrets ***OP***eration***S***), a CLI tool used for encrypting, and decrypting the files with [AWS KMS](https://aws.amazon.com/kms/) (supports [GCP KMS](https://cloud.google.com/security/products/security-key-management?hl=en*), [Age](https://github.com/FiloSottile/age), [Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault) ..etc).

To get started install [SOPS binary](https://github.com/getsops/sops/releases) from GitHub, complete [AWS configuration](https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-configure.html), and [KMS key creation](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html).

### How does it work?

The sops command takes a file to encrypt it along with the encryption keys and produces a decrypted file. By default, it encryps only the values leaving the keys as it is for easy readability.

![Sops flow](https://cdn-images-1.medium.com/max/2000/1*Iq-RnLSlHNDTL0jKZbQZsw.png)

SOPS uses AWS SDK to communicate with AWS KMS. It makes use of the credentials configured from `~/.aws/credentials`.

### SOPS configuration

Configurations related to SOPS can be maintained in `.sops.yaml` file. Different sets of keys can be used in various environments to encrypt files and what files to encrypt or decrypt. And also its supports `encrypted_regex` option will only encrypt values under keys that match the supplied regular expression.

```yml
# creation rules are evaluated sequentially, the first match will be evaluated
creation_rules:
    # dev encryption
    - path_regex: \.dev\.(yml|yaml)$
    encrypted_regex: "^(.*[Pp]ass.*|.*[Pp]wd.*|.*[Tt]oken.*)$"
    kms: "arn:aws:kms:us-east-1:654654270281:key/0be9b8e7-ada8-45b3-84d8-075d66359205"

    # prod encryption
    - path_regex: \.prod\.(yml|yaml)$
    encrypted_regex: "^(data|stringData)$"
    key_groups:
        - kms:
            - arn: "arn:aws:kms:us-west-2:361527076523:key/5052f06a-5d3f-489e-b86c-57201e06f31e+arn:aws:iam::361527076523:role/hiera-sops-prod,arn:aws:kms:eu-central-1:361527076523:key/cb1fab90-8d17-42a1-a9d8-334968904f94+arn:aws:iam::361527076523:role/hiera-sops-prod"
            role: "arn:aws:iam::927034868273:role/sops-dev-xyz"
            aws_profile: foo
```

### SOPS as an editor

To get started run sops new_test.dev.yaml, opens up the text editor (whatever $EDITOR is set to, or, if it's not vim is the default) which handles encryption/decryption. Delete the preconfigured ones, and add the key value (e.g. password: secret!!!) to be encrypted, save & exit the editor. As soon as we exit the editor, a new file named new_test.dev.yaml created with encryption and SOPS metadata.

```yml
password: ENC[AES256_GCM,data:b08sONBNBtZO,iv:1/8nRdqE7r48PQ/9npyO8em4hP99crKyyZp8iAC2icQ=,tag:5XcTNEyB5HElpr9kacAFtg==,type:str]
sops:
    kms:
    - arn: arn:aws:kms:us-east-1:654654270281:key/0be9b8e7-ada8-45b3-84d8-075d66359205
        created_at: "2024-06-17T02:47:48Z"
        enc: AQICAHiCvIqgTXzQ2oe26RUr4Aml9dfI/ipc8iysDVNaOdrQ2AEDLJnvLUWTUB2o+gP4VdJ7AAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMqFvk+2zi/80QdnYgAgEQgDs8LszoAgJmmNWoZLGJSchLZ7dIi2DV3xvJvVwT6lP+a38BP5H7eeTbGyayeSyuBPsKwu5nqP6HElBo2g==
        aws_profile: ""
    gcp_kms: []
    azure_kv: []
    hc_vault: []
    age: []
    lastmodified: "2024-06-17T02:48:54Z"
    mac: ENC[AES256_GCM,data:Vv07a3WKh15AtMM5vpr3giT7TDPdz6P2TliZNcGBG2mlTet0LvSHp0Kum55bAtyES6snMWYFisW22mQ2znpcYaPb4g/lR7b10fLOVRf0IoDRz00QKmyIa2YAev5EMLwmKL1+lstogiL4VDlxtEzH1FfgQsiqYGe775RSYxxvhfI=,iv:pDct2OW+VM4Hi5OAxDIVw0yaKVYIEiZF5txj+SJGG+k=,tag:Nur3BmsL+P47XxW5ltaWig==,type:str]
    pgp: []
    encrypted_regex: ^(.*[Pp]ass.*|.*[Pp]wd.*|.*[Tt]oken.*)$
    version: 3.8.1
```

### Encrypting and decrypting an existing file

To encrypt the existing file, run `sops -e -i db-secrets.dev.yml` and write the output back to the same file. To decrypt the file, run `sops -d -i db-secrets.dev.yml`

```yml
key: value
mongo:
    username: admin
    password: ENC[AES256_GCM,data:ooShx+mdmdSiewDwPA8PlHFU,iv:gGcIHgqJkxEZqGgTymvbTX+uQrIzjUcIOWV+OCw6vAM=,tag:igUY1HZAZK1xk5YMZp+tEQ==,type:str]
aws:
    us-east:
        token: ENC[AES256_GCM,data:1MtcUMcY2WyfCk14THO510fldbsbYtlpwsC0LKMC/g/zIyaP3L9H4DEMSgXin1LTtpx+tM3TKcEhV9cYWxf9dQ==,iv:cdz4YPQTopxt57ykPIxLm2kv/2/f8bY6JWdPnb296lU=,tag:vzkWMHpmA04QeHsZMF0uGw==,type:str]
sops:
    kms:
        - arn: arn:aws:kms:us-east-1:654654270281:key/0be9b8e7-ada8-45b3-84d8-075d66359205
            created_at: "2024-06-17T01:55:38Z"
            enc: AQICAHiCvIqgTXzQ2oe26RUr4Aml9dfI/ipc8iysDVNaOdrQ2AGUFtQshsDeNaln7oQh2ralAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMKZqrEGi/JQ1iLpk+AgEQgDtCuGzeWJLtcU1vdXOGmtXbOaOTPyzHK4Rf99NohCOL4iGBUZm5G4L0Sv5agA70Qlx/f37ev9SpHDQg9Q==
            aws_profile: ""
    gcp_kms: []
    azure_kv: []
    hc_vault: []
    age: []
    lastmodified: "2024-06-17T01:55:38Z"
    mac: ENC[AES256_GCM,data:N6KNibwZ3nNY6X7U0zpScaUiePbAlEUXbhMfe6T5a1WAtfK/tgDMcUdEcZmHKmle6XJ5Asf9Lt/+vEFOvtnqPSKjZwIEQhB7N3F8vRcQ6VvwyORqN1gx9F8ew3xPFoIPTh9k77e0+Z9hCxQUiamekXRB+G/0u5diafV5ZojZeIk=,iv:W6Q5Rn5FEjzGk1+c1GbQo9dDOoGtw9FT92Dnc+8djis=,tag:eK7YdE9fhN3+YT2D0MERvg==,type:str]
    pgp: []
    encrypted_regex: ^(.*[Pp]ass.*|.*[Pp]wd.*|.*[Tt]oken.*)$
    version: 3.8.1Rotating keys
```

### Key Rotation

As a good practice, it is always recommended to rotate the key and re-encrypt the file with a new key using the command sops roatet -i filename.yml after updating the .sops.yaml file with a new key.

### Use cases

* Config management for applications

* Managing [Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/) and [helm](https://github.com/jkroepke/helm-secrets)

* Encrypting Terraform/Pulumi state files

### Other features

* [Audit](https://github.com/getsops/sops?tab=readme-ov-file#auditing)— keeps track of who accessed what files in PostgreSQL

* [Publish](https://github.com/getsops/sops?tab=readme-ov-file#using-the-publish-command)— publishes a file to a pre-configured destination like s3


<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/securely-storing-secrets-part-i-f1b5f3b56d49)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/learn-securing-secrets](https://github.com/madhank93/learn-securing-secrets/tree/main/sops-part-1)

</center>

**Reference**:

* [https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

* [https://getsops.io/](https://getsops.io/)

* [https://github.com/jkroepke/helm-secrets/wiki/Usage](https://github.com/jkroepke/helm-secrets/wiki/Usage)

* [https://www.youtube.com/playlist?list=PLEAHWp_qqCrKRQXPcnfEd4Cim1cL6MTjT](https://www.youtube.com/playlist?list=PLEAHWp_qqCrKRQXPcnfEd4Cim1cL6MTjT)

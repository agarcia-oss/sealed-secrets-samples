# Sealed Secrets Samples

A collection of different Sealed Secrets uses cases and examples

## Table of Contents

- [Simple Sealed Secrets Example ](#simple_sealed_secrets_example)
- [Managing Existing Secrets](#managing_existing_secrets)

## Simple Sealed Secrets Example

The most basic example of Sealed Secrets, just encrypt a Kubernetes Secrets manifest and apply the resulting Sealed Secrets into your cluster:

```yaml
$ kubeseal -f simple-secret.yaml | kubectl apply -f -
sealedsecret.bitnami.com/simple-secret created
$ kubectl get sealedsecrets.bitnami.com simple-secret -o yaml

apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: simple-secret
  namespace: default
spec:
  encryptedData:
    password: AgC+fJDLQWeIBsW5+XE72bb+ea2Pnoeji/CDcSQls9t+ioFSmIWtdhZ+htTHpTM3tcsfeRDlDQdG4e1QKuS93d0kaxpJnnWUubK76/7DbXa+9UoXKQE5h9IakDyLQXw6E6MaWbHbkJ67RG8Cts6tfyuqXGdvJQI0BNDB1NoFNafWDdRghFtUIoIggoiThTDaGwvQwqKcjOsmoDpY8oeY3TL8I3AqnMN8vBG4P85guNZaZuqtHb9uX2GCBPtmcfnPj0pk+oUK8jX5SdqI7R6UuweH4u+PTIkzi24ar2BySLYZXxelxfE61jMwr9nsrL0g6adYTgemWbBQMhQgT4l1gnqFYMm+VqCevTrC4Adk8krDF0/Veu0m474j54qA/ndzT6a4vvqSToGaqUze97TlZbjffh4a+TYEb5tZZ25rllyj/ThDCrBLq4012/YIgYegpYw19PI4keYRQwDzqSQK6BYEveEAYnF2TF8J3FW0HvOuK1LzpUvks0p59wHTZ17J/+00P1hUgu1JgnYFPmVTOHtruWibQRtp3Ogj8M+mkD8AczkGQT+qRo+xQU1jCpS0VFb4G3VH8qnCQ1/v1ws1Y9FqCRXlHdwVjDUxK4+5I0c+gQSsa/RzZYxHvgLed1W6YcAbVkaWOsExg3eXmygvcU4sX8ZnNq/jwsU9vdVX8VQu9HGb9POZwBOCRpFVmj6Hz/H25Vh3bO6XLMJhOzl4y4MD
  template:
    metadata:
      creationTimestamp: null
      name: simple-secret
      namespace: default
status:
  conditions:
  - lastTransitionTime: "2024-04-05T14:28:57Z"
    lastUpdateTime: "2024-04-05T14:28:57Z"
    status: "True"
    type: Synced
  observedGeneration: 1
```
A new associated K8S Secrets will be created in the same namespace. This Secret will be fully managed by the Sealed Secrets controller and deleted in case you decide to remove the Sealed Secrets object from your cluster.

## Managing Existing Secrets

Now, what happens if you want to operate with Secrets that already exist in your cluster? This is also a very popular use case since it will be used when you start to adopt Sealed Secrets on an already existing cluster.

```yaml
$ kubectl get secrets managed-secret -o yaml
apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  annotations:
    sealedsecrets.bitnami.com/managed: "true"
  name: managed-secret
  namespace: default
type: Opaque

$ kubectl apply -f managed-sealed-secret.yaml

$ kubectl get secrets managed-secret -o yaml
apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  annotations:
    sealedsecrets.bitnami.com/managed: "true"
  name: managed-secret
  namespace: default
  ownerReferences:
  - apiVersion: bitnami.com/v1alpha1
    controller: true
    kind: SealedSecret
    name: managed-secret
type: Opaque
```
## Managing the Sealed Secrets sealing keys

Sealed Secrets sealing keys can be automatically managed by the Sealed Secrets controller. This is the recommended option for most users since, by default, the controller deals with all the aspects of renewing the key, managing key history, etc.

However, here you'll find a few samples on how to manage the sealing key directly.

```shell
$ kubeseal --fetch-cert | tee cert.pem
-----BEGIN CERTIFICATE-----
TLS Certificate...
-----END CERTIFICATE-----
```

```fetch-cert``` retrieves the encryption public key. Users without direct access to the cluster can use the following method to encrypt secrets locally:

```yaml
kubeseal --cert cert.pem -f simple-secret2.yaml
kubeseal --cert cert.pem -f simple-secret2.yaml -o yaml
---
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: simple-secret2
  namespace: default
spec:
  encryptedData:
    key1: AgBZ3ul0C9qtfs4Yhy/CJQyu1fHZwO/AYnLXWFL3xORwUKKbIxTF/pm2iIAciomKa9cd7ixtphWi1cRHLTH0re4eT2dDwrhKBP3v0sZlXuFlanYzhkGpE3XtEBNfSkvMk9p5Z/akaE++1gp/0r37Shn9WyX4KfkFURcS3zp3XNwW2tjiaPJyNkOiczIVbl5VCMYsI2GduoY8ybbV/3tHxLn2md9BTrRT9IMJ7wRhl8Bg70K5JOTh7Ar8lqKcUTTXvTdk5TYuAVFoRO4ePlYBRua8z2QfkK7hPeWNtnuzFXDx1s/mbPjXIqOyqs4exXyowP8Ehq0F1g/OXeKQai4HfMg4rOBQU1fhOjVWHXXW62dmnOsZ1qUr1CzB8V8jXyncayejzOYm8wZwf6BZTYYmuhjSE8tg0knCPo3EGKUeuVAEn7D9N4kJU04KqsjTsubtjgvRjC5jjuyiiclK46O7yUhDbF5fZ3yDyH+kxz0atgw3kdQq39Wi2iK7qblrbwH/GPNbKOUMwZN3YLlr6ajfUClc41jP/q8DEMTyPqum8KxzbkHWN/qvDsEvWmPGK629vDENOeJMqK8NIMGv32VMve5jZ3js0BpgH7beQ0DOCbFFiEsShXVBRa6uHW8hEk8r6CaZ/BKJvEZLeH20xF/6H6OtLxzjpsg4TZoUvvL87zYlXswESQARGjWI5WgvM7Ettjhew8/E/1paFOQTyw==
  template:
    metadata:
      annotations:
        sealedsecrets.bitnami.com/managed: "true"
      creationTimestamp: null
      name: simple-secret2
      namespace: default
```
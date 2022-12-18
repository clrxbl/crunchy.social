### p2w.social

A Mastodon instance available on https://p2w.social

### Flux

Before installing [Flux2](https://fluxcd.io), secret decryption has to be set up. [age](https://github.com/FiloSottile/age) is used to encrypt secrets.

Create the `flux-system` namespace and create the secret containing the age private key that can decrypt the secrets in this repository using the following commands:

```bash
kubectl create namespace flux-system

cat age.agekey |
kubectl create secret generic sops-age \
--namespace=flux-system \
--from-file=age.agekey=/dev/stdin
```

Bootstrap Flux2 afterwards with

```bash
flux bootstrap git --url=ssh://git@github.com/clrxbl/crunchy.social --branch=main --ssh-key-algorithm ed25519 --path=base
```

## Secrets

Ensure that you have the `sops` CLI utility installed.
Aslong as your current working directory is somewhere in this repository, sops should pick up on the `.sops.yaml` file and be able to encrypt Kubernetes secrets with the public key we use.

Example scenario for encrypting a new secret:

```bash
# Create an example secret.yaml file
cat <<EOF > ./secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: discord-webhook
  namespace: flux-system
stringData:
  address: https://discord.com/webhook-url-here
type: Opaque
EOF

# Encrypt the stringData in-place
sops --encrypt --in-place secret.yaml
```

For the secrets in base/flux-config/secrets use
`sops --decrypt --input-type=dotenv --output-type=dotenv cluster-secrets.env.encrypted > cluster-secrets.env`
and
`sops --encrypt --input-type=env cluster-secrets.env > cluster-secrets.env.encrypted && rm cluster-secrets.env`

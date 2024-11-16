# Bluesky PDS Helm Chart

A helm chart for Bluesky PDS (Personal Data Server)

## Install

Secrets must be created during install.

REQUIRED:

```shell
APPNAME="my-bsky"
NAMESPACE="my-bsky"
PDS_HOSTNAME="bsky.example.com"

PDS_ADMIN_PASSWORD=$(openssl rand --hex 16)
PDS_JWT_SECRET=$(openssl rand --hex 16)
PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX=$(openssl ecparam --name secp256k1 --genkey --noout --outform DER | tail --bytes=+8 | head --bytes=32 | xxd --plain --cols 32)

kubectl create secret generic --namespace ${NAMESPACE} ${APPNAME}-pds \
    --from-literal=adminPassword=${PDS_ADMIN_PASSWORD} \
    --from-literal=jwtSecret=${PDS_JWT_SECRET} \
    --from-literal=plcRotationKey=${PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX}

helm upgrade ${APPNAME} bluesky-pds \
  --install \
  --namespace ${NAMESPACE} \
  --create-namespace \
  --set hostname=${PDS_HOSTNAME} \
  --set pds.secretsName=${APPNAME}-pds
```

## Usage

### pdsadmin

A sidecar container with `pdsadmin` is optionally available (enabled by
default).

Use it as shown in the example below:

```shell
$ kubectl exec -it -n ${NAMESPACE} deploy/${APPNAME}-pds -c pdsadmin -- /bin/bash -l
utils@my-bsky-pds-86bfc97b8c-wt7xl:~$ pdsadmin account create
Enter an email address (e.g. alice@bsky.example.com):
Enter a handle (e.g. alice.bsky.example.com):
```

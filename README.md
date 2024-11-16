# Bluesky PDS Helm Chart

A helm chart for Bluesky PDS (Personal Data Server)

## Install or Upgrade

Secrets must be managed during install/upgrade.

REQUIRED:

```
APPNAME="my-bsky"
NAMESPACE="my-bsky"
PDS_HOSTNAME="bsky.example.com"
PDS_ADMIN_PASSWORD=$(kubectl get secret --namespace "$NAMESPACE" $APPNAME-secrets -o jsonpath="{.data.adminPassword}" 2>/dev/null | base64 -d)
PDS_JWT_SECRET=$(kubectl get secret --namespace "$NAMESPACE" $APPNAME-secrets -o jsonpath="{.data.jwtSecret}" 2>/dev/null | base64 -d)
PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX=$(kubectl get secret --namespace "$NAMESPACE" $APPNAME-secrets -o jsonpath="{.data.plcRotationKey}" 2>/dev/null | base64 -d)

if [ -z "$PDS_ADMIN_PASSWORD" ]; then
    export PDS_ADMIN_PASSWORD=$(openssl rand --hex 16)
fi
if [ -z "$PDS_JWT_SECRET" ]; then
    export PDS_JWT_SECRET=$(openssl rand --hex 16)
fi
if [ -z "$PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX" ]; then
    export PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX=$(openssl ecparam --name secp256k1 --genkey --noout --outform DER | tail --bytes=+8 | head --bytes=32 | xxd --plain --cols 32)
fi

helm upgrade ${APPNAME} bluesky-pds \
  --install \
  --namespace ${NAMESPACE} \
  --create-namespace \
  --set hostname=${PDS_HOSTNAME} \
  --set secrets.adminPassword=${PDS_ADMIN_PASSWORD} \
  --set secrets.jwtSecret=${PDS_JWT_SECRET} \
  --set secrets.plcRotationKey=${PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX}
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

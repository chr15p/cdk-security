
> openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
>
> where CN=registry.whatever.com

Create a secret with the resulting certs

> kubectl create secret tls registry-tls-data --key key.pem --cert cert.pem

Then set up the registry

>kubectl apply -f registry.yaml

and set the registry to be insecure

> juju config kubernetes-worker docker-opts="--insecure-registry registry.54.164.115.34.xip.io"

We still need to add storage....


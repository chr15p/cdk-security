
##> openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
>
> mkdir certs
> openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
>
> where CN=registry.whatever.com

Then add these certs into the yaml file in base64 encoded form

> sed "s/DOMAIN_CERT/`cat certs/domain.crt|base64 -w0`/g" secret.yml.tmpl | \
> 
> sed "s/DOMAIN_KEY/`cat certs/domain.key|base64 -w0`/g" > registry-tls.yaml

Then simply apply the config

> kubectl apply -f registry-tls.yaml

Alternativly you can create the secret by hand:

> kubectl -n default create secret generic registry-tls-data --from-file=tls.crt=certs/domain.crt  --from-file=tls.key=certs/domain.key

and just remove that from the yaml.


Now the registry should be listening and ready for use on port 35000


On AWS you will need to open the port for all workers

> juju run  --unit kubernetes-worker/0 "open-port 35000"


but then you should be able to tag and push an image

> docker tag busybox  registry.54.164.115.34.xip.io:35000/busybox
> docker push  registry.54.164.115.34.xip.io:35000/busybox


We still need to add storage....



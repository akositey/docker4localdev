# Docker4LocalDev
Linux Docker Setup for Local Development
```
https://{YOUR_APP_1}.docker.localdev
https://{YOUR_APP_2}.docker.localdev
```

## Tested On
I've tried it on Ubuntu 20.04, which uses NetworkManager by default.
It has a `dnsmasq` plugin so we don't need to install a separate package. 
I think it's safe to assume that you can apply the same principle to any Unix system.

## Available domains

* Traefik Dashboard: https://docker.localdev
* Portainer: https://portainer.docker.localdev
* Whoami: https://whoami.docker.localdev

## Getting Started

### Generate your own _locally-trusted_ certificate.

Install [Mkcert](https://github.com/FiloSottile/mkcert) on your system.
You can just download from the [releases](https://github.com/FiloSottile/mkcert/releases)
Then move it to your `/usr/local/bin/`
Once installed, run `mkcert -install`.
Then generate your self-signed certificates:

`mkcert -key-file ./certs/key.pem -cert-file ./certs/cert.pem localdev 'docker.localdev' '*.docker.localdev'`

Move the generated certificates (`cert.pem`, `key.pem`) to the folder `certs/`

### Configure `dnsmasq` to resolve localdev

Configure NetworkManager to use `dnsmasq` plugin:
`sudo nano /etc/NetworkManager/NetworkManager.conf`
```
[main]
plugins=ifupdown,keyfile
dns=dnsmasq # modify this line to use dnsmasq plugin
```
Tell dnsmasq to resolve localdev as 127.0.0.1
`sudo nano /etc/NetworkManager/dnsmasq.d/localdev.conf`
```
address=/.localdev/127.0.0.1
```
Reload Network
`sudo systemctl reload NetworkManager`

### Configure your apps to use `traefik`
Ex. using docker-compose
```
    # websecure = defined in docker-compose.yml, under our traefik service
    # traefik = defined in docker-compose.yml, under networks
    nginx:
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.my-app_web.entrypoints=websecure"
            - "traefik.http.routers.my-app_web.rule=Host(`my-app.docker.localdev`)"
            - "traefik.http.routers.my-app_web.tls=true"
            - "traefik.docker.network=traefik"
```
after you reload your containers, you should be able to access it. In this example: https://my-app.docker.localdev

## Credits
This based on this article on [Medium](https://medium.com/soulweb-academy/docker-local-dev-stack-with-traefik-https-dnsmasq-locally-trusted-certificate-for-ubuntu-20-04-5f036c9af83d). Please visit it if you need more explanations. This is just a modified version of that tutorial.

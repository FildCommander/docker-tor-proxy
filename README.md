# docker-tor-proxy
The Container provides a SOCKS5 Proxy via Port 9050 into the Tor Network.
You can connect your Browser or OS or other Container to the Proxy.


## Usage
### docker
```bash
$ docker run --rm -p 9050:9050 ghcr.io/fildcommander/docker-tor-proxy:main
```

After that you can connect via SOCKS5 to the container by providing *127.0.0.1:9050* or the IP of your host.

```bash
$ curl --socks5 127.0.0.1:9050 https://check.torproject.org/api/ip
```

### docker compose
Create a docker-compose.yaml file with following content:
```yaml
services:
  tor-proxy:
    container_name: tor-proxy
    ports:
      - 9050:9050
    image: ghcr.io/fildcommander/docker-tor-proxy:main
```

After that start the container with:
```bash
$ docker compose up -d 
```

Now you can connect to the socks proxy.

## Connect container to proxy
To connect a container to the proxy there are some additional steps required.
For some Images you will need to install additional packages, because many are not build with socks proxy support in mind.

### Create unique network
First create a unique network for the proxy.
```bash
$ docker network create --subnet 10.28.0.0/24 tor
```

The name *tor* and the *subnet* can be freely chosen. You should use a clearly identifiable name and an not already used subnet.

### Create proxy container
Next we create the container via docker compose:
```yaml
services:
  tor-proxy:
    container_name: tor-proxy
    image: ghcr.io/fildcommander/docker-tor-proxy:main
    networks:
      tor:
        ipv4_address: 10.28.0.10

networks:
  tor:
    external: true
```

Keep in mind to use **your** name for the network and an IP address from **your** chosen subnet.
You should note the static IP address. We sill need it for every container you will connect.

### Connect container
This step must be repeated for every container you want to connect to Tor.
As an example I will use the [alpine/curl](https://hub.docker.com/r/alpine/curl) Image.

For docker cli:
```bash
$ docker run --rm -e ALL_PROXY=socks5h://10.28.0.10:9050 --network=tor alpine/curl https://check.torproject.org/api/ip
```

For docker compose:
```yaml
services:
  image: alpine/curl
  command: https://check.torproject.org/api/ip
  environments:
    ALL_PROXY: "socks5h://10.28.0.10:9050"
  networks:
    - tor

networks:
  tor:
    external: true
```

```bash
$ docker compose up
```

In both cases will the container go through the Tor Network.

If your docker compose container uses multiple networks, than make sure you have the *tor* network at the first position. The other should follow after that. This includes also the top level networks declaration.

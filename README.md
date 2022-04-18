# Registry Cluster

### Step by step

1. Generate the keys
```shell
openssl req -newkey rsa:4096 -nodes -sha256 -keyout ./certs/domain.key -addext "subjectAltName = DNS:localregistry.com" -x509 -days 365 -out ./certs/domain.crt
```
2. Create local registry
```shell
docker run -d --restart=always --name "localregistry.com" -v $PWD/certs:/certs -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=./certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=./certs/domain.key -p 443:443 registry:2
```
3. See the step-by-step to tag

4. Build the images for spire-server and spire-agent
```shell
docker build -f ./docker/spire-agent/Dockerfile --tag localregistry.com/spire-agent .
docker build -f ./docker/spire-server/Dockerfile --tag localregistry.com/spire-server .
```
5. Build two signed images and two unsigned images from spire-agent
```shell
docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image1-signed .
docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image2-signed .
docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image1-unsigned .
docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image2-unsigned .
```
6. Copy certificates locally
```shell
sudo cp ./certs/domain.crt ./certs/domain.key /usr/local/share/ca-certificates/
sudo update-ca-certificates --fresh
```
7. Sign the images
```shell
export COSIGN_EXPERIMENTAL=1
cosign sign [image]
```
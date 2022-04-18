0 - Gerar as chaves
	- openssl req -newkey rsa:4096 -nodes -sha256 -keyout ./certs/domain.key -addext "subjectAltName = DNS:localregistry.com" -x509 -days 365 -out ./certs/domain.crt

1 - Criar local registry
	- docker run -d --restart=always --name "localregistry.com" -v $PWD/certs:/certs -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=./certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=./certs/domain.key -p 443:443 registry:2

2 - Ver o passo a passo para colocar as tags

3 - Buildar as imagens do spire-server e do spire-agent
	- docker build -f ./docker/spire-agent/Dockerfile --tag localregistry.com/spire-agent .
	- docker build -f ./docker/spire-server/Dockerfile --tag localregistry.com/spire-server .

4 - Buildar duas imagens assinadas e duas não assinadas a partir do spire-agent
	- docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image1-signed .
	- docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image2-signed .

	- docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image1-unsigned .
	- docker build -f ./docker/workload/Dockerfile --tag localregistry.com/image2-unsigned .

5 - Copiar certificados
    - sudo cp ./certs/domain.crt ./certs/domain.key /usr/local/share/ca-certificates/
    - sudo update-ca-certificates --fresh

6 - Assinar as imagens
    - export COSIGN_EXPERIMENTAL=1
    - cosign sign [image]
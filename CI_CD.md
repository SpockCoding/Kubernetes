O exemplo de código do .gitlab-ci.yaml a seguir é um passo da pipeline no git que faz um push de sua aplicação no dockerhub empacotando tudo, assim, é possível no kubernetes, docker swarn, docker, fazer um pull dessa imagem de sua aplicação.
Toda ver que houver um commit no repositório o Git irá executar essa pipeline fazendo esse push dessa aplicação docker.
### .gitlab-ci.yml 

stages:
  - build

build_images:
  stage: build
  image: docker:20.10.16

  services:
    - docker:20.10.16-dind
  
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  
  before_script:
    - docker login -u  $REGISTRY_USER -p $REGISTRY_PASS

  script:
    - docker build -t spockiscoding/jarjarbinx:1.0 app/.
    - docker push spockiscoding/jarjarbinx:1.0
### dockerfile 

FROM httpd:latest

WORKDIR /usr/local/apache2/htdocs/

COPY index.html /usr/local/apache2/htdocs/

EXPOSE 80
### Configuração da Pipeline no Gitlab

![image](https://user-images.githubusercontent.com/97816800/210677199-ea1ce211-69c3-4be8-a2b1-09b01e67b47a.png)

### No DockerHub o resultado do job da pipeline

![image](https://user-images.githubusercontent.com/97816800/210677310-d7edf94b-9a1f-4fc6-8591-999c75b6feaf.png)

### Abaixo a mostra do Job da Pipeline que criamos 

```
Preparing environment
00:00
Running on runner-zxwgkjap-project-42312902-concurrent-0 via runner-zxwgkjap-shared-1672881379-14f383f8...
Getting source from Git repository
00:01
$ eval "$CI_PRE_CLONE_SCRIPT"
Fetching changes with git depth set to 20...
Initialized empty Git repository in /builds/valdir.cruz/jarjarbinx/.git/
Created fresh repository.
Checking out 6326a2f8 as main...
Skipping Git submodules setup
Executing "step_script" stage of the job script
00:11
Using docker image sha256:2a153cb5c7c52610ceb46876231f1c7ab8d2a0926aaeb5283994ef3d6f78def9 for docker:20.10.16 with digest docker@sha256:5bc07a93c9b28e57a58d57fbcf437d1551ff80ae33b4274fb60a1ade2d6c9da4 ...
$ docker login -u  $REGISTRY_USER -p $REGISTRY_PASS
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
$ docker build -t spockiscoding/jarjarbinx:1.0 app/.
Step 1/4 : FROM httpd:latest
latest: Pulling from library/httpd
3f4ca61aafcd: Pulling fs layer
2e3d233b6299: Pulling fs layer
6d859023da80: Pulling fs layer
f856a04699cc: Pulling fs layer
ec3bbe99d2b1: Pulling fs layer
f856a04699cc: Waiting
ec3bbe99d2b1: Waiting
2e3d233b6299: Verifying Checksum
2e3d233b6299: Download complete
6d859023da80: Verifying Checksum
6d859023da80: Download complete
3f4ca61aafcd: Verifying Checksum
3f4ca61aafcd: Download complete
ec3bbe99d2b1: Verifying Checksum
ec3bbe99d2b1: Download complete
f856a04699cc: Verifying Checksum
f856a04699cc: Download complete
3f4ca61aafcd: Pull complete
2e3d233b6299: Pull complete
6d859023da80: Pull complete
f856a04699cc: Pull complete
ec3bbe99d2b1: Pull complete
Digest: sha256:f8c7bdfa89fb4448c95856c6145359f67dd447134018247609e7a23e5c5ec03a
Status: Downloaded newer image for httpd:latest
 ---> 73c10eb9266e
Step 2/4 : WORKDIR /usr/local/apache2/htdocs/
 ---> Running in 3f98eaa5a291
Removing intermediate container 3f98eaa5a291
 ---> d404462c00d3
Step 3/4 : COPY index.html /usr/local/apache2/htdocs/
 ---> 166ff7d3e3eb
Step 4/4 : EXPOSE 80
 ---> Running in 98924bfad340
Removing intermediate container 98924bfad340
 ---> faf50f56d56f
Successfully built faf50f56d56f
Successfully tagged spockiscoding/jarjarbinx:1.0
$ docker push spockiscoding/jarjarbinx:1.0
The push refers to repository [docker.io/spockiscoding/jarjarbinx]
51c3cc38e25d: Preparing
eeed9f7c3966: Preparing
e4e39a1ab63d: Preparing
7f754426121f: Preparing
28a8796736c9: Preparing
8a70d251b653: Preparing
8a70d251b653: Waiting
7f754426121f: Layer already exists
e4e39a1ab63d: Layer already exists
eeed9f7c3966: Layer already exists
28a8796736c9: Layer already exists
8a70d251b653: Layer already exists
51c3cc38e25d: Pushed
1.0: digest: sha256:1fc6065769b596d87d8ec170f64f2cfcdb236e4ed200fae2a395a22739d7c760 size: 1573
```

### Para gerar o par de chaves pode-se usar no windows o [puttygen](https://putty.org), ao final da chave gerada o argumento iniciado por rsa-key-* deve ser alterado para o username do linux (no caso do Bastion Host). É o mesmo existente no campo "Key comment"

![image](https://user-images.githubusercontent.com/97816800/210683796-d30cc415-5baa-4f97-9589-3d8d59157e32.png)

### Ao se configurar no dockerfile para que o job tenha acesso ao Bastion Host, é necessário fornecer permissão para o aquivo Key.pem.

![image](https://user-images.githubusercontent.com/97816800/210686169-3b3576f6-206c-46b7-a48f-9c7e82486023.png)

```
chmod 400 $SSH_KEY (onde esse último é a variável do GitLab que você configurou com a Key)
```


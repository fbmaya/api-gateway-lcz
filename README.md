# Kong + Konga

Esse App tem objetivo de substituir o Ingress Controller do GKE e gerenciar as requests de acordo com a chamada.

## Installation

A instalação é feita através da esteira do Gitlab-ci

* Existe uma sequencia especifica para a instalação através do arquivos YAML. elas estão realizadas de forma hierárquica nas pastas 0-install e 1-install. Observe abaixo.



```bash
---
image: docker:latest

services:
  - docker:dind

stages:
  - deploy

deploy:
  stage: deploy
  image: kiwigrid/gcloud-kubectl-helm:latest
  script:
    - mkdir $CI_PROJECT_DIR/creds
    - export GOOGLE_APPLICATION_CREDENTIALS=$CI_PROJECT_DIR/creds/serviceaccount.json
    - echo $GOOGLE_APPLICATION_CREDENTIALS_BASE64 | base64 -d > $GOOGLE_APPLICATION_CREDENTIALS
    - gcloud auth activate-service-account ci-cd-740@fb-games-306023.iam.gserviceaccount.com  --key-file=$CI_PROJECT_DIR/creds/serviceaccount.json --project=fb-games-306023
    - gcloud container clusters get-credentials some-cluster --region us-central1 --project fb-games-306023
    - kubectl apply -f $CI_PROJECT_DIR/kong-manifests/0-install/
    - kubectl apply -f $CI_PROJECT_DIR/kong-manifests/1-install/
    - kubectl apply -f $CI_PROJECT_DIR/konga-manifests/
    - kubectl get pod -n kong
  only:
    - master
```

* Utilizamos uma imagem com gcloud + kubectl e Helm.

## Usage

A criação de rotas e services dentro do Kong foi automatizada atráves das APIs de Kind "KongIngress e Ingress" que estão dentro das pastas "k8s" nos projetos dos Apps. Observe o exemplo:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongIngress
metadata:
  name: backend-lcz
route:
  regex_priority: 0
  strip_path: true
  preserve_host: false
  protocols:
  - http
  - https
  paths:
  - /api

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: backend-lcz
  annotations:
    kubernetes.io/ingress.class: kong
spec:
   rules:
     - host: www.containerd.com.br
       http:
         paths:
           - path: /api
             backend:
               serviceName: backend-lcz-service
               servicePort: 80

```
* OBS: Essas configs de rotas e services são aplicadas dinamicamente no deploy dos apps.

## Diagrama

![alt text](https://raw.githubusercontent.com/fbmaya/api-gateway-lcz/master/Diagram.png)

## Contributing
Solicitações pull são bem-vindas. Para mudanças importantes, abra um problema primeiro para discutir o que você gostaria de mudar.

## License

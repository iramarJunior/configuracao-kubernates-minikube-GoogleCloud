# Configuracao-kubernates-Minikube-GoogleCloud

## Configurar o Kubernetes no Google Cloud

---

- Acessar o [console do google cloud](console.cloud.google.com).

- Acessar e ativar a API do Kubernetes Engine.

- Clicar no Web Console:

![console](https://github.com/iramarJunior/configuracao-kubernates-minikube-GoogleCloud/blob/master/img/start_interactive_cli.png)

- Instalar o kubectl:

```
$ gcloud components install kubectl
```

- Criar um cluster Kubernetes gerenciado e um conjunto de nós padrão:

```
$ gcloud container clusters create \
  --machine-type n1-standard-2 \
  --num-nodes 2 \
  --zone us-central1-a \
  --cluster-version latest \
  kubecluster
```

- Testar a Inicialização do Cluster:

```
$ kubectl get node
```

- Conceder à sua conta permissões para executar todas as ações administrativas necessárias:

```
$ kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=<DIGITE AQUI SEU EMAIL DO GOOGLE CLOUD>
```

## Configuração do Helm

---

- Instalar o Helm:

```
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
```

- Inicializar o Helm:

```
$ kubectl --namespace kube-system create serviceaccount tiller

$ kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller

$ helm init --service-account tiller --wait

$ helm init --client-only

$ kubectl patch deployment tiller-deploy --namespace=kube-system --type=json --patch='[{"op": "add", "path": "/spec/template/spec/containers/0/command", "value": ["/tiller", "--listen=localhost:44134"]}]'
```

- Verificar Instalação do Helm:

```
$ helm version
```

output:

```
$ Client: &version.Version{SemVer:"v2.16.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
```

## Configuração do JupyterHub

---

- Gerar uma cadeia hexadecimal para utilizar como token de segurança:

```
$ openssl rand -hex 32
```

- Criar um arquivo chamado _config.yaml_:

```
$ nano config.yaml
```

- Escrever o seguinte comando no arquivo subistituindo `<RANDOM_HEX>` pela sequência gerada no passo anterior:

```
proxy:
  secretToken: "<RANDOM_HEX>"
```

- Salvar o arquivo utilizando o comando `CTRL+X` ou `CMD+X`.

- Informar o Helm sobre o repositório de gráficos do JupyterHub Helm para instalar o gráfico do JupyterHub a partir dele:

```
$ helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/

$ helm repo update
```

- Instalar o gráfico utilizando o _config.yaml_ executando o comando no mesmo diretório desse arquivo:

```
$ RELEASE=jhub
NAMESPACE=jhub

helm upgrade --install $RELEASE jupyterhub/jupyterhub \
  --namespace $NAMESPACE  \
  --version=0.8.2 \
  --values config.yaml
```

## Vizualizando e acessando os serviços

---

> Para verificar a criação dos pods executar:

```
$ kubectl get pod --namespace jhub
```

> Para visualizar o ip para acessar o JupyterHub executar:

```
$ kubectl get service --namespace jhub
```

> Para visualizar o ip de forma completa executar:

```
$ kubectl describe service proxy-public --namespace jhub
```

> Para acessar os serviços basta colar na barra de endereço do navegador o IP listado em _EXTERNAL-IP_.

```
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
hub            ClusterIP      10.51.243.14    <none>          8081/TCP       1m
proxy-api      ClusterIP      10.51.247.198   <none>          8001/TCP       1m
proxy-public   LoadBalancer   10.51.248.230   104.196.41.97   80:31916/TCP   1m
```

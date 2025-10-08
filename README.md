# Projeto GitOps de Kubernetes

> A documentação a seguir, tem como objetivo apresentar um tutorial de como realizar um deploy GitOps local, usando Kubernetes e ArgoiCD para aplicação da Online Boutique.







## Etapa 1

#### Passo 1: Criar Fork e repositório no GitHub

1. Acesse o link: https://github.com/GoogleCloudPlatform/microservices-demo, no canto superior da página clique em  "Fork" para criar uma cópia desse repositório na sua conta do GitHub.
   * O Fork é uma cópia do repositório que pode sofrer modificações sem alterar o repositório original.

 <img src=""  alt="" width="700"/>
</p>
     



#### Passo 2: Criar repositório GitOps
É importante criar um repositório separado do Fork, para que possa anexar o manifesto necessário para o deploy. A estrutura utilizada será:

```
gitops
└── k8s/
    └── online-boutique.yaml
```


1. Crie um repositório público no GitHbub e clone localmente:
  ```

  git clone https://github.com/seuusuario/seurepositorio.git
  cd seurepositorio
  ```

2. Bauixe o arquivo "release" do fork e o renomeie para online-boutique.yaml. Crie uma pasta chamada  k8s dentro do seu repositório e mova o arquivo para a pasta.

3. Faça o commit e push do arquivo:
```
git add .
git commit -m "Adicionar manifest online-boutique"
git push origin main
```


## Etapa 2 

#### Passo 1: Iniciar o cluster e instalar o ArgoCD

1. Com o minekube instalado, inicie o cluster
```

minikube start
```

2. Crie o namespace onde o ArgoCD irá rodar
```
kubectl create namespace argocd
```
3. Fça a instalação do ArgoCD usando o manifesto oficial
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### Passo 2: Verifique os pods 

```
kubectl get pods -n argocd
```

* Aparecerá uma lista de podss e o ideal é que todos apresentem status de "Running". Pode demorar alguns minutos para carregar.


Etapa 3: Acesse o ArgoCD via Web

### Passo 1: Ativar op port-forward

1. Para criar uma conexão entre o navegador e o ArgoCD, para o acesso via navegador, use o comando:
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

2. Abra o navgador e e use a URl:
```
https://localhost:8080
```

3. Para fazer o login, usre "admin" como usuário padrão e para descobrir a senha, utilize o comando:

```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
```


## Etapa 3

#### Passo 1: Criar App no ArgoCD

1. Clique em " Criar Aplicação"  no ArgoCD e preencha os campos:

* Application Name: online-boutique
* Project: default
* Sync Policy: "Manual" 
* Repository URL: link HTTPS do repositório que você criou no GitHub
* Revision: main
* Path: k8s
* Cluster URL: use o endereço do minikube
* Namespace: default

2. Clique em "Create".


## Etapa 4

#### Passo 1: Verificação do Status

* Como  o ArgoCD ainda não foi sincronizado, o status deverá aparecer como "OutOfSync".
  
1. Para sicronizar o app, clique em "Sync" e aguarde até que apareça o status de saudável
   
2. Para verificar se os pods então rodando corretamente, execute no terminal:
```
kubectl get pods
```

3. Após a confirmação dos pods rodando, a página da Online Boutique poderá apareer perfeitamente.


### Problema encontrado!

Ao executar o deploy, o pod currencyservice ficava com status "pending".
No terminal, com o comando `kubectl getr pods` era exibido `currencyservice-768c464f5-sr9lz   0/1  Pending  0  10m`.  E ao rodar `kubectl describe pod currencyservice-768c464f5-sr9lz` foi exibido `Warning  FailedScheduling  ...  0/1 nodes are available: 1 Insufficient cpu.`. Demonstrando que o Minikube não tinha espaço o suficiente na CPU.

* Para a resolução do problema, foi necessário aumentar os recursos alocados para o Minekube através dos passos:

1. No Termianl, como administradoer, parar o Minekube de deletar o cluster.

```
minikube stop
minikube delete
```

2. Configurar mais CPU e memória

```
minikube config set cpus 4
minikube config set memory 4096
```

3. Reniciar o Minekube
```
minikube start
```

4. Após reiniciar, verificar se os pods estavam rodando corretamente, mostrando que oo problema foi resolvido.
```
kubectl get pods
```

## Conclusão
Com o projeto realizado, foi possíel colocar em prática, o aprendizado sobre Kubernetes e ArgoCD. Além de apresentar o tutorial do GitOps para o ambiente de Kubernetes.



   

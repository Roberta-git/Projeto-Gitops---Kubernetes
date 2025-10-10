# Projeto GitOps de Kubernetes

> A documentação a seguir, tem como objetivo apresentar um tutorial de como realizar um deploy GitOps local, usando Kubernetes e ArgoiCD para aplicação da Online Boutique.




## Sumário

- [Introdução](#projeto-gitops-de-kubernetes)
- [Ferramentas Utilizadas](#ferramentas-utilizadas)
- [Etapa 1: Preparação do Repositório](#etapa-1)
- [Etapa 2: Preparação do Cluster e Instalação do ArgoCD](#etapa-2)
- [Etapa 3: Acesso e Configuração do ArgoCD](#etapa-3)
- [Etapa 4: Deploy da Aplicação Online Boutique](#etapa-4)
- [Etapa 5: Configurações finais da aplicação](#etapa-5)
- [Problema encontrado: Pending no currencyservice](#problema-encontrado)
- [Conclusão](#conclusão)

---

## Ferramentas Utilizadas

- **Git** – Para versionamento dos arquivos e colaboração no desenvolvimento.
- **GitHub** – Hospedagem dos repositórios, incluindo o repositório do Fork (aplicação) e o repositório GitOps (manifests).
- **Minikube** – Cluster Kubernetes local para simular um ambiente real de deploy.
- **kubectl** – Ferramenta de linha de comando para gerenciar recursos Kubernetes.
- **ArgoCD** – Ferramenta GitOps para deploy, sincronização e visualização de aplicações via Git.
- **Online Boutique** – Aplicação de exemplo composta por microserviços.
- **Terminal/PowerShell** – Execução de todos os comandos e scripts necessários durante o processo.

---







## Etapa 1

#### Passo 1: Criar Fork e repositório no GitHub

1. Acesse o link: https://github.com/GoogleCloudPlatform/microservices-demo, no canto superior da página clique em  "Fork" para criar uma cópia desse repositório na sua conta do GitHub.
   * O Fork é uma cópia do repositório que pode sofrer modificações sem alterar o repositório original.

 <img src="https://github.com/user-attachments/assets/d299d708-e656-41d5-ab96-73d4f027ffff"  alt="" width="700"/>
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

 <img src="https://github.com/user-attachments/assets/f6f8c929-922c-4765-971a-eb3a7b88955e"  alt="" width="700"/>
</p>
     



## Etapa 3
#### Passo 1: Ativar op port-forward

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



 <img src="https://github.com/user-attachments/assets/2566d291-56be-4ab8-870a-a89b17210667"  alt="" width="700"/>
</p>
     


## Etapa 4

#### Passo 1: Criar App no ArgoCD

1. Clique em " Criar Aplicação"  no ArgoCD e preencha os campos:

* Application Name: online-boutique
* Project: default
* Sync Policy: "Manual" 
* Repository URL: link HTTPS do repositório que você criou no GitHub
* Revision: main
* Path: k8s
* Cluster URL: https://kubernetes.default.svc
* Namespace: default

2. Clique em "Create".


 <img src="https://github.com/user-attachments/assets/639194a7-527d-435b-921c-f5eb0d144098"  alt="" width="700"/>
</p>


## Etapa 5

#### Passo 1: Verificação do Status

* Como  o ArgoCD ainda não foi sincronizado, o status deverá aparecer como "OutOfSync".
  
1. Para sicronizar o app, clique em "Sync" e aguarde até que apareça o status de saudável
   
2. Para verificar se os pods então rodando corretamente, execute no terminal:
```
kubectl get pods
```

3. Após a confirmação dos pods rodando, a página da Online Boutique poderá apareer perfeitamente.


 <img src="https://github.com/user-attachments/assets/6f865cbb-0050-4124-8e7d-3db213068f29"  alt="" width="700"/>
</p>
     


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



### Adicionando réplicas
Para realizar a parte opcional do projeto e adicinar mais réplicas ao ArgoCD, siga os seguintes passos:


#### Passo 1: Alterar o manifesto YAML 
Edite o arquivo k8s/online-boutique.yaml, adicionando a linha `replicas: (quantidade de réplicas que você deseja)`, o bloco `spec` de cada Deployment e salve o arquivo.

 <img src="https://github.com/user-attachments/assets/c9cd2b70-b218-4754-8976-983880fe38f1"  alt="" width="700"/>
</p>



#### Passo 2: Commit e push
Faça o Commit e push para salvar as alterações feitas.

```
git add k8s/online-boutique.yaml
git commit -m "Ajusta número de réplicas dos microserviços"
git pull origin main            
git push origin main

```

Etapa 3: Sincronização
Abra o ArgoCD no navegador e clique em `Sync` para sincronizar os dados.
Verifique se aparece o número de pods desejado.



## Conclusão
Com o projeto realizado, foi possíel colocar em prática, o aprendizado sobre Kubernetes e ArgoCD. Além de apresentar o tutorial do GitOps para o ambiente de Kubernetes.



   

# Projeto CI/CD com Github Actions
Projeto feito no Windows para simular o CI/CD de uma empresa com Github Actions e Argo CD, com o Github como fonte de verdades.  
Argo CD é a ferramenta declarativa de entrega contínua (CD) e de GitOps para o Kubernetes.
# O que foi usado
Para este projeto, estes foram os "itens" usados:  
- Github com repositórios públicos  
- Conta no Docker Hub (com token de acesso)  
- Docker desktop  
- Cluster local (Kind) com ArgoCD instalado  
- Git  
- Python 3  
- Kubernetes
Uma recomendação é usar o VS Code com o terminal ativado pra fazer push mais facilmente pro Github, facilita todo o processo.  
Primeiramente, comece com a criação de dois repositórios no Github, um para o arquivo main.py (e mais alguns que serão explicados) e um (esse mesmo) para os manifestos.  
Com isso, podemos iniciar por etapas:  
# Etapa 1 - App
- 1° Passo  
Dentro da raiz (no Github mesmo ou na pasta no terminal com Git), crie uma pasta chamada ```app```, uma pasta chamada ```.github``` (também na raiz) e, dentro da pasta .github, crie uma pasta chamada ```workflows```. A estrutura deve ficar assim de início:  
├── hello-app/  
├── .github/  
│   └── workflows/  
└── app/  
- 2° Passo  
Após ter essa estrutura pronta, é hora de adicionar os arquivos:  
em  ```.github/workflows```, adicione o arquivo [ci.yaml](evidencias/ci.txt) (IMPORTANTE: mude a extensão do arquivo de ```.txt``` para ```.yaml```). Após ter adicionado, volte à pasta app e adicione o [arquivo main.py](evidencias/main.txt) (mude o tipo do arquivo de ```.txt``` para ```.py```) e o [arquivo dos requerimentos](evidencias/requerimentos.txt) (mude o nome do arquivo para ```requirements.txt```) e adicione o arquivo [Dockerfile](evidencias/docker_file.txt) na raiz do repositório (mude o nome apenas para ```Dockerfile```, sem nenhuma extensão).  
A estrutura final tem que ficar assim:  
├── hello-app/
├── .github/
│   └── workflows/
│       └── ci.yaml  
├── app/
│   ├── main.py  
│   └── requirements.txt  
└── Dockerfile  
Com tudo isso pronto, podemos partir pro próximo passo.  
- 3° Passo  
Ainda no repositório do app (no Github mesmo, não mais no terminal), clique em ```Settings```, vá até ```Secrets and variables``` e clique em ```Actions```. Em Actions, é necessária a criação de cinco segredos: ```DOCKER_USERNAME```, ```DOCKER_PASSWORD```, ```SSH_PRIVATE_KEY```, ```MANIFEST_REPO``` e ```GIT_EMAIL```. Eles são usados pelo [ci.yaml](evidencias/ci.txt).  
Os valores guardados em cada segredo têm que ser inseridos manualmente, e para obter os valores:  
**DOCKER_USERNAME**: guarda seu nome (Docker ID) do Docker Hub. É usado para autenticar o Github Actions ao fazer build e push da imagem para o Docker Hub.  
**DOCKER_PASSWORD**: guarda seu Token de Acesso Pessoal (PAT). É a credencial usada com permissão ```Write``` para publicar a imagem no Docker Hub. Para obter uma:  
1. Fazer o login;  
2. Vá no seu ícone e clique em Account Settings;  
3. Navegue até Personal Access Token e clique lá;  
4. Clique em Generate new token;  
5. Dê uma descrição ao token (exemplo: github-actions-app-token);  
6. Mude a permissão para Read & Write;  
7. Clique em Generate;  
8. IMEDIATAMENTE, após o gerar o token, copie o valor completo do token gerado;  
9. Adicione o valor no segredo ```DOCKER_PASSWORD``` (e nunca o compartilhe)  
Assim, é obtido o valor necessário.  
**SSH_PRIVATE_KEY**: é o conteúdo do arquivo ```gitops_key```. Para obté-lo, basta usar este comando:  
```ssh-keygen -t rsa -b 4096 -f gitops_key -C "gitops"```  
e copiar 100% do conteúdo dele para o segredo (ele é usado como uma senha para o Github Actions autenticar e provar que é uma ação sua).  
Ele é usado para gerar um arquivo além do gitops_key, o ```gitops_key.pub```, que é a chave pública.  
**MANIFEST_REPO**: é o URL SSH do repositório dos manifestos, o alvo da operação de GitOps. O link SSH é:  
```git@github.com:seu-user/repositorio-dos-manifestos.git```  
**GIT_EMAIL**: é seu e-mail do Github, basta adicioná-lo ao segredo.  
Com isso, você tem tudo do repositório do app pronto!  
# Etapa 2 - Manifestos
Com a 1° etapa pronta, podemos dar início à segunda, e nela, é ainda mais simples: basta adicionar os arquivos de [deployment](deployment.yaml) e [service](service.yaml).  
O arquivo de deployment serve para controlar o estado dos Pods que serão criados. Se você atualizar o main.py, ele executa o rollout (substituição dos pods).  
O arquivo de service é o "endereço" do aplicativo. Ele roteia o tráfego que chega para a porta 80 (do service) para a porta 8000 (do container), e faz isso de forma consistente, mesmo que os pods por trás dele morram e sejam recriados.  
Juntos, garantem que sua aplicação esteja acessível e rodando! Agora, o repositório que será usado pelo ArgoCD está pronto!  
# Etapa 3 - ArgoCD
Para dar início ao processo do ArgoCD, é necessário baixar o Kind. Ele é necessário para fazer o cluster local. Para baixá-lo, vá no navegador e cole esse link na pesquisa:  
```https://kind.sigs.k8s.io/dl/v0.30.0/kind-windows-amd64```  
Após isso, renomeie o arquivo baixado para ```kind.exe``` e mova-o para a pasta ```System32``` (C:\Windows\System32). Após isso, para testar o Kind, abra o terminal e use o comando:  
````kind --version```  
Se aparecer a versão do Kind, deu certo!  
Agora, com o funcionamento do Kind testado, siga os passos abaixo:  
1. Volte ao terminal e use este comando:  
```kind create cluster --name argocd-cluster```  
Ele cria o cluster e configura o Docker para usá-lo como contexto, fazendo o kubectl funcionar. Pode demorar um pouco.  
Após a criação e mais alguns minutos, você já pode fazer o teste com este comando:  
```kubectl get nodes```  
Se retornar o nome, o status ```Ready``` e as outras informações, deu certo!  
Agora, execute os comandos  
```kubectl create namespace argocd```;
```kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml``` (esse demora um pouco)  
Após realizar o último comando, use o comando ```kubectl get pods -n argocd``` para ver se está funcionando. Ele tem que mostrar informações como as abaixo:  
![img1](evidencias/argocdpods.png)  
2. Agora que o ArgoCD está configurado:  
Use este comando:  
```kubectl port-forward svc/argocd-server -n argocd 8080:443```  
Ele fará o site do ArgoCD ficar disponível para você usar.  
Agora, abra seu navegador e digite ```http://localhost:8080``` e você estará no ArgoCD!  
3. No ArgoCD:  
![img2](evidencias/argocdts.png)  
Seu username sempre será ```admin``` e sua senha é obtida no terminal usando o comando:  
```(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | Out-String) -replace '\s' | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }```  
Copie o resultado e volte ao ArgoCD e insira tal resultado na senha (sim, esse resultado maluco é sua senha) e clique em sign in.  
4. Agora que está no ArgoCD, siga:  
Clique em ```+ NEW APP```
![img3](evidencias/passo1.png)  
Escolha um nome para a sua aplicação, selecione ```default``` como nome do projeto ```Automatic``` em ```SYNC POLICY```  
![img4](evidencias/passo2.png)  
Desça até ```Source``` e cole o link do seu repositório com os manifestos e em ```Path``` digite ```.```, ele vai ler a raiz do seu repositório  
![img5](evidencias/passo3.png)  
Desça mais um pouco até ```DESTINATION```, clique em ```Cluster URL``` e selecione a única opção (```https://kubernetes.default.svc```) e, em ```Namespace```, digite ```default```
![img6](evidencias/passo4.png)  
Agora, no canto superior esquerdo, clique em ```CREATE```e sua aplicação estará pronta!  
![img7](evidencias/passo5.png)  
Vai aparecer uma nova aplicação na sua tela principal e, depois de um tempo, se tudo tiver dado certo, sua tela vai aparecer assim:  
![img8](evidencias/sucesso.png)  
# Etapa 4 - Fim/testes
Com tudo pronto, você pode iniciar seus testes! O primeiro deles é port-forwarding do seu aplicativo. Para isso, volte ao terminal e dê ```ctrl+c``` para parar o ArgoCD e dê esse comando:  
```kubectl port-forward svc/hello-app-service 8080:80 -n default```  
Volte no navegador e digite ```http://localhost:8080```  
Você deve ver algo parecido com isso:  
![img9](evidencias/app.png)  
Para o próximo teste, você pode testar suas Actions:  
Dê ````ctrl+c``` para parar o app;  
Vá para o repositório do seu app e vá até o ```main.py``` e faça qualquer mudança (mudei a mensagem no exemplo)  
![img10](evidencias/changes.png)  
e dê commit.  
O GitHub Actions gera uma nova tag SHA para a imagem e, como parte do script de GitOps, cria o Pull Request no repositório dos manifestos com essa nova tag.  
![img11](evidencias/action.png)  
Depois de um tempo, vai ficar com um sinal de ok, e quando aparecer esse sinal, vá até o repositório dos manifestos e vai ter uma notificação:  
![img12](evidencias/notify.png)  
Clique em ```Compare & pull request```  
Você será levado a essa tela:  
![img13](evidencias/pr.png)  
Clique em ```Create & pull request```  
Agora, você será levado a essa tela:  
![img14](evidencias/confirming.png)  
Clique em ```Merge pull request```  
E nessa última tela:  
![img15](evidencias/confirm.png)  
Clique em ```Confirm merge```  
E pronto, você agora sabe como atualizar sua imagem! inclusive, pode voltar a fazer o port-forwarding e verá suas mudanças:  
![img16](evidencias/app2.png)  
Inclusive, a imagem sempre é atualizada no Docker Hub:  
![img17](evidencias/dockerhubimage.png)  

É isso. Agora, você já sabe simular o CI/CD!


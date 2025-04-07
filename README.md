# **CI/CD com Jenkins: Automatizando Builds e Pushes para Docker Hub**

Este projeto demonstra como configurar um pipeline Jenkins para automatizar a construção de imagens Docker e o envio (push) para o Docker Hub. O pipeline utiliza um `Jenkinsfile` declarativo para definir as etapas do processo de CI/CD.

---

## **Índice**
1. [Visão Geral](#visão-geral)
2. [Pré-requisitos](#pré-requisitos)
3. [Configuração do Ambiente](#configuração-do-ambiente)
   - [Instalação do Jenkins](#instalação-do-jenkins)
   - [Instalação do Docker](#instalação-do-docker)
   - [Credenciais do Docker Hub](#credenciais-do-docker-hub)
4. [Configuração do Pipeline no Jenkins](#configuração-do-pipeline-no-jenkins)
5. [Estrutura do Projeto](#estrutura-do-projeto)
6. [Explicação do Jenkinsfile](#explicação-do-jenkinsfile)
7. [Execução do Pipeline](#execução-do-pipeline)
8. [Erros Comuns e Soluções](#erros-comuns-e-soluções)
9. [Contribuição](#contribuição)
10. [Licença](#licença)
11. [Contato](#contato)

---

## **Visão Geral**

O objetivo deste projeto é automatizar o processo de construção e envio de imagens Docker para o Docker Hub usando o Jenkins. O pipeline é definido em um arquivo `Jenkinsfile`, que inclui as seguintes etapas:

1. **Login no Docker Hub**: Autenticação no Docker Hub usando credenciais seguras.
2. **Build da Imagem Docker**: Construção da imagem Docker com base no `Dockerfile`.
3. **Push para o Docker Hub**: Envio da imagem construída para o Docker Hub.

---

## **Pré-requisitos**

Antes de começar, certifique-se de que os seguintes requisitos estão atendidos:

- **Jenkins instalado e configurado**.
- **Docker instalado e configurado**.
- **Conta no Docker Hub** com permissões de `write` habilitadas.
- **Repositório Git** contendo o código-fonte e o `Dockerfile`.

---

## **Configuração do Ambiente**

### **Instalação do Jenkins**

1. Instale o Jenkins seguindo a [documentação oficial](https://www.jenkins.io/doc/book/installing/).
2. Após a instalação, acesse o Jenkins em `http://localhost:8080` e conclua a configuração inicial.

### **Instalação do Docker**

1. Instale o Docker seguindo a [documentação oficial](https://docs.docker.com/get-docker/).
2. Certifique-se de que o serviço Docker está rodando:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

### **Credenciais do Docker Hub**

1. No Docker Hub, gere um token de acesso pessoal (PAT) com permissões de `read` e `write`.
2. No Jenkins, adicione as credenciais:
   - Acesse `Manage Jenkins > Manage Credentials`.
   - Adicione uma nova credencial do tipo `Username with password`.
   - Insira seu nome de usuário (`luandocs`) e o token gerado.

---

## **Configuração do Pipeline no Jenkins**

1. Crie um novo job no Jenkins:
   - Escolha o tipo **Pipeline**.
   - Configure o repositório Git na seção **Source Code Management (SCM)**:
     - Repository URL: `https://github.com/seu-usuario/seu-repositorio.git`
     - Branch: `*/main`
   - Na seção **Pipeline**, escolha:
     - Definition: `Pipeline script from SCM`.
     - Script Path: `Jenkinsfile`.

2. Salve o job e execute-o manualmente para testar.

---

## **Estrutura do Projeto**

Aqui está a estrutura básica do projeto:

```
seu-repositorio/
├── Jenkinsfile                # Arquivo de definição do pipeline
├── Dockerfile                 # Definição da imagem Docker
├── app.py                     # Código da aplicação Flask
├── requirements.txt           # Dependências Python
└── README.md                  # Documentação do projeto
```

---

## **Explicação do Jenkinsfile**

O arquivo `Jenkinsfile` define o pipeline declarativo. Aqui está o conteúdo detalhado:

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "luandocs/myapp-jenkins" // Substitua pelo seu nome de usuário e nome da imagem
    }

    stages {
        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
                        sh """
                            echo \$DOCKER_HUB_PASS | docker login -u \$DOCKER_HUB_USER --password-stdin
                        """
                    }
                    echo "Logado no Docker Hub!"
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    echo "Building image with tag: ${env.BUILD_ID}"
                    def appImage = docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                    appImage.push()
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executado com sucesso!"
        }
        failure {
            echo "Ocorreu um erro no pipeline."
        }
    }
}
```

### **Explicação das Etapas**

1. **Environment Variables**:
   - Define variáveis globais, como o nome da imagem Docker.

2. **Login to Docker Hub**:
   - Usa `withCredentials` para injetar as credenciais de forma segura.
   - Executa o comando `docker login` com `--password-stdin` para evitar expor senhas.

3. **Build and Push Docker Image**:
   - Constrói a imagem Docker com base no `Dockerfile`.
   - Envia a imagem para o Docker Hub com a tag correspondente ao número do build.

4. **Post Actions**:
   - Exibe mensagens de sucesso ou falha após a execução do pipeline.

---

## **Execução do Pipeline**

1. Faça um push no repositório GitHub para acionar o pipeline.
2. Monitore o progresso no Jenkins:
   - Verifique os logs para garantir que todas as etapas foram concluídas com sucesso.
3. Confirme no Docker Hub que a imagem foi enviada corretamente.

---

## **Erros Comuns e Soluções**

### **Erro 1: `denied: requested access to the resource is denied`**
- **Causa**: Problemas com credenciais ou permissões.
- **Solução**:
  - Verifique se as credenciais estão corretas.
  - Certifique-se de que o token tem permissão de `write`.

### **Erro 2: `Cannot run program "docker"`**
- **Causa**: O Jenkins não tem acesso ao Docker.
- **Solução**:
  - Adicione o usuário Jenkins ao grupo Docker:
    ```bash
    sudo usermod -aG docker jenkins
    sudo systemctl restart jenkins
    ```

### **Erro 3: Repositório Não Existe**
- **Causa**: O repositório no Docker Hub não foi criado.
- **Solução**:
  - Crie o repositório manualmente no Docker Hub antes de fazer o push.

---

## **Contribuição**

Contribuições são bem-vindas! Se você deseja contribuir para este projeto, siga estas etapas:

1. Faça um fork do repositório.
2. Crie uma branch para sua feature ou correção:
   ```bash
   git checkout -b feature/nova-feature
   ```
3. Faça suas alterações e commit:
   ```bash
   git commit -m "Adiciona nova feature"
   ```
4. Envie suas alterações para o repositório remoto:
   ```bash
   git push origin feature/nova-feature
   ```
5. Abra um Pull Request no repositório original.

---

## **Licença**

Este projeto está licenciado sob a **MIT License**. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## **Contato**

Se tiver dúvidas ou sugestões, entre em contato:

- **Nome**: Luan Castro
- **Email**: luandecastrosilva@gmail.com
- **LinkedIn**: [linkedin.com/in/luancastrosilva](https://www.linkedin.com/in/luancastrosilva/)
- **GitHub**: [github.com/luangitdev](https://github.com/luangitdev)
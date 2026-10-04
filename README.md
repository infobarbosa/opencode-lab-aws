# Ambiente de Laboratório: Engenharia de Dados com OpenCode e Ollama (AWS EC2 + code-server)

Author: Prof. Barbosa<br>
Contact: infobarbosa@gmail.com<br>
Github: [infobarbosa](https://github.com/infobarbosa)

---

## 1. Visão Geral e Arquitetura

Bem-vindo ao laboratório prático de Engenharia de Dados com Inteligência Artificial Generativa! Este ambiente foi projetado para fornecer uma experiência profissional, rápida e padronizada utilizando uma instância **AWS EC2** (`m5.large`) equipada com **Docker**, executando um container com a stack completa para desenvolvimento orientado por especificações (Spec-Driven Development):

- **IDE Web:** `code-server` (Visual Studio Code acessível diretamente no seu navegador, porta 8080).
- **Runtime de LLM:** `ollama` (servidor de inferência local e ponte para modelos cloud, porta 11434 interna).
- **Coding Agent:** `opencode` (CLI oficial do agente de codificação integrado ao Ollama).
- **Engenharia de Dados:** `Python 3`, `PySpark`, `Java 21 Headless`, `Boto3` e `Parquet-Tools`.

Todo o seu trabalho fica persistido no diretório `/home/ubuntu/workspace` da instância EC2, que é montado dentro do container em `/home/barbosa/project`. Dessa forma, mesmo que o container seja reiniciado ou a máquina seja desligada, seu código e arquivos permanecem intactos no disco EBS.

### Arquitetura do Ambiente

```text
+-------------------------------------------------------------------------------+
| Instância AWS EC2 (m5.large - Ubuntu 24.04 LTS)                               |
|                                                                               |
|  Disco EBS Persistente: /home/ubuntu/workspace (UID/GID 1000)                 |
|                                  ▲                                            |
|                                  │ (Montagem de Volume -v)                    |
|                                  ▼                                            |
|  +-------------------------------------------------------------------------+  |
|  | Container Docker: opencode-lab-docker-image                             |  |
|  | Diretório de Trabalho: /home/barbosa/project                            |  |
|  |                                                                         |  |
|  |  [code-server:8080] <--- Acesso via Navegador Web (Passwordless)        |  |
|  |                                                                         |  |
|  |  Terminal Integrado:                                                    |  |
|  |   ├── OpenCode CLI (Agente de IA)                                       |  |
|  |   ├── Ollama Server (Porta 11434 - Backend de Modelos)                  |  |
|  |   └── PySpark + Java 21 Runtime (Execução de Pipelines)                 |  |
|  +-------------------------------------------------------------------------+  |
+-------------------------------------------------------------------------------+
```

### ⚠️ Alerta Crítico: Região e Gestão de Custos

> **ATENÇÃO - OBRIGATÓRIO:**
>
> 1. **Região Ativa:** Certifique-se sempre de que você está operando na região **Norte da Virgínia (`us-east-1`)** no canto superior direito do Console AWS. Todos os recursos do laboratório devem residir exclusivamente nessa região.
> 2. **Consumo de Créditos:** A instância EC2 `m5.large` e o volume EBS gp3 consomem créditos do seu laboratório AWS Academy.
> 3. **Desligamento Automático:** O AWS Academy desliga suas máquinas após o período da sessão (timeout de 4 horas). Ao encerrar sua aula ou sessão de estudos, garanta que a instância esteja parada (**Stopped**) ou aguarde o desligamento automático.
> 4. **NUNCA Recrie a Máquina:** Uma vez provisionada na Aula 1, você **não** deve executar o script de provisionamento novamente nas aulas seguintes. Siga as instruções do *Guia de Sobrevivência* para religar a máquina existente.

---

## 2. Passo a Passo: Aula 1 (Provisionamento Único)

Na sua primeira aula de laboratório, executaremos o script de provisionamento automatizado via **AWS CloudShell**. Este passo só precisa ser feito **uma única vez** no início do curso.

### Passo 1: Abrir o AWS CloudShell

1. Faça login no Console da AWS através do seu painel do AWS Academy.
2. Certifique-se de que a região selecionada no canto superior direito é **N. Virginia (`us-east-1`)**.
3. No topo do console, clique no ícone do **AWS CloudShell** (`>_`) ao lado da barra de pesquisa, ou pesquise por "CloudShell" no menu de serviços.
4. Aguarde o terminal do CloudShell carregar.

### Passo 2: Executar o Script de Provisionamento

No terminal do CloudShell, copie e cole o comando "one-liner" abaixo e pressione `Enter`:

```bash
curl -sS https://raw.githubusercontent.com/infobarbosa/opencode-lab-aws/main/launch-lab.sh | bash
```

O script realizará de forma totalmente automatizada:
- Descoberta da sua VPC padrão e Subnet pública.
- Criação e configuração do Security Group (`opencode-lab-sg`) liberando as portas `8080` (IDE) e `22` (SSH).
- Consulta da AMI oficial mais recente do Ubuntu Server 24.04 LTS via SSM.
- Lançamento da instância EC2 `m5.large` com volume EBS de 30 GB gp3 e perfil IAM educacional.
- Execução do User Data (instalação do Docker e inicialização do container com a imagem pública `ghcr.io/infobarbosa/opencode-lab-docker-image:latest`).
- Monitoramento até a instância estar ativa (`running`), exibindo o IP público de acesso.

### Passo 3: Acessar a IDE code-server

Ao final da execução, o terminal exibirá as seguintes informações:

```text
===================================================================
            AMBIENTE DE LABORATÓRIO PRONTO PARA USO!              
===================================================================
 Instância ID:     i-0123456789abcdef0
 IP Público:       54.xxx.xxx.xxx
 Link de Acesso:   http://54.xxx.xxx.xxx:8080
 Autenticação:     Passwordless (Acesso Direto - Sem Senha)
-------------------------------------------------------------------
 IMPORTANTE - TEMPO DE INICIALIZAÇÃO:
 O script User Data está instalando o Docker e baixando a imagem oficial
 com code-server, PySpark, Ollama e OpenCode em segundo plano.
 -> Tempo estimado de download e inicialização: 3 a 5 minutos.
 Se a página web não responder de imediato, aguarde 2 minutos e
 atualize o seu navegador (Ctrl+F5 ou Cmd+Shift+R).
-------------------------------------------------------------------
 PRIMEIROS PASSOS NO TERMINAL DO CODE-SERVER:
 1. Autenticar no Ollama:        ollama login
 2. Iniciar o OpenCode:          ollama launch opencode --model gemma4:cloud
===================================================================
```

1. Abra uma nova aba no seu navegador e cole o endereço: `http://<IP-PUBLICO>:8080`.
2. Caso a página apresente erro de conexão inicial, **aguarde de 3 a 5 minutos** para que o Docker conclua o download da imagem e inicialize o serviço.
3. O **code-server** iniciará diretamente na interface do VS Code sem solicitar senha.

---

## 3. Configuração Inicial e Ativação do Agente de IA

Com o VS Code aberto no seu navegador, execute os passos abaixo para autenticar o modelo de IA e configurar seu repositório:

### 3.1. Abrir o Terminal Integrado

Na interface do code-server:
- Clique no menu superior (três linhas horizontais no canto superior esquerdo).
- Selecione **Terminal** > **New Terminal** (ou pressione `Ctrl + ` ` ` / ``Cmd + ` ``).

### 3.2. Autenticação no Ollama (Obrigatório para o modelo Cloud)

O modelo utilizado nas aulas é o **Gemma 4** (`gemma4:cloud`). Para que o Ollama acesse o modelo na nuvem sem necessidade de GPU local pesada, execute:

```bash
ollama login
```

Siga as instruções exibidas na tela para autenticar sua conta. Após o login, valide a conectividade:

```bash
curl http://localhost:11434/api/tags
```

### 3.3. Testar a Inicialização do OpenCode

Inicie o agente interativo de codificação:

```bash
ollama launch opencode --model gemma4:cloud
```

- O OpenCode abrirá na interface TUI de terminal, conectado à API local do Ollama.
- Para sair da interface do agente e retornar ao terminal bash, digite `/exit` (ou pressione `Ctrl + C`).

### 3.4. Configurar Autenticação Git via SSH (Opcional, mas Recomendado)

Para enviar seus projetos desenvolvidos no laboratório para o seu GitHub pessoal:

1. Gere um par de chaves SSH no terminal:
   ```bash
   ssh-keygen -t ed25519 -C "seu-email@exemplo.com"
   ```
   *(Pressione Enter em todas as perguntas para aceitar os valores padrão).*
2. Exiba a chave pública gerada:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
3. Copie o texto exibido e cadastre-o no seu GitHub em: **Settings > SSH and GPG keys > New SSH key**.
4. Teste a conexão segura:
   ```bash
   ssh -T git@github.com
   ```
5. Configure sua identidade global do Git:
   ```bash
   git config --global user.name "Seu Nome Completo"
   git config --global user.email "seu-email@exemplo.com"
   git config --global init.defaultBranch main
   ```

---

## 4. Guia de Sobrevivência: Aulas Seguintes (Retomada Sem Perda de Dados)

O ambiente AWS Academy desliga suas máquinas após 4 horas de uso contínuo, colocando a sua instância no estado **Stopped** (Parada).

### Como Religar a Instância Existente:

1. Acesse o Console AWS na região **us-east-1**.
2. No campo de busca superior, digite **EC2** e clique no serviço.
3. No menu lateral esquerdo, clique em **Instances** (Instâncias).
4. Localize a sua instância chamada **`lab-opencode-ai`** (ela estará com status *Stopped*).
5. Selecione a caixinha ao lado da instância, clique no botão superior **Instance state** e selecione **Start instance** (Iniciar instância).
6. Aguarde cerca de 1 minuto até o status mudar para **Running** (Em execução).

### Como Obter o Novo Endereço de Acesso:

> ℹ️ **Atenção:** No modelo sem IP elástico, toda vez que uma máquina é parada e reiniciada, a AWS atribui um **novo endereço IP público**.

1. Com a instância `lab-opencode-ai` selecionada na lista, localize no painel de detalhes o campo **Public IPv4 address** (Endereço IPv4 público).
2. Copie esse endereço e acesse no navegador adicionando a porta `:8080`:
   ```text
   http://<NOVO_IP_PUBLICO>:8080
   ```
3. O code-server abrirá diretamente na IDE.
4. Como o container Docker está configurado com `--restart always`, ele iniciará automaticamente junto com o boot da máquina, e todos os seus arquivos em `/home/barbosa/project` estarão exatamente onde você os deixou.

---

## 5. Próximo Passo: Mão na Massa!

Com o seu ambiente pronto e validado, siga diretamente para o laboratório prático dirigido por especificação:

👉 **Acesse o tutorial da aula:** [Data-Pipelines-com-GenAI](https://github.com/infobarbosa/Data-Pipelines-com-GenAI)

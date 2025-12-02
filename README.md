# Automação de Postagem de Vídeos no YouTube com n8n

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-6BA0B4?style=flat-square&logo=ai&logoColor=white)
![YouTube](https://img.shields.io/badge/YouTube-FF0000?style=flat-square&logo=youtube&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-3390EC?style=flat-square&logo=telegram&logoColor=white)
![Ngrok](https://img.shields.io/badge/Ngrok-140648?style=flat-square&logo=ngrok&logoColor=white)

---

***Descrição: Sistema completo de automação para publicação de vídeos no YouTube usando n8n, Ollama (IA local) e Docker. O fluxo é acionado por agendamento, o AgentAI processa vídeos automaticamente gera título, descriçõe e hashtags notifica resultados via Telegram. Inclui Ngrok para acesso remoto seguro.***

---

### 🚀 Funcionalidades Principais

- **🕐 Agendamento Inteligente**: Acionamento automático via CRON
- **🎯 Seleção Aleatória**: Escolhe vídeos automaticamente do diretório
- **🤖 IA Local (Ollama)**: Gera títulos, descrições e hashtags
- **📤 Upload Automático**: Posta vídeos via YouTube Data API
- **📱 Notificações**: Envia status e links via Telegram
- **🗂️ Organização**: Move arquivos processados automaticamente
- **🌐 Tunnel**: Ngrok fornece acesso remoto seguro

---

### 🏗️ Arquitetura do Sistema

```
Agendamento CRON → Seleção Vídeo → Validação → IA Ollama → Formatação → Binário → Upload YouTube → Notificação Telegram → Mover Vídeo
     ↑
Ngrok Tunnel (Acesso Remoto)
```
---

### 📋 Pré-requisitos
- **Docker** (>= 20.10) e **Docker Compose**
- **Credenciais:**
  - YouTube Data API (Client ID, Secret, API Key, Refresh Token) [Google Cloud](https://cloud.google.com/)
  - Telegram Bot Token (obtido via @BotFather) [Telegram BotFather](https://telegram.me/BotFather)
  - Ngrok Auth Token [Ngrok](https://dashboard.ngrok.com/get-started/your-authtoken)
  - Acesso ao Ollama 3.2 (local)
    
---

### 📁 Estrutura do Projeto
```
\\wsl.localhost\Ubuntu\home\user\docker-n8n

\\wsl.localhost\Ubuntu\
 └──home/
    └──user/
       └──docker-n8n/
          └── n8n_data/                    # Dados persistentes do n8n
              │   ├── config/
              │   ├── binaryData/
              │   └── nodes/
              ├── videos_novos/            # Vídeos aguardando processamento
              ├── videos_postados/         # Vídeos já publicados
              ├── docker-compose.yml       # Orquestração de serviços
              └── docker+n8n.json          # Backup do workflow n8n                             
```              

---

### ⚙️ Docker Compose
```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_docker
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - GENERIC_TIMEZONE=America/Sao_Paulo 
      - N8N_HOST_MODE=default
    extra_hosts: 
      - "host.docker.internal:host-gateway"
    volumes:
      - ./n8n_data:/home/node/.n8n
      - ./videos_novos:/files/videos_novos
      - ./videos_postados:/files/videos_postados
  ngrok:
    image: ngrok/ngrok:latest
    container_name: ngrok_docker
    restart: unless-stopped
    networks:
      - default
    ports:
      - "4040:4040"
    environment:
      NGROK_AUTHTOKEN: "SuaAuthtoken" 
      NGROK_REGION: "sa" 
    command: 
      "http n8n:5678" 
    depends_on:
      - n8n
networks:
  default:
    driver: bridge
```

---

## 🛠️ Configuração

**1. Abra o `docker-compose.yaml` no seu VSCode**

**2. Configure no `docker-compose.yaml` sua NGROK_AUTHTOKEN** [Ngrok](https://dashboard.ngrok.com/get-started/your-authtoken)

**3. No terminal Ubuntu(WSL) execute o comando: `docker-compose up -d`**

**4. Acesse o n8n em `http://localhost:5678`**

**5. Vá para Workflows → Importe o arquivo `docker+n8n.json`**

**6. Configure as credenciais nos nós:**
   - **YouTube API** (OAuth2) [Google Cloud](https://cloud.google.com/)
   - **Telegram Bot** (Token do Bot) [Telegram BotFather](https://telegram.me/BotFather) 
   - **HTTP Request** para Ollama (`http://host.docker.internal:11434`)
  
<img src="https://i.imgur.com/0BQgrVk.png" alt="VSCode Settings" width="650">

*(Imagem: Workflow docker+n8n.json)*


---

### 🎯 Como Usar

*Coloque vídeos no diretório de entrada*
`\\wsl.localhost\Ubuntu\home\rafavfa\docker-n8n\videos_novos`

### 🛡️ Configuração, Permissões e Inicialização!

*Siga esta sequência de comandos no seu terminal WSL, dentro do diretório `~/docker-n8n`, para garantir que o Docker e a aplicação n8n tenham as permissões corretas para acessar os volumes.*

1.  **Corrigir Acesso ao Docker (Grupo):** *Adiciona o usuário ao grupo `docker` para habilitar o acesso ao Docker sem `sudo`.*
    ```bash
    sudo usermod -aG docker [usuario_padrao]
    ```
2.  **Navegar para o Projeto:** *Navega para o diretório raiz do projeto.*
    ```bash
    cd ~/docker-n8n
    ```
3.  **Ativar Permissões de Grupo:** *Ativa as permissões do grupo `docker` no shell atual. (Você permanecerá na pasta `~/docker-n8n`).*
    ```bash
    newgrp docker
    ```
4.  **Parar Containers Antigos:** *Para e remove quaisquer containers antigos para iniciar o processo de limpeza de permissões.*
    ```bash
    docker-compose down
    ```
5.  **Definir Proprietário dos Volumes (`chown`):** *Define o seu usuário como proprietário recursivo de todos os volumes para evitar problemas de "Acesso Negado".*
    ```bash
    sudo chown -R [usuario_padrao]:[usuario_padrao] n8n_data videos_novos videos_postados
    ```
6.  **Definir Permissão do Volume n8n\_data:** *Concede permissão de leitura/escrita/execução (`775`) para o volume de configuração.*
    ```bash
    sudo chmod -R 775 n8n_data
    ```
7.  **Definir Permissão do Volume videos\_novos:** *Concede permissão `775` para a pasta de vídeos de entrada, permitindo acesso tanto ao seu usuário quanto ao Docker.*
    ```bash
    sudo chmod -R 775 videos_novos
    ```
8.  **Definir Permissão do Volume videos\_postados:** *Concede permissão `775` para a pasta de vídeos postados, permitindo que o n8n mova os arquivos.*
    ```bash
    sudo chmod -R 775 videos_postados
    ```
9.  **Iniciar o Projeto:** *Inicia os containers em segundo plano.*
    ```bash
    docker-compose up -d
    ```
***⚠️ Nota: Substitua `[usuario_padrao]` pelo seu usuário real.***

---

### 2. O Sistema Processa Automaticamente
- Agendamento CRON aciona o workflow
- Vídeo aleatório é selecionado
- IA gera metadados otimizados
- Upload é realizado no YouTube
- Notificação é enviada via Telegram
- Arquivo é movido para `videos_postados/`

### 3. Monitoramento Local ou Remoto
- **n8n:** Acesse via URL Ngrok de qualquer lugar
- **Ngrok:** Interface: http://localhost:4040 (local)
- **Telegram:** Receba notificações em tempo real
- **Logs:** `docker-compose logs -f n8n`

---

### ⚠️ Boas Práticas
- **🔒 Segurança:** Nunca versionar com credenciais
- **📊 Backup:** Exporte workflows regularmente do n8n
- **📈 Monitoramento:** Acompanhe quotas da YouTube API
- **🔄 Atualização:** Mantenha imagens Docker atualizadas
- **🔍 Logs:** Monitore execuções via interface n8n

### YouTube API Quotas
- Monitorar uso em [Google Cloud Console](https://console.cloud.google.com)
- Limite padrão: 10 unidades/dia

### 📄 Licença
Este projeto está sob licença MIT. Veja o arquivo [LICENSE.md](LICENSE.md) para detalhes.

### ❓ Suporte
- **Documentação n8n:** https://docs.n8n.io
- **YouTube Data API:** https://developers.google.com/youtube/v3
- **Ngrok:** https://ngrok.com/docs/what-is-ngrok
- **Ollama:** https://ollama.ai
- **Issues:** [GitHub Issues](https://github.com/rafavfa/autopost-videos-on-youtube-with-n8n/issues)

---
### ⭐ Se este projeto foi útil, considere dar uma estrela no repositório!

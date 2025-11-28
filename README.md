# Automação de Postagem de Vídeos no YouTube com n8n

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-FF6D00?style=flat-square&logo=n8n&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-2b7a78?style=flat-square&logo=ai&logoColor=white)
![YouTube](https://img.shields.io/badge/YouTube-FF0000?style=flat-square&logo=youtube&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-26A5E4?style=flat-square&logo=telegram&logoColor=white)

**Descrição:** Sistema completo de automação para publicação de vídeos no YouTube usando n8n, Ollama (IA local) e Docker. O fluxo é acionado por agendamento, processa vídeos automaticamente e notifica resultados via Telegram.

## 🚀 Funcionalidades Principais

- **🕐 Agendamento Inteligente**: Acionamento automático via CRON
- **🎯 Seleção Aleatória**: Escolhe vídeos automaticamente do diretório
- **🤖 IA Local (Ollama)**: Gera títulos, descrições e tags otimizadas para SEO
- **📤 Upload Automático**: Posta vídeos via YouTube Data API
- **📱 Notificações**: Envia status e links via Telegram
- **🗂️ Organização**: Move arquivos processados automaticamente

## 🏗️ Arquitetura do Sistema

```
Agendamento CRON → Seleção Vídeo → Validação → IA Ollama → Formatação → Binário → Upload YouTube → Notificação Telegram → Mover Vídeo
```

## 📋 Pré-requisitos

- **Docker** (>= 20.10) e **Docker Compose**
- **Credenciais:**
  - YouTube Data API (Client ID, Secret, API Key, Refresh Token)
  - Telegram Bot Token (obtido via @BotFather)
  - Acesso ao Ollama (local ou remoto)

## 📁 Estrutura do Projeto

```
docker-n8n/
├── docker-compose.yml          # Orquestração de serviços
├── .env                       # Variáveis de ambiente (NÃO versionar)
├── README.md                  # Esta documentação
├── n8n_data/                  # Dados persistentes do n8n
│   ├── config/
│   ├── binaryData/
│   └── nodes/
├── videos_novos/              # Vídeos aguardando processamento
├── videos_postados/           # Vídeos já publicados
└── docker+n8n.json           # Backup do workflow n8n
```

## ⚙️ Configuração

### 1. Arquivo .env
Crie um arquivo `.env` na raiz:

```env
# YouTube Data API
YOUTUBE_CLIENT_ID=your_youtube_client_id_here
YOUTUBE_CLIENT_SECRET=your_youtube_client_secret_here
YOUTUBE_API_KEY=your_youtube_api_key_here
YOUTUBE_REFRESH_TOKEN=your_youtube_refresh_token_here

# Telegram (Notificações)
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
TELEGRAM_CHAT_ID=your_telegram_chat_id_here

# Ollama (IA Local)
OLLAMA_HOST=http://ollama
OLLAMA_PORT=11434
OLLAMA_MODEL=llama2

# n8n (Autenticação & Config)
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=n8nuser
N8N_BASIC_AUTH_PASSWORD=your_secure_password_here
N8N_ENCRYPTION_KEY=your-32-character-encryption-key-here
N8N_HOST=0.0.0.0
N8N_PORT=5678

# Timezone
TZ=America/Sao_Paulo
```

### 2. Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_automator
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - GENERIC_TIMEZONE=America/Sao_Paulo
    extra_hosts: 
      - "host.docker.internal:host-gateway"
    volumes:
      - ./n8n_data:/home/node/.n8n
      - ./videos_novos:/files/videos_novos
      - ./videos_postados:/files/videos_postados
  ngrok:
    image: ngrok/ngrok:latest
    container_name: ngrok_tunnel
    restart: unless-stopped
    networks:
      - default
    ports:
      - "4040:4040"
    environment:
      NGROK_AUTHTOKEN: "360Q3iaxIoF8xrVbhYizqKzfrxx_2cxeWKUriV6LBTrr6phQR" 
      NGROK_REGION: "sa" 
    command: 
      "http n8n:5678" 
networks:
  default:
    driver: bridge
```

## 🛠️ Instalação

### 1. Clone e Configuração
```bash
git clone https://github.com/rafavfa/autopost-videos-on-youtube-with-n8n.git
cd autopost-videos-on-youtube-with-n8n

# Crie o arquivo .env e configure as credenciais
cp .env.example .env  # Se disponível, ou crie manualmente
nano .env

# Crie diretórios necessários
mkdir -p videos_novos videos_postados n8n_data
```

### 2. Inicie os Serviços
```bash
docker-compose up -d
```

### 3. Acesse o n8n
- **URL:** http://localhost:5678
- **Usuário:** `n8nuser` (do .env)
- **Senha:** Sua senha configurada

## 📥 Importação do Workflow

1. Acesse o n8n em `http://localhost:5678`
2. Vá para **Workflows** → **Import**
3. Cole o JSON do workflow ou importe o arquivo `docker+n8n.json`
4. Configure as credenciais nos nós:
   - **YouTube API** (OAuth2)
   - **Telegram Bot** (Token do Bot)
   - **HTTP Request** para Ollama (`http://ollama:11434/api/generate`)

## 🎯 Como Usar

### 1. Adicione Vídeos
```bash
# Coloque vídeos no diretório de entrada
cp seu_video.mp4 videos_novos/
```

### 2. O Sistema Processa Automaticamente
- Agendamento CRON aciona o workflow
- Vídeo aleatório é selecionado
- IA gera metadados otimizados
- Upload é realizado no YouTube
- Notificação é enviada via Telegram
- Arquivo é movido para `videos_postados/`

### 3. Monitoramento
- **n8n Executions:** Verifique histórico de execuções
- **Logs:** `docker-compose logs -f n8n`
- **Notificações:** Canal do Telegram

## 🔧 Troubleshooting

| Problema | Solução |
|----------|---------|
| **n8n não inicia** | Verifique logs: `docker-compose logs n8n` |
| **Erro YouTube 401** | Valide `YOUTUBE_REFRESH_TOKEN` |
| **Ollama não responde** | Verifique: `docker-compose logs ollama` |
| **Telegram sem notificação** | Confirme `TELEGRAM_BOT_TOKEN` e `CHAT_ID` |
| **Vídeo não encontrado** | Verifique permissões em `videos_novos/` |

### Comandos Úteis
```bash
# Ver logs
docker-compose logs -f
docker-compose logs -f n8n
docker-compose logs -f ollama

# Reiniciar serviços
docker-compose restart n8n

# Parar e limpar
docker-compose down
```

## ⚠️ Boas Práticas

- **🔒 Segurança:** Nunca versionar `.env` com credenciais
- **📊 Backup:** Exporte workflows regularmente do n8n
- **📈 Monitoramento:** Acompanhe quotas da YouTube API
- **🔄 Atualização:** Mantenha imagens Docker atualizadas
- **🔍 Logs:** Monitore execuções via interface n8n

## 🐛 Solução de Problemas Comuns

### Ollama Model Não Carregado
```bash
# Acesse o container Ollama
docker exec -it ollama_ai ollama pull llama2
```

### Permissões de Arquivo
```bash
# Garanta permissões de leitura/escrita
chmod 755 videos_novos videos_postados
```

### YouTube API Quotas
- Monitorar uso em [Google Cloud Console](https://console.cloud.google.com)
- Limite padrão: 10 unidades/dia

## 🤝 Contribuição

1. Fork o projeto
2. Crie uma branch: `git checkout -b feature/nova-funcionalidade`
3. Commit: `git commit -m 'feat: nova funcionalidade'`
4. Push: `git push origin feature/nova-funcionalidade`
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob licença MIT. Veja o arquivo [LICENSE](LICENSE) para detalhes.

## ❓ Suporte

- **Documentação n8n:** https://docs.n8n.io
- **YouTube Data API:** https://developers.google.com/youtube/v3
- **Ollama:** https://ollama.ai
- **Issues:** [GitHub Issues](https://github.com/rafavfa/autopost-videos-on-youtube-with-n8n/issues)

---

### ⭐ Se este projeto foi útil, considere dar uma estrela no repositório!

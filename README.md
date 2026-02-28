# 🌤️ ClimaBot BR — Chatbot de Temperatura no Telegram

Chatbot no Telegram desenvolvido com N8N que informa a temperatura atual de qualquer cidade do Brasil. O bot recebe o nome da cidade e estado, consulta a API do OpenWeather e retorna uma mensagem amigável com a temperatura.

## Como funciona

O usuário envia uma mensagem no Telegram no formato `Cidade,UF` (ex: `João Pessoa,PB`) e o bot responde com a temperatura atual e descrição do clima. Opcionalmente, o Google Gemini pode reescrever a mensagem com linguagem mais natural e amigável.

### Fluxo do workflow

```
Telegram Trigger → Config Gemini (USE_GEMINI) → Set (Formata Entrada) → HTTP Request (OpenWeather)
    → IF (Validação: cod=200 + campos existem)
        ✅ true  → Success Response → IF (Usa Gemini?)
                                        ✅ true  → Basic LLM Chain (Gemini) → Telegram Send
                                        ❌ false → Telegram Send
        ❌ false → Erro Response → Telegram Send
```

## Pré-requisitos

- Docker Desktop instalado e rodando
- Conta no Telegram (para criar o bot via BotFather)
- Conta gratuita no [OpenWeather](https://openweathermap.org/api)
- (Opcional) Conta gratuita no [Google AI Studio](https://aistudio.google.com) para o Gemini

## Variáveis esperadas

| Variável | Onde configurar | Obrigatória |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | Credential do Telegram no n8n | Sim |
| `OPENWEATHER_API_KEY` | Credential Query Auth no n8n | Sim |
| `USE_GEMINI` | Nó "Config Gemini" no workflow | Não (padrão: `false`) |

## Importando o workflow

1. No n8n, vá em **Workflows** → **Import from File**
2. Selecione o arquivo `workflow-chatbot-telegram.json` deste repositório
3. O workflow será carregado com todos os nós configurados

## Configurando as credenciais

### Passo 1: Configurar a credential do Telegram no n8n

1. No workflow importado, clique no nó **Telegram Trigger**
2. Em **Credential to connect with**, clique em **Create New Credential**
3. No campo **Access Token**, cole o `TELEGRAM_BOT_TOKEN`
4. Clique em **Save**
5. Repita para o nó **Telegram response** — selecione a mesma credential criada

### Passo 2: Configurar a credential do OpenWeather no n8n

1. No workflow, clique no nó **OpenWeather API** (HTTP Request)
2. Em **Credential to connect with**, clique em **Create New Credential**
3. Selecione **Query Auth**
4. Preencha:
   - **Name (campo do parâmetro):** `appid`
   - **Value:** cole o valor da sua `OPENWEATHER_API_KEY`
5. Clique em **Save**

### Passo 3: Ativar o workflow

1. Clique em **Publish** (canto superior direito) para ativar o workflow
2. Envie uma mensagem para o bot no Telegram com o nome de uma cidade, ex: `Campina Grande,PB`
3. O bot responderá com a temperatura atual

## Configurando o Google Gemini (Opcional)

O nó Gemini reescreve a mensagem do bot com linguagem mais natural e amigável. Para habilitá-lo, siga os dois passos abaixo:

### Passo 1: Adicionar a credential do Gemini

1. No n8n, clique no nó **Google Gemini Chat Model** (dentro do Basic LLM Chain)
2. Em **Credential to connect with**, clique em **Create New Credential**
3. Cole a API key no campo e clique em **Save**

### Passo 2: Ativar o Gemini no workflow

1. No workflow, clique no nó **Config Gemini** (primeiro nó após o Telegram Trigger)
2. No campo **USE_GEMINI**, altere o valor de `false` para `true`
3. Salve o workflow (teste e depois publique novamente)

Para desativar o Gemini, basta voltar o campo para `false`.

**Configuração recomendada do modelo:**
- Modelo: `gemini-2.5-flash-lite`
- Temperature: `0.2` (respostas mais consistentes)

## Exemplo de uso

**Entrada do usuário:**
```
Campina Grande,PB
```

**Resposta (sem Gemini — USE_GEMINI: false):**
```
🌡️ Agora em Campina Grande, a temperatura é de 29°C com céu limpo.
```

**Resposta (com Gemini — USE_GEMINI: true):**
```
🌤️ Campina Grande tá com 29°C e céu limpo! Dia perfeito pra uma corridinha. 🏃‍♀️
```

## Tratamento de erros

- Cidade não encontrada → `Cidade [nome] não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP)`
- Resposta da API incompleta (campos ausentes) → mesmo tratamento de erro
- Gemini desativado → a mensagem do fallback (Success Response) é enviada diretamente

## Observações

- **Não inclua chaves de API no repositório.** As credenciais devem ser configuradas diretamente no n8n conforme as instruções acima.
- A API gratuita do OpenWeather permite até 60 chamadas por minuto.
- O Gemini gratuito tem rate limit — se retornar erro 429, aguarde alguns minutos ou desative o Gemini.

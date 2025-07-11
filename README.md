# 🌦️ mvp-previsao-do-tempo

Este fluxo desenvolvido em **n8n** oferece aos usuários uma experiência automatizada via **WhatsApp** para consultar a previsão do tempo e ativar/desativar alertas automáticos para mudanças climáticas bruscas. Ele integra as APIs do WhatsApp (Evolution API), Supabase, WeatherAPI e OpenAI (para tradução e formatação).

---

## 📌 Objetivo

Permitir que usuários:
- Consultem a previsão atual do tempo informando uma cidade;
- Ativem ou recusem o recebimento de **alertas automáticos** de clima;

---

## ⚙️ Tecnologias Utilizadas

- **n8n**: Plataforma de automação para orquestração do fluxo
- **Supabase**: Banco de dados (PostgreSQL) para armazenar usuários e alertas
- **WeatherAPI**: Fonte de dados meteorológicos
- **Evolution API**: Conexão com o WhatsApp
- **OpenAI GPT-4.1-mini**: Tradução e formatação dos alertas em linguagem natural

---

## 🔄 Fluxo Principal

### 1. **Gatilho**
- O fluxo inicia com um Webhook que escuta mensagens do WhatsApp via Evolution API.

### 2. **Identificação do Usuário**
- Extrai número de telefone e nome.
- Verifica se o usuário já existe no Supabase.
  - Se não existir, cria um novo registro.

### 3. **Tratamento de Mensagens**
- Analisa se a mensagem recebida é texto simples, ou um dos comandos:
  - `1` para **Aceitar** alertas automáticos
  - `2` para **Recusar**

### 4. **Consulta da Previsão do Tempo**
- Utiliza a WeatherAPI para buscar os dados meteorológicos da cidade informada.
- Retorna mensagem com:
  - Condição do tempo
  - Temperatura
  - Umidade
  - Pergunta se o usuário deseja receber alertas automáticos.

### 5. **Gerenciamento de Alertas**
- Se o usuário aceita (`1`), o alerta é ativado (ou criado) no banco Supabase.
- Se recusa (`2`), a flag `alertaAtivo` é desativada.
- Ambos os casos resultam em mensagem de confirmação no WhatsApp.

---

## ⏰ Agendamento Diário

- O nó `Schedule Trigger` executa a cada 1 hora.
- O fluxo:
  - Verifica todos os usuários com `alertaAtivo = true`.
  - Consulta os alertas ativos via WeatherAPI.
  - Gera mensagem humanizada com auxílio da IA.
  - Valida se a **mensagem é nova** (compara `qtd_alertas`).
  - Atualiza o campo `qtd_alertas` no Supabase.
  - Envia o alerta somente se houver mudança.

---

## 🧠 Agente de IA

- A OpenAI GPT é usada para:
  - Traduzir mensagens técnicas da WeatherAPI, uma vez que, mesmo ao definir o parâmetro lang=pt as informações continuam sendo entregues em inglês. 

---

## 🗃️ Estrutura de Tabelas Supabase

### `usuarios`
| Campo     | Tipo     | Descrição                  |
|-----------|----------|----------------------------|
| nome      | string   | Nome do contato            |
| whatsapp  | string   | Número de telefone (ID)    |

### `alertas`
| Campo        | Tipo         | Descrição                               |
|--------------|--------------|-----------------------------------------|
| whatsapp     | string       | Referência ao usuário                   |
| cidade       | string       | Nome da cidade para previsão            |
| alertaAtivo  | boolean      | Define se o alerta está ativo           |
| qtd_alertas  | string       | Quantidade de alertas identificados     |
| updated_at   | timestamptz  | Informa última alteração                |

---

## 📬 Mensagens Automatizadas

### ✅ Aceite:
> "✅ Perfeito! Você agora receberá alertas automáticos sobre mudanças climáticas em **[cidade]**."

### ❌ Recusa:
> "Ok, sem problemas. Se quiser receber alertas depois, é só avisar!"

### ⚠️ Cidade não encontrada:
> "Ops! 😕 Não consegui encontrar a previsão do tempo para a cidade que você informou. Verifique se o nome está correto e tente novamente."
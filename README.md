# 🌦️ mvp-previsao-do-tempo

Este fluxo desenvolvido em **n8n** oferece aos usuários uma experiência automatizada via **WhatsApp** para consultar a previsão do tempo e ativar/desativar alertas automáticos para mudanças climáticas bruscas. Ele integra as APIs do WhatsApp (Evolution API), Supabase, WeatherAPI e OpenAI (para tradução e formatação).

---

## 📌 Objetivo

Permitir que usuários:
- Consultem a previsão atual do tempo informando uma cidade;
- Ativem ou recusem o recebimento de **alertas automáticos** de clima;
- Recebam alertas traduzidos e formatados diretamente no WhatsApp.

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

- O nó `Schedule Trigger` executa diariamente às 08:00.
- O fluxo:
  - Consulta todos os usuários com `alertaAtivo = true`
  - Verifica se há alertas climáticos severos via `WeatherAPI`
  - Traduz e formata a mensagem com ajuda da OpenAI
  - Envia para os respectivos usuários via Evolution API

---

## 🧠 Agente de IA

- A OpenAI GPT é usada para:
  - Traduzir mensagens técnicas da WeatherAPI
  - Manter emojis, clareza e instruções práticas

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
| alertaAtivo  | boolean      | Flag indicando se o alerta está ativo   |
| updated_at   | timestamptz  | Informa última alteração                |

---

## 📬 Mensagens Automatizadas

### ✅ Aceite:
> "✅ Perfeito! Você agora receberá alertas automáticos sobre mudanças climáticas em **[cidade]**."

### ❌ Recusa:
> "Ok, sem problemas. Se quiser receber alertas depois, é só avisar!"

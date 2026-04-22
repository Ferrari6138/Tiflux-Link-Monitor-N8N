# 🔗 Monitoramento de Links — TiFlux + Zabbix/Ubiquiti + WhatsApp

Workflow n8n para monitoramento automático de alertas de conectividade. Detecta tickets de queda de link abertos no **TiFlux** (originados por **Zabbix** ou **Ubiquiti**), notifica a equipe via **WhatsApp** e acompanha o restabelecimento automaticamente.

---

## 🔁 Arquitetura Geral

O workflow possui **dois fluxos independentes** acionados por schedules separados:

```
[FLUXO 1 — Polling de Tickets]
Trigger (1 min)
  └── Buscar Tickets TiFlux
        └── Filtrar por título (Link Inoperante / Internet Down)
              └── Classificar tipo e grupo de cliente
                    └── Loop por ticket
                          └── Resolver tabela Baserow (Link ou WAN)
                                └── Deve Processar?
                                      ├── Buscar Detalhes → Formatar Mensagem → Enviar WhatsApp
                                      └── Preparar Dados → Consultar Baserow → Criar Row (se novo)

[FLUXO 2 — Verificação de Fechamento]
Trigger (1 min)
  └── Buscar tickets abertos no Baserow (Link e WAN em paralelo)
        └── Loop por ticket
              └── Consultar TiFlux → Ticket fechado?
                    ├── SIM → Notificar restabelecimento → Marcar como resolvido no Baserow
                    └── NÃO → Continuar loop
```

---

## 💡 Tipos de Alerta Suportados

| Tipo | Título do Ticket | Fonte | Tabela Baserow |
|---|---|---|---|
| `link_inoperante` | `Link de Internet - Inoperante` | Zabbix | `{{BASEROW_TABLE_LINK_ID}}` |
| `internet_down` | `Internet Down` | Ubiquiti | `{{BASEROW_TABLE_WAN_ID}}` |

---

## ⚙️ Configuração — Placeholders

### TiFlux

| Placeholder | Descrição |
|---|---|
| `{{TIFLUX_CREDENTIAL_ID}}` | ID da credencial Bearer Token do TiFlux no n8n |
| `{{TIFLUX_CREDENTIAL_NAME}}` | Nome da credencial do TiFlux no n8n |

### Baserow

| Placeholder | Descrição |
|---|---|
| `{{BASEROW_CREDENTIAL_ID}}` | ID da credencial Baserow no n8n |
| `{{BASEROW_CREDENTIAL_NAME}}` | Nome da credencial Baserow no n8n |
| `{{BASEROW_DATABASE_ID}}` | ID do banco de dados no Baserow |
| `{{BASEROW_TABLE_LINK_ID}}` | ID da tabela para tickets de Link Inoperante |
| `{{BASEROW_TABLE_WAN_ID}}` | ID da tabela para tickets de Internet Down |
| `{{BASEROW_FIELD_TICKET_LINK}}` | ID do campo de número de ticket (tabela Link) |
| `{{BASEROW_FIELD_TICKET_WAN}}` | ID do campo de número de ticket (tabela WAN) |
| `{{BASEROW_FIELD_CLIENT_LINK}}` | ID do campo de nome do cliente (tabela Link) |
| `{{BASEROW_FIELD_CLIENT_WAN}}` | ID do campo de nome do cliente (tabela WAN) |
| `{{BASEROW_FIELD_CREATED_LINK}}` | ID do campo de data de criação (tabela Link) |
| `{{BASEROW_FIELD_CREATED_WAN}}` | ID do campo de data de criação (tabela WAN) |
| `{{BASEROW_FIELD_STATUS_LINK}}` | ID do campo de status/fechamento (tabela Link) |
| `{{BASEROW_FIELD_STATUS_WAN}}` | ID do campo de status/fechamento (tabela WAN) |
| `{{BASEROW_FIELD_NOTIFIED_LINK}}` | ID do campo de notificação enviada (tabela Link) |

### WhatsApp (UazAPI)

| Placeholder | Descrição |
|---|---|
| `{{UAZAPI_ENDPOINT_URL}}` | URL completa do endpoint com token. Formato: `https://SEU_SUBDOMINIO.uazapi.com/send/text?token=SEU_TOKEN` |
| `{{MEMBER_1_NAME}}` | Nome do membro da equipe |
| `{{MEMBER_1_WHATSAPP}}` | Número WhatsApp em formato internacional sem `+` (ex: `5511999999999`) |

> Para adicionar mais membros, duplique a linha `{ nome: ..., numero: ... }` no array `equipe` dos nós de código.

### Grupos de Clientes

| Placeholder | Descrição |
|---|---|
| `{{CLIENT_GROUP_REGEX}}` | Regex com os nomes dos clientes que recebem tratamento prioritário. Ex: `ClienteA\|ClienteB\|ClienteC` |

> Tickets de `link_inoperante` só são processados para clientes que correspondem a este regex. Tickets de `internet_down` são sempre processados.

### n8n

| Placeholder | Descrição |
|---|---|
| `{{N8N_INSTANCE_ID}}` | ID da instância n8n (opcional) |

---

## 🗄️ Estrutura das Tabelas no Baserow

Crie **duas tabelas** com estrutura similar:

**Tabela Link Inoperante:**

| Campo | Tipo | Descrição |
|---|---|---|
| `ID_Ticket` | Texto | Número do ticket no TiFlux |
| `Cliente` | Texto | Nome do cliente |
| `Data Abertura` | Data/Hora | Timestamp de criação do ticket |
| `Resolvido` | Booleano | `false` = aberto, `true` = resolvido |
| `Notificado` | Booleano | Se o alerta foi enviado |

**Tabela Internet Down** — estrutura idêntica, sem o campo `Notificado`.

---

## 📋 Como importar no n8n

1. Importe o arquivo `tiflux-link-monitor-whatsapp.json` no n8n
2. Configure a credencial Bearer Token para o TiFlux
3. Configure a credencial da API do Baserow
4. Substitua todos os placeholders pelos valores reais
5. Ative os dois triggers: **Polling Tickets** e **Verificar Fechamento**

---

## 🛠️ Requisitos

- n8n v1.0+
- Conta TiFlux com acesso à API v2
- Baserow (self-hosted ou cloud) com duas tabelas configuradas
- Instância UazAPI conectada ao WhatsApp

---

## 📁 Arquivos

```
.
├── tiflux-link-monitor-whatsapp.json   # Workflow principal
└── README.md
```

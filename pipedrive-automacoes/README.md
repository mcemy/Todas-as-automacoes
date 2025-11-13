# 🔗 Automações Pipedrive

Esta pasta contém automações específicas para integração com o Pipedrive CRM seguindo o padrão: `pipedrive-automacoes-[nome-da-funcionalidade]`

## 🎯 Tipos de Automações Pipedrive

### 1. Sincronização de Dados

- **Importação de leads**: Planilhas → Pipedrive
- **Exportação de deals**: Pipedrive → Relatórios/Planilhas
- **Sincronização bidirecional**: Manter dados atualizados
- **Backup automático**: Backup periódico dos dados

### 2. Automações de Processo

- **Criação automática de atividades**: Baseada em eventos
- **Movimentação de deals**: Automation de pipeline
- **Notificações personalizadas**: Alertas customizados
- **Relatórios automáticos**: Geração e envio periódico

### 3. Integrações via Webhook

- **Notificações em tempo real**: Eventos instantâneos
- **Sincronização imediata**: Atualização automática de planilhas
- **Triggers para outros sistemas**: Disparo de ações externas
- **Logs de atividades**: Registro detalhado de mudanças

## 📁 Estrutura Padrão

```text
pipedrive-automacoes-[nome]/
├── README.md              # Documentação específica
├── Code.gs                # Código principal (use template PipedriveCode.gs)
├── appsscript.json        # Manifesto do Apps Script
├── config/
│   └── pipedrive_config.gs # Configurações específicas do Pipedrive
├── utils/
│   └── pipedrive_utils.gs  # Funções auxiliares para API
├── webhooks/
│   └── webhook_handlers.gs # Handlers específicos por evento
├── tests/
│   └── pipedrive_tests.gs  # Testes da integração
├── screenshots/           # Capturas de tela
├── .env.example          # Exemplo de configuração
└── CHANGELOG.md          # Histórico de versões
```

## 🔧 Configuração de Webhooks

### 1. Configuração no Google Apps Script

```javascript
/**
 * URL do webhook: https://script.google.com/macros/s/SEU_ID_DO_SCRIPT/exec
 */
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    processarEventoPipedrive(data);

    return ContentService.createTextOutput("Success").setMimeType(
      ContentService.MimeType.TEXT
    );
  } catch (error) {
    Logger.log("Erro no webhook:", error.toString());
    return ContentService.createTextOutput("Error").setMimeType(
      ContentService.MimeType.TEXT
    );
  }
}
```

### 2. Configuração no Pipedrive

1. **Acesse**: Settings → Webhooks
2. **URL do endpoint**: `https://script.google.com/macros/s/SEU_ID_DO_SCRIPT/exec`
3. **Eventos**: Selecione os eventos necessários
4. **Teste**: Use a função de teste do Pipedrive

### 3. Eventos Comuns

- `deal.created` - Novo deal criado
- `deal.updated` - Deal atualizado
- `deal.deleted` - Deal removido
- `person.created` - Nova pessoa criada
- `person.updated` - Pessoa atualizada
- `activity.created` - Nova atividade criada

## ⚠️ Limites e Considerações

### Limites da API Pipedrive

| Tipo                | Limite                   |
| ------------------- | ------------------------ |
| **Requests/dia**    | 10.000 (planos pagos)    |
| **Rate Limiting**   | ~100 requests/minuto     |
| **Timeout webhook** | 10 segundos              |
| **Retry webhook**   | 3 tentativas automáticas |

### Limites do Google Apps Script

| Tipo                     | Limite        |
| ------------------------ | ------------- |
| **Execução máxima**      | 6 minutos     |
| **Triggers simultâneos** | 20 por script |
| **Calls UrlFetchApp**    | 20.000/dia    |

### Exemplo de Rate Limiting

```javascript
function fazerRequisicaoSegura(url, opcoes) {
  const maxTentativas = 3;
  let tentativa = 0;

  while (tentativa < maxTentativas) {
    try {
      const response = UrlFetchApp.fetch(url, opcoes);

      if (response.getResponseCode() === 429) {
        Logger.log("Rate limit - aguardando...");
        Utilities.sleep(2000 * (tentativa + 1));
        tentativa++;
        continue;
      }

      return response;
    } catch (error) {
      tentativa++;
      if (tentativa >= maxTentativas) throw error;
      Utilities.sleep(1000 * tentativa);
    }
  }
}
```

## 🚨 Gestão de Webhooks com Erro

### ⚠️ IMPORTANTE: Remoção de Webhooks Problemáticos

Webhooks que retornam erro consistentemente **DEVEM SER REMOVIDOS** para evitar:

- **Sobrecarga do sistema**: Tentativas excessivas
- **Logs poluídos**: Spam nos logs do Pipedrive
- **Possível bloqueio**: Pipedrive pode desabilitar a conta

### Monitoramento de Erros

```javascript
function monitorarWebhooks() {
  const properties = PropertiesService.getScriptProperties();
  const erros = parseInt(properties.getProperty("WEBHOOK_ERROS") || "0");

  if (erros > 10) {
    // Alertar administrador
    enviarAlertaAdmin();

    // Considerar desabilitar webhook
    Logger.log("ALERTA: Muitos erros - webhook deve ser revisado");
  }
}

function registrarErroWebhook() {
  const properties = PropertiesService.getScriptProperties();
  const atual = parseInt(properties.getProperty("WEBHOOK_ERROS") || "0");
  properties.setProperty("WEBHOOK_ERROS", (atual + 1).toString());
}
```

### Quando Remover Webhooks

1. **+15 erros consecutivos**: Revisar código urgentemente
2. **+25 erros consecutivos**: Desabilitar webhook temporariamente
3. **Erros de timeout**: Otimizar processamento ou usar processamento assíncrono
4. **Erros de parsing**: Verificar formato dos dados recebidos

## 🔄 Redirecionamento para Webhooks

### Requisitos do Pipedrive

O endpoint do webhook DEVE:

1. **Responder em até 10 segundos**: Timeout automático após isso
2. **Retornar status 2xx**: Para marcar como entregue com sucesso
3. **Processar ou delegar**: Para operações longas, use processamento assíncrono

### Exemplo de Resposta Adequada

```javascript
function doPost(e) {
  const startTime = new Date();

  try {
    const data = JSON.parse(e.postData.contents);

    // Para processamento rápido (< 8 segundos)
    if (isProcessamentoRapido(data)) {
      processarImediatamente(data);
    } else {
      // Para processamento longo, salvar e processar depois
      salvarParaProcessamentoPosterior(data);
      ScriptApp.newTrigger("processarAssincrono")
        .timeBased()
        .after(1000) // 1 segundo após
        .create();
    }

    // Sempre responder rapidamente
    return ContentService.createTextOutput("Success");
  } catch (error) {
    Logger.log("Erro:", error.toString());
    registrarErroWebhook();

    // Retornar erro para que Pipedrive tente novamente
    return ContentService.createTextOutput("Error");
  }
}
```

## 🛠️ Templates Disponíveis

### 1. Sincronização Básica

```bash
pipedrive-automacoes-sincronizacao-leads/
├── Code.gs                    # Sync periódico via trigger
├── config/pipedrive_sync.gs   # Configurações de sincronização
└── README.md                  # Documentação específica
```

### 2. Webhook em Tempo Real

```bash
pipedrive-automacoes-webhook-notificacoes/
├── Code.gs                    # Handler do webhook
├── webhooks/event_handlers.gs # Processadores por evento
└── README.md                  # Documentação específica
```

### 3. Backup Automático

```bash
pipedrive-automacoes-backup-dados/
├── Code.gs                    # Backup periódico
├── utils/backup_utils.gs      # Funções de backup
└── README.md                  # Documentação específica
```

## 📋 Permissões Necessárias

```json
{
  "oauthScopes": [
    "https://www.googleapis.com/auth/script.external_request",
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/gmail.send"
  ]
}
```

## 💡 Boas Práticas

### Segurança

- ✅ Use PropertiesService para API tokens
- ✅ Valide dados recebidos nos webhooks
- ✅ Implemente retry com backoff exponencial
- ✅ Monitore erros e performance

### Performance

- ✅ Processe webhooks em menos de 8 segundos
- ✅ Use processamento assíncrono para operações pesadas
- ✅ Implemente cache quando possível
- ✅ Monitore rate limits da API

### Monitoramento

- ✅ Log todas as operações importantes
- ✅ Configure alertas para erros
- ✅ Monitore contadores de erro de webhook
- ✅ Teste conexões periodicamente

## 🚀 Exemplos de Uso

### Sincronização de Leads

```javascript
function sincronizarLeadsPlanilha() {
  const deals = buscarDealsRecentes();
  const planilha = obterPlanilhaDestino();

  deals.forEach((deal) => {
    adicionarLinhaPlanilha(planilha, deal);
  });

  Logger.log(`${deals.length} deals sincronizados`);
}
```

### Notificação de Novo Deal

```javascript
function processarNovoDeal(dealData) {
  const mensagem = `
    Novo deal criado: ${dealData.title}
    Valor: ${dealData.value} ${dealData.currency}
    Cliente: ${dealData.person_name}
  `;

  enviarNotificacaoSlack(mensagem);
  adicionarLinhaPlanilha(dealData);
}
```

## 📚 Recursos Úteis

- [Pipedrive API Documentation](https://developers.pipedrive.com/docs/api/v1)
- [Pipedrive Webhooks Guide](https://developers.pipedrive.com/docs/webhooks)
- [Google Apps Script UrlFetchApp](https://developers.google.com/apps-script/reference/url-fetch)

## ⚠️ Troubleshooting

### Webhook não recebe dados

1. Verificar URL do webhook no Pipedrive
2. Confirmar que o script está publicado como Web App
3. Testar webhook manualmente no Pipedrive

### Rate limit atingido

1. Implementar delays entre requests
2. Usar processamento em lotes
3. Considerar upgrade do plano Pipedrive

### Timeout no webhook

1. Otimizar processamento
2. Usar processamento assíncrono
3. Responder imediatamente e processar depois

---

💡 **Dica**: Sempre teste integrações com dados de desenvolvimento antes de usar em produção!
💡 **AVISO**: O pipedrive não aceita o link direto do appscript, ele PRECISA de um link de redirecionamento.

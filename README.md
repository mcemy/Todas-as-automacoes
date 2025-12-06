# 🤖 Automações Google Apps Script

Repositório de **exemplos** de automações desenvolvidas em **JavaScript** usando **Google Apps Script** para integração com diversos serviços.

## 📋 Categorias de Automações

### 📊 [Sheets](./sheets-automacoes/)

- Relatórios automatizados
- Processamento de dados
- Sincronização entre planilhas
- Dashboards dinâmicos

### 📧 [Gmail](./gmail-automacoes/)

- Envio automático de emails
- Processamento de anexos
- Filtros inteligentes
- Notificações personalizadas

### 📁 [Drive](./drive-automacoes/)

- Organização de arquivos
- Backup automático
- Conversão de documentos
- Gestão de permissões

### 📝 [Forms](./forms-automacoes/)

- Processamento de respostas
- Validação de dados
- Integração com planilhas
- Geração de certificados

### 🔗 [Pipedrive](./pipedrive-automacoes/)

- Sincronização de CRM
- Webhooks em tempo real
- Automação de pipeline
- Relatórios de vendas

### 🌐 [Web](./automacoes-web/)

- Web scraping
- Automação de navegadores
- Integração com sites
- Coleta de dados

### 🔌 [API](./automacoes-api/)

- Integração com APIs REST
- Sincronização de dados
- Middleware de sistemas
- Processamento em lote

## 🛠️ Tecnologias Utilizadas

- **Linguagem**: JavaScript (Google Apps Script)
- **Plataforma**: Google Apps Script
- **Serviços**: Google Workspace, APIs externas
- **Triggers**: Tempo, eventos, webhooks

## 📝 Padrões de Desenvolvimento

### Estrutura de Nomenclatura

```
[categoria]-automacoes-[funcionalidade]
```

**Exemplos:**

- `sheets-automacoes-relatorio-vendas`
- `pipedrive-automacoes-sync-leads`
- `gmail-automacoes-backup-anexos`

### Boas Práticas

- ✅ Use `PropertiesService` para dados sensíveis
- ✅ Inclua tratamento de erros completo
- ✅ Documente todas as funções
- ✅ Crie funções de teste
- ✅ Use logs detalhados (`Logger.log`)
- ✅ Teste antes de fazer commit
- ⚠️ **Pipedrive**: Teste webhooks em apenas **1 negócio**

### Estrutura de Código

```javascript
/**
 * Automação: [Nome da Funcionalidade]
 *
 */

// Configurações
const CONFIG = {
  // Use PropertiesService para dados sensíveis
  SHEET_ID: PropertiesService.getScriptProperties().getProperty("SHEET_ID"),
  // Constantes podem ser hardcoded
  TIMEOUT: 30000,
};

// Função principal
function main() {
  try {
    Logger.log("Iniciando automação...");
    // Sua lógica aqui
  } catch (error) {
    Logger.log("Erro:", error.toString());
    throw error;
  }
}

// Configuração inicial
function setup() {
  const properties = PropertiesService.getScriptProperties();
  properties.setProperties({
    SHEET_ID: "SEU_ID_AQUI",
  });
}
```

## 🔒 Segurança

### ❌ NUNCA Faça Commit De:

- IDs reais de planilhas/documentos
- Emails corporativos
- API keys ou tokens
- Senhas ou credenciais
- URLs com dados sensíveis
- Domínios de produção

### ✅ Sempre Use:

- `PropertiesService` para dados sensíveis
- Arquivos `.env.example` com exemplos
- Validação de configurações
- Logs sem dados sensíveis

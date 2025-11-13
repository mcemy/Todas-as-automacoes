# 📊 Automações Google Sheets

Esta pasta contém automações específicas para Google Sheets seguindo o padrão: `sheets-automacoes-[nome-da-funcionalidade]`

## 🎯 Tipos de Automações

### Relatórios Automatizados

- Geração automática de relatórios
- Consolidação de dados de múltiplas abas
- Envio automático por email
- Formatação condicional

### Processamento de Dados

- Limpeza e normalização de dados
- Cálculos complexos automatizados
- Validação de informações
- Import/Export de dados

### Integrações

- Sincronização com outros sistemas
- APIs de terceiros
- Webhooks e triggers
- Backup automático

## 📁 Estrutura Padrão

```text
sheets-automacoes-[nome]/
├── README.md              # Documentação específica
├── Code.gs                # Código principal
├── appsscript.json        # Manifesto do Apps Script
├── config/
│   └── config.gs          # Configurações
├── utils/
│   └── helpers.gs         # Funções auxiliares
├── tests/
│   └── tests.gs           # Testes
├── screenshots/           # Capturas de tela
├── .env.example          # Exemplo de configuração
└── CHANGELOG.md          # Histórico de versões
```

## 🔧 Funções Comuns do Sheets

### Acesso a Dados

```javascript
// Abrir planilha
const sheet = SpreadsheetApp.openById("ID").getSheetByName("Aba");

// Ler dados
const dados = sheet.getRange("A1:Z100").getValues();

// Escrever dados
sheet.getRange("A1").setValue("Novo valor");
```

### Formatação

```javascript
// Formatação condicional
const range = sheet.getRange("A1:A10");
range.setBackground("#ff0000");
range.setFontWeight("bold");
```

### Fórmulas

```javascript
// Inserir fórmula
sheet.getRange("B1").setFormula("=SUM(A1:A10)");
```

## 📋 Permissões Necessárias

- `https://www.googleapis.com/auth/spreadsheets`
- `https://www.googleapis.com/auth/drive` (se acessar múltiplas planilhas)

## 💡 Boas Práticas

- Use `getValues()` para ler múltiplas células de uma vez
- Minimize chamadas à API do Sheets
- Implemente cache quando possível
- Use `flush()` para forçar gravação imediata
- Trate erros de acesso e permissão

## 🚀 Templates Disponíveis

- **Relatório de Vendas**: Consolida dados de vendas e envia por email
- **Dashboard Automático**: Atualiza dashboard com dados em tempo real
- **Backup de Dados**: Faz backup automático de planilhas importantes
- **Validador de Dados**: Valida e limpa dados importados

---

📚 **Recursos**: [Documentação Google Sheets API](https://developers.google.com/sheets/api)

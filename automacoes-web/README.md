# 🌐 Automações Web

Automações em **JavaScript** usando **Google Apps Script** para interação com sites e coleta de dados web.

## 🚀 Funcionalidades

### 🕷️ Web Scraping

- Extração de dados de sites públicos
- Parsing de HTML e XML
- Coleta de preços e informações
- Monitoramento de mudanças

### 📊 Monitoramento de Sites

- Verificação de disponibilidade
- Alertas de mudança de conteúdo
- Monitoramento de status codes
- Relatórios de uptime

### 🔍 Análise de Conteúdo

- Extração de metadados
- Análise de SEO básico
- Captura de informações específicas
- Validação de links

### 📈 Coleta de Dados Públicos

- Informações de mercado
- Dados governamentais abertos
- RSS feeds
- APIs públicas sem autenticação

## 🛠️ APIs Utilizadas

- **UrlFetchApp**: Requisições HTTP para sites
- **XmlService**: Parsing de XML e HTML básico
- **Utilities**: Processamento de strings e regex
- **SpreadsheetApp**: Armazenamento de dados coletados
- **MailApp**: Notificações de mudanças

## Template de Estrutura

```
nome-da-automacao-web/
├── src/
│   ├── scraper.py          # Lógica principal
│   ├── config.py           # Configurações
│   └── utils.py            # Funções auxiliares
├── data/
│   ├── input/              # Dados de entrada
│   └── output/             # Resultados
├── logs/                   # Arquivos de log
├── screenshots/            # Capturas de tela (opcional)
├── requirements.txt        # Dependências
├── .env.example           # Exemplo de variáveis
└── README.md              # Documentação
```

---

💡 **Dica**: Sempre respeite o `robots.txt` dos sites e implemente delays apropriados para não sobrecarregar os servidores.

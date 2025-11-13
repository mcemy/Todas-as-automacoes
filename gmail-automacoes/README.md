# 📧 Automações Gmail

Esta pasta contém automações específicas para Gmail seguindo o padrão: `gmail-automacoes-[nome-da-funcionalidade]`

## 🎯 Tipos de Automações

### Envio Automático

- Envio de relatórios periódicos
- Notificações baseadas em eventos
- Newsletters automatizadas
- Lembretes e follow-ups

### Processamento de Emails

- Classificação automática
- Respostas automáticas
- Extração de dados de emails
- Arquivamento inteligente

### Monitoramento

- Alertas de emails importantes
- Monitoramento de caixa de entrada
- Estatísticas de email
- Backup de emails

## 📁 Estrutura Padrão

```text
gmail-automacoes-[nome]/
├── README.md              # Documentação específica
├── Code.gs                # Código principal
├── appsscript.json        # Manifesto do Apps Script
├── templates/
│   ├── email_template.html # Template HTML de email
│   └── email_template.txt  # Template texto de email
├── config/
│   └── config.gs          # Configurações
├── utils/
│   └── email_utils.gs     # Funções auxiliares
├── tests/
│   └── tests.gs           # Testes
├── screenshots/           # Capturas de tela
├── .env.example          # Exemplo de configuração
└── CHANGELOG.md          # Histórico de versões
```

## 🔧 Funções Comuns do Gmail

### Envio de Emails

```javascript
// Email simples
GmailApp.sendEmail("destinatario@email.com", "Assunto", "Corpo do email");

// Email com anexo
const arquivo = DriveApp.getFileById("ID_DO_ARQUIVO");
GmailApp.sendEmail("destinatario@email.com", "Assunto", "Corpo do email", {
  attachments: [arquivo.getBlob()],
  htmlBody: "<p>Email em HTML</p>",
});
```

### Leitura de Emails

```javascript
// Buscar emails
const threads = GmailApp.search("from:remetente@email.com");

// Ler mensagens
threads.forEach((thread) => {
  const messages = thread.getMessages();
  messages.forEach((message) => {
    console.log(message.getSubject());
    console.log(message.getBody());
  });
});
```

### Organização

```javascript
// Aplicar labels
const label = GmailApp.getUserLabelByName("Importante");
thread.addLabel(label);

// Arquivar
thread.moveToArchive();

// Marcar como lido
thread.markRead();
```

## 📋 Permissões Necessárias

- `https://www.googleapis.com/auth/gmail.send`
- `https://www.googleapis.com/auth/gmail.readonly`
- `https://www.googleapis.com/auth/gmail.modify` (para organização)

## 💡 Boas Práticas

- Respeite limites de envio (100 emails/dia para contas gratuitas)
- Use templates HTML para emails formatados
- Implemente retry para falhas de envio
- Monitore bounces e erros
- Use BCC para múltiplos destinatários

## 🚀 Templates Disponíveis

- **Relatório por Email**: Envia relatórios automaticamente
- **Notificações de Sistema**: Alertas de eventos importantes
- **Resposta Automática**: Respostas baseadas em conteúdo
- **Backup de Anexos**: Salva anexos automaticamente no Drive

## ⚠️ Limitações

- **Cota diária**: 100 emails/dia (conta gratuita)
- **Tamanho**: 25MB por email (incluindo anexos)
- **Rate limit**: Evite envios muito rápidos seguidos

---

📚 **Recursos**: [Documentação Gmail API](https://developers.google.com/gmail/api)

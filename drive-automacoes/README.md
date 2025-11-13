# 📁 Automações Google Drive

Automações em **JavaScript** usando **Google Apps Script** para gerenciamento de arquivos no Google Drive.

## 🚀 Funcionalidades

### 📂 Organização de Arquivos

- Classificação automática por tipo
- Estruturação de pastas
- Limpeza de arquivos duplicados
- Movimentação baseada em regras

### 💾 Backup Automático

- Backup de planilhas importantes
- Sincronização entre contas
- Versionamento de documentos
- Restauração automática

### 🔄 Conversão de Documentos

- PDF para texto
- Planilhas para JSON
- Imagens para texto (OCR)
- Formatos de arquivo

### 🔐 Gestão de Permissões

- Auditoria de compartilhamentos
- Aplicação de políticas de segurança
- Notificações de alterações
- Controle de acesso

## 🛠️ APIs Utilizadas

- **DriveApp**: Manipulação de arquivos e pastas
- **Utilities**: Conversões e utilitários
- **MailApp**: Notificações por email
- **SpreadsheetApp**: Integração com planilhas

## 📋 Exemplo: Organização Automática

```javascript
/**
 * Organiza arquivos por tipo em pastas específicas
 * Execução: Trigger diário às 06:00
 */
function organizarArquivos() {
  try {
    Logger.log("Iniciando organização de arquivos...");

    const pastaRaiz = DriveApp.getFolderById(CONFIG.PASTA_RAIZ_ID);
    const arquivos = pastaRaiz.getFiles();

    let contador = 0;

    while (arquivos.hasNext()) {
      const arquivo = arquivos.next();
      const tipoMime = arquivo.getBlob().getContentType();

      // Determina pasta destino baseada no tipo MIME
      const pastaDestino = obterPastaDestino(tipoMime);

      if (pastaDestino && !arquivoJaEstaNaPasta(arquivo, pastaDestino)) {
        moverArquivo(arquivo, pastaDestino);
        contador++;

        Logger.log(`Movido: ${arquivo.getName()} -> ${pastaDestino.getName()}`);
      }
    }

    enviarRelatorio(contador);
    Logger.log(`Organização concluída. ${contador} arquivos movidos.`);
  } catch (error) {
    Logger.log("Erro na organização:", error.toString());
    enviarNotificacaoErro(error);
  }
}

/**
 * Determina pasta destino baseada no tipo MIME
 */
function obterPastaDestino(tipoMime) {
  const mapeamento = {
    "application/pdf": CONFIG.PASTA_PDF_ID,
    "image/": CONFIG.PASTA_IMAGENS_ID,
    "application/vnd.google-apps.spreadsheet": CONFIG.PASTA_PLANILHAS_ID,
    "application/vnd.google-apps.document": CONFIG.PASTA_DOCUMENTOS_ID,
  };

  for (const [tipo, pastaId] of Object.entries(mapeamento)) {
    if (tipoMime.includes(tipo)) {
      return DriveApp.getFolderById(pastaId);
    }
  }

  return null; // Não move se não encontrar pasta adequada
}

/**
 * Move arquivo para pasta destino
 */
function moverArquivo(arquivo, pastaDestino) {
  const pastasAtuais = arquivo.getParents();

  // Adiciona à nova pasta
  pastaDestino.addFile(arquivo);

  // Remove das pastas antigas
  while (pastasAtuais.hasNext()) {
    const pastaAtual = pastasAtuais.next();
    pastaAtual.removeFile(arquivo);
  }
}
```

## 📋 Exemplo: Backup Inteligente

```javascript
/**
 * Backup automático de arquivos importantes
 * Execução: Trigger semanal aos domingos
 */
function backupArquivos() {
  try {
    Logger.log("Iniciando backup de arquivos...");

    const pastaBackup = obterPastaBackup();
    const arquivosImportantes = identificarArquivosImportantes();

    let backupsRealizados = 0;

    arquivosImportantes.forEach((arquivo) => {
      if (precisaBackup(arquivo)) {
        const backup = criarBackup(arquivo, pastaBackup);

        if (backup) {
          backupsRealizados++;
          Logger.log(`Backup criado: ${arquivo.getName()}`);
        }
      }
    });

    limparBackupsAntigos(pastaBackup);
    enviarRelatorioBackup(backupsRealizados);
  } catch (error) {
    Logger.log("Erro no backup:", error.toString());
    enviarNotificacaoErro(error);
  }
}

/**
 * Identifica arquivos que precisam de backup
 */
function identificarArquivosImportantes() {
  const arquivos = [];

  // Planilhas com dados financeiros
  const pastaFinanceiro = DriveApp.getFolderById(CONFIG.PASTA_FINANCEIRO_ID);
  const planilhasFinanceiras = pastaFinanceiro.getFilesByType(
    MimeType.GOOGLE_SHEETS
  );

  while (planilhasFinanceiras.hasNext()) {
    arquivos.push(planilhasFinanceiras.next());
  }

  // Documentos de contratos
  const pastaContratos = DriveApp.getFolderById(CONFIG.PASTA_CONTRATOS_ID);
  const documentosContratos = pastaContratos.getFiles();

  while (documentosContratos.hasNext()) {
    const arquivo = documentosContratos.next();
    if (arquivo.getName().toLowerCase().includes("contrato")) {
      arquivos.push(arquivo);
    }
  }

  return arquivos;
}

/**
 * Verifica se arquivo precisa de novo backup
 */
function precisaBackup(arquivo) {
  const ultimaModificacao = arquivo.getLastUpdated();
  const agora = new Date();

  // Backup se foi modificado nos últimos 7 dias
  const diasDiferenca = (agora - ultimaModificacao) / (1000 * 60 * 60 * 24);

  return diasDiferenca <= 7;
}
```

## 📋 Exemplo: Monitoramento de Permissões

```javascript
/**
 * Auditoria de permissões de arquivos sensíveis
 * Execução: Trigger diário às 09:00
 */
function auditarPermissoes() {
  try {
    Logger.log("Iniciando auditoria de permissões...");

    const pastasSensiveis = obterPastasSensiveis();
    const relatorio = [];

    pastasSensiveis.forEach((pasta) => {
      const arquivos = pasta.getFiles();

      while (arquivos.hasNext()) {
        const arquivo = arquivos.next();
        const permissoes = analisarPermissoes(arquivo);

        if (permissoes.temRisco) {
          relatorio.push({
            nome: arquivo.getName(),
            pasta: pasta.getName(),
            riscos: permissoes.riscos,
            url: arquivo.getUrl(),
          });
        }
      }
    });

    if (relatorio.length > 0) {
      enviarAlertaSeguranca(relatorio);
    }

    Logger.log(`Auditoria concluída. ${relatorio.length} arquivos com risco.`);
  } catch (error) {
    Logger.log("Erro na auditoria:", error.toString());
    enviarNotificacaoErro(error);
  }
}

/**
 * Analisa permissões de um arquivo
 */
function analisarPermissoes(arquivo) {
  const resultado = {
    temRisco: false,
    riscos: [],
  };

  try {
    const editores = arquivo.getEditors();
    const visualizadores = arquivo.getViewers();

    // Verifica compartilhamento público
    const sharingAccess = arquivo.getSharingAccess();
    if (
      sharingAccess === DriveApp.Access.ANYONE ||
      sharingAccess === DriveApp.Access.ANYONE_WITH_LINK
    ) {
      resultado.temRisco = true;
      resultado.riscos.push("Compartilhamento público ativo");
    }

    // Verifica muitos editores
    if (editores.length > CONFIG.MAX_EDITORES) {
      resultado.temRisco = true;
      resultado.riscos.push(`Muitos editores: ${editores.length}`);
    }

    // Verifica editores externos
    editores.forEach((editor) => {
      if (!editor.getEmail().endsWith(CONFIG.DOMINIO_EMPRESA)) {
        resultado.temRisco = true;
        resultado.riscos.push(`Editor externo: ${editor.getEmail()}`);
      }
    });
  } catch (error) {
    Logger.log("Erro ao analisar permissões:", error.toString());
  }

  return resultado;
}
```

## ⚙️ Configuração

### 1. Script Properties Necessárias

```javascript
function configurarProperties() {
  const properties = PropertiesService.getScriptProperties();

  properties.setProperties({
    // Pastas principais
    PASTA_RAIZ_ID: "ID_DA_PASTA_RAIZ",
    PASTA_BACKUP_ID: "ID_DA_PASTA_BACKUP",

    // Pastas por tipo
    PASTA_PDF_ID: "ID_DA_PASTA_PDF",
    PASTA_IMAGENS_ID: "ID_DA_PASTA_IMAGENS",
    PASTA_PLANILHAS_ID: "ID_DA_PASTA_PLANILHAS",
    PASTA_DOCUMENTOS_ID: "ID_DA_PASTA_DOCUMENTOS",

    // Configurações de segurança
    DOMINIO_EMPRESA: "suaempresa.com",
    MAX_EDITORES: "5",
    EMAIL_ADMIN: "admin@suaempresa.com",
  });
}
```

### 2. Triggers Recomendados

- **Organização**: Diário às 06:00
- **Backup**: Semanal aos domingos
- **Auditoria**: Diário às 09:00
- **Limpeza**: Mensal no dia 1

### 3. Permissões Necessárias

- Drive API (leitura/escrita)
- Gmail API (notificações)
- Script triggers

## 🔒 Considerações de Segurança

- ✅ Use IDs de pastas em Properties
- ✅ Valide permissões antes de mover arquivos
- ✅ Mantenha logs detalhados
- ✅ Configure notificações de erro
- ❌ Nunca hardcode IDs de pastas sensíveis
- ❌ Não processe arquivos sem validação

## 📈 Monitoramento

### Métricas Importantes

- Número de arquivos organizados
- Backups realizados com sucesso
- Arquivos com riscos de segurança
- Tempo de execução das automações

### Alertas Configurados

- Falhas em backups críticos
- Compartilhamentos públicos detectados
- Editores externos em arquivos sensíveis
- Excesso de arquivos em pastas específicas

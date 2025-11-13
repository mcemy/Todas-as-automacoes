# 📝 Automações Google Forms

Automações em **JavaScript** usando **Google Apps Script** para processamento avançado de formulários e respostas.

## 🚀 Funcionalidades

### 📊 Processamento de Respostas

- Validação automática de dados
- Análise de sentimentos
- Classificação de respostas
- Geração de relatórios

### 🎓 Geração de Certificados

- Certificados personalizados
- Envio automático por email
- Controle de numeração
- Templates customizáveis

### 🔗 Integração com Planilhas

- Sincronização em tempo real
- Formatação automática
- Cálculos dinâmicos
- Dashboards automáticos

### 📧 Notificações Inteligentes

- Alertas para administradores
- Confirmações para respondentes
- Relatórios periódicos
- Notificações condicionais

## 🛠️ APIs Utilizadas

- **FormApp**: Manipulação de formulários
- **SpreadsheetApp**: Integração com planilhas
- **MailApp**: Envio de emails
- **DriveApp**: Gestão de arquivos
- **Utilities**: Formatação e conversões

## 📋 Exemplo: Processamento de Inscrições

```javascript
/**
 * Processa inscrições de eventos automaticamente
 * Trigger: Envio de formulário
 */
function processarInscricao(e) {
  try {
    Logger.log("Processando nova inscrição...");

    const respostas = e.namedValues;
    const inscricao = {
      nome: respostas["Nome completo"][0],
      email: respostas["Email"][0],
      telefone: respostas["Telefone"][0],
      evento: respostas["Evento"][0],
      timestamp: new Date(),
    };

    // Validação dos dados
    if (!validarInscricao(inscricao)) {
      enviarEmailErro(inscricao);
      return;
    }

    // Processar inscrição
    const numeroInscricao = gerarNumeroInscricao();
    inscricao.numeroInscricao = numeroInscricao;

    // Salvar na planilha de controle
    salvarInscricao(inscricao);

    // Gerar certificado se aplicável
    if (eventoComCertificado(inscricao.evento)) {
      gerarCertificado(inscricao);
    }

    // Enviar email de confirmação
    enviarConfirmacao(inscricao);

    // Notificar administradores se necessário
    if (inscricao.evento === "VIP" || inscricao.evento === "Executivo") {
      notificarAdministradores(inscricao);
    }

    Logger.log(`Inscrição processada: ${inscricao.nome} - ${numeroInscricao}`);
  } catch (error) {
    Logger.log("Erro no processamento:", error.toString());
    enviarAlertaErro(error, e);
  }
}

/**
 * Valida dados da inscrição
 */
function validarInscricao(inscricao) {
  // Validar email
  const regexEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!regexEmail.test(inscricao.email)) {
    Logger.log("Email inválido:", inscricao.email);
    return false;
  }

  // Validar telefone brasileiro
  const regexTelefone = /^\(\d{2}\)\s\d{4,5}-\d{4}$/;
  if (!regexTelefone.test(inscricao.telefone)) {
    Logger.log("Telefone inválido:", inscricao.telefone);
    return false;
  }

  // Validar nome (mínimo 2 palavras)
  if (inscricao.nome.split(" ").length < 2) {
    Logger.log("Nome incompleto:", inscricao.nome);
    return false;
  }

  return true;
}

/**
 * Gera número sequencial de inscrição
 */
function gerarNumeroInscricao() {
  const planilha = SpreadsheetApp.openById(CONFIG.PLANILHA_CONTROLE_ID);
  const aba = planilha.getSheetByName("Controle");

  let ultimoNumero =
    PropertiesService.getScriptProperties().getProperty("ULTIMO_NUMERO") || "0";

  ultimoNumero = parseInt(ultimoNumero) + 1;

  PropertiesService.getScriptProperties().setProperty(
    "ULTIMO_NUMERO",
    ultimoNumero.toString()
  );

  return "INS" + ultimoNumero.toString().padStart(6, "0");
}
```

## 📋 Exemplo: Geração de Certificados

```javascript
/**
 * Gera certificado personalizado para participante
 */
function gerarCertificado(inscricao) {
  try {
    Logger.log(`Gerando certificado para: ${inscricao.nome}`);

    // Obter template do certificado
    const templateId = obterTemplateCertificado(inscricao.evento);
    const template = DriveApp.getFileById(templateId);

    // Criar cópia do template
    const certificado = template.makeCopy(
      `Certificado_${inscricao.numeroInscricao}_${inscricao.nome}`,
      DriveApp.getFolderById(CONFIG.PASTA_CERTIFICADOS_ID)
    );

    // Abrir documento para edição
    const doc = DocumentApp.openById(certificado.getId());
    const body = doc.getBody();

    // Substituir placeholders
    const substituicoes = {
      "{{NOME}}": inscricao.nome.toUpperCase(),
      "{{EVENTO}}": inscricao.evento,
      "{{DATA}}": Utilities.formatDate(new Date(), "GMT-3", "dd/MM/yyyy"),
      "{{NUMERO}}": inscricao.numeroInscricao,
      "{{CARGA_HORARIA}}": obterCargaHoraria(inscricao.evento),
    };

    Object.keys(substituicoes).forEach((placeholder) => {
      body.replaceText(placeholder, substituicoes[placeholder]);
    });

    // Salvar alterações
    doc.saveAndClose();

    // Converter para PDF
    const pdfBlob = DriveApp.getFileById(certificado.getId()).getAs(
      MimeType.PDF
    );

    const certificadoPdf = DriveApp.createFile(pdfBlob).setName(
      `Certificado_${inscricao.numeroInscricao}.pdf`
    );

    // Mover para pasta correta
    DriveApp.getFolderById(CONFIG.PASTA_CERTIFICADOS_PDF_ID).addFile(
      certificadoPdf
    );

    // Remover da raiz
    DriveApp.removeFile(certificadoPdf);

    // Anexar à inscrição
    inscricao.certificadoId = certificadoPdf.getId();

    // Remover documento temporário
    DriveApp.removeFile(certificado);

    Logger.log(`Certificado gerado: ${certificadoPdf.getName()}`);

    return certificadoPdf;
  } catch (error) {
    Logger.log("Erro na geração do certificado:", error.toString());
    throw error;
  }
}

/**
 * Obtém template baseado no tipo de evento
 */
function obterTemplateCertificado(evento) {
  const templates = {
    Workshop: CONFIG.TEMPLATE_WORKSHOP_ID,
    Palestra: CONFIG.TEMPLATE_PALESTRA_ID,
    Curso: CONFIG.TEMPLATE_CURSO_ID,
    VIP: CONFIG.TEMPLATE_VIP_ID,
  };

  return templates[evento] || CONFIG.TEMPLATE_PADRAO_ID;
}

/**
 * Envia certificado por email
 */
function enviarCertificadoPorEmail(inscricao, certificado) {
  const assunto = `Certificado - ${inscricao.evento} - ${inscricao.nome}`;

  const corpo = `
    Olá ${inscricao.nome}!
    
    Parabéns pela participação no evento "${inscricao.evento}"!
    
    Seu certificado está em anexo.
    
    Número do certificado: ${inscricao.numeroInscricao}
    Data de emissão: ${Utilities.formatDate(new Date(), "GMT-3", "dd/MM/yyyy")}
    
    Atenciosamente,
    Equipe de Eventos
  `;

  MailApp.sendEmail({
    to: inscricao.email,
    subject: assunto,
    body: corpo,
    attachments: [certificado.getBlob()],
  });

  Logger.log(`Certificado enviado para: ${inscricao.email}`);
}
```

## 📋 Exemplo: Análise de Feedback

```javascript
/**
 * Analisa feedback de eventos e gera relatórios
 * Execução: Trigger semanal
 */
function analisarFeedback() {
  try {
    Logger.log("Iniciando análise de feedback...");

    const planilhaFeedback = SpreadsheetApp.openById(
      CONFIG.PLANILHA_FEEDBACK_ID
    );
    const dados = planilhaFeedback.getDataRange().getValues();

    const analise = {
      totalRespostas: dados.length - 1, // Exclui cabeçalho
      mediaGeral: 0,
      distribucaoNotas: {},
      comentariosPositivos: [],
      comentariosNegativos: [],
      sugestoes: [],
    };

    // Processar cada resposta
    for (let i = 1; i < dados.length; i++) {
      const linha = dados[i];
      const nota = linha[2]; // Coluna da nota
      const comentario = linha[3]; // Coluna do comentário
      const sugestao = linha[4]; // Coluna da sugestão

      // Calcular estatísticas
      analise.mediaGeral += nota;

      if (!analise.distribucaoNotas[nota]) {
        analise.distribucaoNotas[nota] = 0;
      }
      analise.distribucaoNotas[nota]++;

      // Classificar comentários
      if (comentario && comentario.length > 0) {
        if (nota >= 4) {
          analise.comentariosPositivos.push({
            nota: nota,
            comentario: comentario,
            data: linha[0],
          });
        } else if (nota <= 2) {
          analise.comentariosNegativos.push({
            nota: nota,
            comentario: comentario,
            data: linha[0],
          });
        }
      }

      // Coletar sugestões
      if (sugestao && sugestao.length > 0) {
        analise.sugestoes.push({
          sugestao: sugestao,
          nota: nota,
          data: linha[0],
        });
      }
    }

    // Calcular média
    analise.mediaGeral = analise.mediaGeral / analise.totalRespostas;

    // Gerar relatório
    gerarRelatorioFeedback(analise);

    // Enviar alertas se necessário
    if (analise.mediaGeral < 3) {
      enviarAlertaMediaBaixa(analise);
    }

    Logger.log("Análise de feedback concluída");
  } catch (error) {
    Logger.log("Erro na análise:", error.toString());
  }
}

/**
 * Gera relatório detalhado de feedback
 */
function gerarRelatorioFeedback(analise) {
  const planilhaRelatorio = SpreadsheetApp.openById(
    CONFIG.PLANILHA_RELATORIO_ID
  );
  const aba =
    planilhaRelatorio.getSheetByName("Feedback Semanal") ||
    planilhaRelatorio.insertSheet("Feedback Semanal");

  // Limpar conteúdo anterior
  aba.clear();

  // Cabeçalho
  aba.getRange(1, 1, 1, 2).setValues([["Relatório de Feedback", ""]]);
  aba.getRange(1, 1, 1, 2).setFontWeight("bold").setFontSize(14);

  let linha = 3;

  // Estatísticas gerais
  aba.getRange(linha, 1).setValue("Total de Respostas:");
  aba.getRange(linha, 2).setValue(analise.totalRespostas);
  linha++;

  aba.getRange(linha, 1).setValue("Média Geral:");
  aba.getRange(linha, 2).setValue(analise.mediaGeral.toFixed(2));
  linha += 2;

  // Distribuição de notas
  aba.getRange(linha, 1).setValue("Distribuição de Notas:");
  aba.getRange(linha, 1).setFontWeight("bold");
  linha++;

  Object.keys(analise.distribucaoNotas).forEach((nota) => {
    aba.getRange(linha, 1).setValue(`Nota ${nota}:`);
    aba.getRange(linha, 2).setValue(analise.distribucaoNotas[nota]);
    linha++;
  });

  linha++;

  // Comentários positivos (top 5)
  if (analise.comentariosPositivos.length > 0) {
    aba.getRange(linha, 1).setValue("Top 5 Comentários Positivos:");
    aba.getRange(linha, 1).setFontWeight("bold");
    linha++;

    analise.comentariosPositivos
      .sort((a, b) => b.nota - a.nota)
      .slice(0, 5)
      .forEach((item) => {
        aba.getRange(linha, 1).setValue(`(${item.nota}) ${item.comentario}`);
        linha++;
      });
  }

  // Salvar data do relatório
  aba.getRange(1, 3).setValue("Gerado em:");
  aba.getRange(1, 4).setValue(new Date());
}
```

## ⚙️ Configuração

### 1. Script Properties Necessárias

```javascript
function configurarProperties() {
  const properties = PropertiesService.getScriptProperties();

  properties.setProperties({
    // Planilhas
    PLANILHA_CONTROLE_ID: "ID_DA_PLANILHA_CONTROLE",
    PLANILHA_FEEDBACK_ID: "ID_DA_PLANILHA_FEEDBACK",
    PLANILHA_RELATORIO_ID: "ID_DA_PLANILHA_RELATORIO",

    // Templates de certificados
    TEMPLATE_PADRAO_ID: "ID_DO_TEMPLATE_PADRAO",
    TEMPLATE_WORKSHOP_ID: "ID_DO_TEMPLATE_WORKSHOP",
    TEMPLATE_CURSO_ID: "ID_DO_TEMPLATE_CURSO",

    // Pastas
    PASTA_CERTIFICADOS_ID: "ID_DA_PASTA_CERTIFICADOS",
    PASTA_CERTIFICADOS_PDF_ID: "ID_DA_PASTA_PDF",

    // Configurações
    EMAIL_ADMIN: "admin@suaempresa.com",
    ULTIMO_NUMERO: "0",
  });
}
```

### 2. Triggers Recomendados

- **Envio de formulário**: Processamento imediato
- **Análise semanal**: Domingos às 08:00
- **Limpeza mensal**: Dia 1 de cada mês
- **Backup diário**: Todos os dias às 23:00

### 3. Permissões Necessárias

- Forms API (leitura)
- Sheets API (leitura/escrita)
- Drive API (criação de arquivos)
- Gmail API (envio de emails)
- Documents API (edição de templates)

## 🔒 Considerações de Segurança

- ✅ Valide todos os dados de entrada
- ✅ Use Properties para IDs de arquivos
- ✅ Configure limitações de taxa
- ✅ Mantenha logs de auditoria
- ❌ Nunca processe dados sem validação
- ❌ Não armazene dados sensíveis em planilhas públicas

## 📈 Monitoramento

### Métricas Importantes

- Taxa de submissões por dia
- Tempo médio de processamento
- Taxa de erro em validações
- Satisfação média dos eventos

### Alertas Configurados

- Falhas no processamento de inscrições
- Média de feedback abaixo de 3.0
- Erros na geração de certificados
- Volume anormal de submissões

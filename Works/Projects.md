Qual é o problema

Essa demanda veio a partir de um cliente proprietário de uma imobiliária. 

O cliente é usuário do RD Station Marketing e CRM.

O cliente precisava de uma ferramenta de BI para visualização das métricas de marketing que estão sendo utilizadas na sua empresa.

Qual foi a solução

A partir do conhecimento da dor do cliente, foram levantadas as métricas que o mesmo gostaria que fossem adicionadas à ferramenta BI.

 Baseados nas métricas solicitadas, foram levantados os dados necessários para construí-las.

Os dados foram extraídos do RD Station a partir de uma API de integração utilizando o Google Apps Script. 

Os dados são extraídos do RD Station e enviados para o Google Sheets via API. Os dados armazenados no Google Sheets alimenta um Dashboard construído no Google Data Studio.

A arquitetura do projeto pode ser visualizada abaixo:

FIGURA

O RD Station envia os dados em forma de Webhook e também podem ser enviados através de uma url de integração.

Dessa forma, para extrair os dados da plataforma, foram criados 3 webhook, sendo eles:

 
Leads: Nesse webhook foi adicionado as landing pages responsáveis pela conversão de novos leads;
Oportunidades: Nesse webhook foi adicionados as landing pages responsáveis pelas conversões de novas oportunidades;
Vendas: Cada Oportunidade marcada como Venda no RD Station CRM, gera um evento de conversão no RD Station Marketing.



E um fluxo de automação para captar oportunidades perdidas. 
As oportunidades marcadas como perdidas no RD Station CRM, geram um evento de conversão no RD Station Marketing, dessa forma essas Oportunidades Perdidas entram no fluxo de automação criado e são enviados via url de integração para o Google Sheets.

O RD Station envia os dados em um JSON padrão, como este por exemplo:

{
 "leads": [
 {
 "id": "390319847",
 "email": "teste@webhook.com",
 "name": "Fulano Suporte RD",
 "company": null,
 "job_title": "Analista",
 "bio": null,
 "public_url": "http:\/\/rdstation.com.br\/leads\/public\/807029c7-267f-4225-8428-87ae2dab34c3",
 "created_at": "2018-09-26T17:57:10.189-03:00",
 "opportunity": "true",
 "number_conversions": "1",
 ...
 }] 
}

A API de integração é responsável por solicitar os dados para o RD, e separar os campos(processo de parse). O RD envia todos os dados disponíveis, mas é possível escolher quais desejamos salvar na nossa planilha. 
O RD Station possui campos padrões e campos personalizados, que diferenciam de acordo com o negócio.
A API para o webhook Leads é apresentada abaixo

// Função que recebe os dados do webhook do RD Station e insere dados na planilha
function doPost(e) {
  // Acessa planilha e aba para inserir dados
  var spreadsheet = SpreadsheetApp.openById('ID');
  // O identificar da planilha está em sua URL
  // Para mais informações acesse: https://developers.google.com/apps-script/reference/spreadsheet/spreadsheet-app#openbyidid
  var sheet = spreadsheet.getSheetByName('Leads');
  
  // Acessa os dados enviados pelo webhook do RD Station   
  var requestData = JSON.parse(e.postData.contents);
  var leadData = requestData.leads;
  
  // Cria uma trava que impede que dois ou mais usuários executem o script simultaneamente
  var trava = LockService.getScriptLock();
  trava.waitLock(2000);
  
  //
  var values = []
  var timestamp = new Date();
  var JSONSource = JSON.stringify(requestData);
  
  //Extrai dados do lead para inserção
  for (var i = 0; i < leadData.length; i++) {
    values.push([JSONSource,
                 timestamp,
                 leadData[i].id,
                 leadData[i].first_conversion.content.identificador,
                 leadData[i].first_conversion.conversion_origin.source,
                 leadData[i].first_conversion.conversion_origin.medium,
                 leadData[i].custom_fields["Como podemos te ajudar?"],
                 leadData[i].custom_fields["Prefiro contato via:"],
                 leadData[i].custom_fields["Qual seu plano do RD Station?"],
                 leadData[i].custom_fields["Utiliza RD Station CRM?"]]);
  }
  
  // Atualiza a planilha com a nova linha  
  sheet.getRange(sheet.getLastRow()+1, 1, values.length, values[0].length).setValues(values);
  SpreadsheetApp.flush();
  
    
  // Desativa a trava do script para que possa receber outras mensagens do webhook
  trava.releaseLock();
  return "OK";
}

function doGet(request) {
  return HtmlService.createHtmlOutput("<h2>Get request recebida.</h2><p>Essa função te ajuda a identificar se o Web App da integração está ativo.</p>");
}

Após fazer o deploy da API, e adicionar a url ao respectivo webhook, a aplicação estava em funcionamento, populando a planilha

FIGURA


Com a primeira parte do projeto finalizado, iniciou a etapa da construção de um dashboard utilizando o Google DataStudio.

O Data Studio é uma ferramenta open source, possui uma diversidade de ferramentas. Nele é possível utilizar diversas fontes de dados dentre elas:


FIGURA


Pode-se combinar fontes de dados 


FIGURS


Criar campos calculados

FIGURA


Filtros 


E um dashboard com um visual sensacional com informações relevantes e que geram valor.  

O resultado final do Dashboard, você confere agora.

O projeto consiste em três telas:

A primeira delas é para se ter uma visão geral da base e do funil de vendas
  
  
  
  A segunda delas é para avaliar as campanhas de marketing


A terceira é para entender a segmentação e interesse dos leads


Esse é um dashboard operacional, apresenta métricas presentes e é ideal para avaliar em tempo real o comportamento da base de leads, vendas, campanhas, performance dos times comercial e marketing.

A partir dele, é possível visualizar com bastante clareza os dados apresentados podendo ser utilizado para obter insights sobre a estratégia atual da empresa.




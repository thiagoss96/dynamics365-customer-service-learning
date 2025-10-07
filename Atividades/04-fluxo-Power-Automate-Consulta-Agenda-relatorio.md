# 04 - Fluxo: Consulta de Reuniões e Exportação para Excel (Power Automate)

## Visão Geral

Fluxo agendado criado no **Power Automate** que é executado **diariamente às 18:00 (BRT)**.  
O fluxo consulta o **calendário do Outlook** de um usuário, obtém todas as **reuniões do dia** (janela entre 06:00 e 22:00) e registra as informações em uma **planilha Excel** no OneDrive for Business. Cada participante é inserido em **uma linha separada** — se uma reunião tiver 6 participantes, o fluxo criará 6 linhas com os mesmos valores de **Assunto, Início e Término**, variando apenas o campo **Participante**.

Esse documento descreve a solução, o gatilho, variáveis, ações, expressões e considerações práticas para inclusão no seu repositório do GitHub.

---

## Título do arquivo sugerido
`04-fluxo-aprovacao-power-automate.md`

---

## Cenário e Objetivo

**Cenário:** era necessário gerar um relatório diário com todas as reuniões realizadas por um usuário do Outlook, detalhando cada participante em linhas separadas para facilitar controle e análises.  
**Objetivo:** automatizar a captura das reuniões do dia e gravar os dados em uma planilha Excel, com opção de enviar a planilha por e‑mail ao final do processo.

---

## Gatilho (Trigger)

- **Tipo:** Recorrência (`Recurrence`)
- **Configuração:**
  - Intervalo: `1`
  - Unidade: `Day`
  - Fuso horário: `UTC -03:00` (E. South America Standard Time / Brasília)
  - Horário de disparo: `18:00`

Exemplo de configuração no Power Automate: use o gatilho **Recurrence** com as propriedades acima.

---

## Variáveis iniciais

Crie as variáveis necessárias logo após o gatilho.

1. **Variável:** `InicioJanela`  
   - **Tipo:** String  
   - **Valor (expressão):**
```text
formatDateTime(addHours(startOfDay(utcNow()), 6), 'yyyy-MM-ddTHH:mm:ssZ')
```  
   - **Descrição:** define o início da janela (06:00 do dia atual em UTC Z format usado pelo conector).

2. **Variável:** `FimJanela`  
   - **Tipo:** String  
   - **Valor (expressão):**
```text
formatDateTime(addHours(startOfDay(utcNow()), 22), 'yyyy-MM-ddTHH:mm:ssZ')
```  
   - **Descrição:** define o fim da janela (22:00 do dia atual).

3. **Variável:** `ListaParticipantes`  
   - **Tipo:** String  
   - **Valor inicial:** string vazia (`""`)  
   - **Descrição:** será usada para montar a lista de participantes de cada reunião (concatenando required e optional attendees).

---

## Ações principais

### 1) Obter eventos do calendário (Outlook V3)

- **Ação:** **Get calendar view of events (V3)** / *Obter o modo de exibição Calendário de eventos (V3)*  
- **Conector:** Office 365 Outlook  
- **Parâmetros:**
  - **Calendar id:** `Calendar` (ou o id do calendário desejado)
  - **Start time:** `@{variables('InicioJanela')}`
  - **End time:** `@{variables('FimJanela')}`

Esta ação retorna uma lista de eventos no período definido. A saída principal que vamos iterar é:  
```text
@outputs('Obter_o_modo_de_exibição_Calendário_de_eventos_(V3)')?['body/value']
```

### 2) Loop: Aplicar a cada (Apply to each)

- **Ação:** `Apply to each` (Aplicar a cada)  
- **Entrada:** saída do passo anterior (lista de eventos).

Dentro do loop, executar as seguintes sub‑ações para cada evento:

#### 2.1) Montar/concatenar lista de participantes
- **Ação:** `Append to string variable` (Acrescentar à variável)
- **Variável:** `ListaParticipantes`
- **Valor (expressão):**
```text
concat(
  replace(coalesce(items('Apply_to_each')?['requiredAttendees'], ''), ';', ', '),
  if(
    and(
      not(empty(items('Apply_to_each')?['optionalAttendees'])),
      not(empty(items('Apply_to_each')?['requiredAttendees']))
    ),
    ', ',
    ''
  ),
  replace(coalesce(items('Apply_to_each')?['optionalAttendees'], ''), ';', ', ')
)
```
> Observação: no designer em PT/BR os nomes dos passos podem ficar com espaços/acentos — ajuste `items('Apply_to_each')` para o nome exato do passo conforme o seu fluxo (ex: `items('Aplicar_a_cada')`).

Essa expressão junta os participantes obrigatórios (`requiredAttendees`) e opcionais (`optionalAttendees`), substitui `;` por `, ` e garante a separação correta quando ambos existem.

#### 2.2) Adicionar linha na tabela do Excel (Excel Online - Business)
- **Ação:** `Add a row into a table` (Adicionar uma linha em uma tabela)  
- **Conector:** Excel Online (Business)  
- **Configuração:**
  - **Location:** OneDrive for Business  
  - **Document Library:** Documentos  
  - **File:** `/pasta/seu_arquivo.xlsx` (caminho no OneDrive)
  - **Table:** `Tabela1` (nome da tabela dentro do arquivo; a planilha precisa estar formatada como Tabela)

**Mapeamento de colunas (exemplos):**
- `Assunto` : `@{items('Apply_to_each')?['subject']}`
- `Inicio`  :  
```text
formatDateTime(convertTimeZone(items('Apply_to_each')?['start'],'UTC','E. South America Standard Time'),'yyyy/MM/dd HH:mm:ss')
```
- `Termino` :  
```text
formatDateTime(convertTimeZone(items('Apply_to_each')?['end'],'UTC','E. South America Standard Time'),'yyyy/MM/dd HH:mm:ss')
```
- `Participante` : `@{variables('ListaParticipantes')}`

> Importante: se você quer inserir **uma linha por participante** (cada participante em uma linha separada), você precisa ajustar a lógica: em vez de concatenar todos os participantes em uma única string e inserir uma linha, percorra a lista de participantes individualmente e insira uma linha para cada participante. O documento original concatenava e inseria a string completa; se o comportamento desejado é uma linha por participante, será necessário dividir a string e iterar sobre ela.

---

## Etapa Opcional: enviar a planilha por e‑mail

Se desejar enviar a planilha ao final do processo, adicione os passos abaixo **após** o loop principal:

### 3) Obter conteúdo do arquivo (OneDrive for Business)
- **Ação:** `Get file content` / `Obter conteúdo de arquivo`  
- **Conector:** OneDrive for Business  
- **File:** `/pasta/seu_arquivo.xlsx`  
- **Infer content type:** `Yes` (Inferir tipo de conteúdo: Sim)

### 4) Enviar um e‑mail (Office 365 Outlook - V2)
- **Ação:** `Send an email (V2)` / `Enviar um e‑mail (V2)`  
- **Conector:** Office 365 Outlook  
- **Configurações principais:**
  - **To:** `seu.email@dominio.com` (ou endereço do destinatário)
  - **Subject:** `Relatório de Reuniões`
  - **Body:** `Segue em anexo o relatório de reuniões do dia.`
- **Anexo (parâmetro avançado):** use o conteúdo obtido no passo anterior:
```json
[
  {
    "Name": "relatorio_reunioes.xlsx",
    "ContentBytes": @{body('Get_file_content')}
  }
]
```

---

## Resultado esperado

- Execução diária às 18:00 (BRT).  
- Coleta de todas as reuniões entre 06:00 e 22:00 do dia.  
- Inserção dos dados no Excel (cada reunião e lista de participantes conforme mapeamento).  
- (Opcional) Envio automático da planilha por e‑mail.

---

## Observações, melhorias e boas práticas

- **Tabela no Excel:** verifique que a planilha esteja formatada como **Tabela** (Insert → Table). O passo “Adicionar linha em tabela” exige que exista uma tabela nomeada.  
- **Duplicidade:** se o fluxo for executado mais de uma vez por dia, implemente checagem para evitar inserir linhas duplicadas (ex.: comparar assunto + início + participante).  
- **Fuso horário:** usar `convertTimeZone` para normalizar horários; o exemplo usa `E. South America Standard Time`. Ajuste se necessário.  
- **Tratamento de erros:** adicione blocos de tratamento (Configure run after / Try/Catch pattern) para capturar falhas no acesso ao OneDrive ou Outlook.  
- **Auditoria:** considere gravar em outra tabela/arquivo um log de execução (data/hora e número de registros inseridos).  
- **Performance:** para grande volume de eventos/participantes, avalie limites do conector Excel (throttling) e considere usar armazenamento intermediário (SharePoint list ou Azure Table) se necessário.

---

## Links úteis (para inclusão no repositório)
- Imagem do fluxo (exemplo): `../imagens/fluxo-power-automate.png`  
- Imagem da planilha gerada (exemplo): `../imagens/planilha-reunioes.png`


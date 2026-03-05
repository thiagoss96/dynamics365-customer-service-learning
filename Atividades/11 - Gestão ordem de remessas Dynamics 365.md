# 📖 Storytelling: Gestão de Abastecimento e Fluxo de Remessas

### 1. O Cenário: O Desafio do Abastecimento Contínuo
Imagine uma grande rede varejista, como a **Toys Mania**, que possui 100 filiais espalhadas pelo Brasil. Eles fecham uma negociação massiva de **200.000 unidades** de um produto para garantir estoque e condições comerciais. 

Embora a venda seja fechada em um único contrato (Cotação), a operação logística é o grande desafio. Se a empresa fornecedora enviar todas as unidades de uma vez:
* **Gargalo no CD:** O cliente estoura a capacidade física de armazenamento.
* **Capital Imobilizado:** O fluxo de caixa do cliente fica preso em produtos que ainda levarão meses para serem vendidos.
* **Risco Operacional:** A logística sofre um pico de trabalho insustentável em um único período.

### 2. A Solução: Fracionamento Inteligente com Ordens de Remessa
Para resolver esse problema, implementamos a lógica de **Ordens de Remessa**. Em vez de um envio massivo, o executivo comercial e o cliente definem um cronograma de meses à frente diretamente no sistema.

* **Contrato Único, Execução Progressiva:** A Cotação garante as condições comerciais, mas a execução acontece em lotes planejados.
* **Identidade e Rastreabilidade:** Cada lote é uma entidade independente com ID próprio (ex: **REM-0001**, **REM-0002**, **REM-0003**). Isso permite que tanto o cliente quanto o vendedor saibam exatamente o que já foi enviado e o que ainda está provisionado para os meses seguintes.

### 3. A Dor Eliminada: Adeus ao Monitoramento Manual
Antes desta solução, o executivo comercial dependia de planilhas paralelas e lembretes manuais para solicitar cada envio ao estoque. O risco de esquecer uma remessa e deixar as prateleiras do cliente vazias era constante.

Com a automação via **Power Automate** e **Dataverse**:
* **Monitoramento Proativo:** O sistema atua como um "fiscal" de prazos. Ao identificar que a **REM-0002** está próxima da data de entrega, ele inicia o processo de separação automaticamente.
* **Inteligência de Dados:** O sistema cruza as informações e abre uma **Ocorrência (Ticket)** para a logística, já contendo o ID da Remessa, dados do Produto e detalhes da Cotação. O trabalho de "detetive" para buscar informações em diferentes tabelas foi eliminado.

### 4. O Resultado: Parceiro Estratégico, não apenas Vendedor
O uso das Ordens de Remessa transforma a relação comercial. A empresa deixa de ser apenas um fornecedor e passa a ser um parceiro logístico estratégico.
* **Para o Comercial:** Liberdade para focar em novas vendas, com a segurança de que o cronograma das vendas fechadas está sendo gerido pelo sistema.
* **Para o Cliente:** Garantia de abastecimento mensal, conforme sua capacidade de venda e recepção, com rastreabilidade total de cada item.







#  Sistema de Gestão de Remessas e Ocorrências (Dataverse + Power Automate)

##  Visão Geral do Projeto
Este repositório documenta a implementação técnica de uma solução para gerenciamento do ciclo de vida de **Ordens de Remessa**. A arquitetura foca na automação proativa de SLAs, utilizando **Dataverse** como camada de persistência relacional e **Power Automate** para a orquestração de processos (abertura e fechamento de chamados).

---

##  1. Arquitetura de Dados (Dataverse Schema)

A modelagem foi projetada para garantir integridade referencial e alta performance em consultas escaláveis através de identificadores globais.

### A. Entidade Principal: Ordem de Remessa (`dyn_ordemderemessa`)

| Nome Lógico | Nome de Exibição | Tipo de Dado | Descrição Técnica |
| :--- | :--- | :--- | :--- |
| `dyn_ordemremessaid` | ID Remessa | Unique Identifier | **Primary Key (GUID)** do registro. |
| `dyn_name` | ID da Remessa | String | Identificador legível (ex: REM-001). |
| `dyn_statusdalinha` | Status da Linha | Choice (OptionSet) | `1`: Pendente, `2`: Em Processo, `3`: Concluído. |
| `dyn_prazolimite` | Prazo Limite | Date and Time | Parâmetro de validação para o motor de SLA. |
| `_dyn_quote_value` | Cotação | Lookup (N:1) | Foreign Key (FK) para a tabela nativa `quote`. |
| `_dyn_plano_value` | Plano | Lookup (N:1) | Relacionamento (FK) com a tabela nativa de Produtos (`product`). |
| `dyn_quantidade` | Quantidade | Whole Number | Campo numérico inteiro para controle de volume da remessa. |

### B. Entidade Relacionada: Ocorrência (`incident`)
Foi realizada uma extensão na tabela nativa de Ocorrências para permitir o rastreio das remessas:
* **Campo Lookup Customizado:** `dyn_ordemderemessa` (Entity Reference).
* **Chave de Ligação:** Armazena o **GUID** da Ordem de Remessa pai, possibilitando filtros **OData** eficientes em nível de servidor.



---

##  2. Interface e Lógica Formulário de Cotação (Dynamics 365)

### Controle Dinâmico de Componentes (Business Rules)
A aba "Termos" no formulário de **Cotação (Quote)** foi customizada para suportar múltiplos tipos de documentos. Para evitar poluição visual e inconsistência de dados, foi implementada uma lógica de exclusividade via Regra de Negócio.

* **Componentes Inseridos:** Subgrids apontando para a entidade relacionada `Ordem de Remessa`.
* **Escopo:** Formulário de Cotação (Aba Termos).
* **Trigger Field:** `dyn_tipodotermo` (Tipo de Termo).

#### Lógica de Execução (State Control):


* **Cenário: Seleção "Termo de Cronograma"**
    * **IF:** `dyn_tipodotermo` Equals `Termo de Cronograma`.
    * **THEN (Actions):**
        * `SetVisibility` (**True**) para a Grid/Subgrid de Cronograma.
        * `SetVisibility` (**False**) para todas as outras grids de termos (ex: Termo de Entrega, Termo de Aceite).
    * **ELSE (Default State):**
        * Mantém as grids ocultas até que um tipo válido seja selecionado.

#### Detalhamento Técnico da Grid:
* **Entidade Relacionada:** `dyn_ordemderemessa`.
* **Data Mapping:** A subgrid exibe colunas dinâmicas (ID da Remessa, Prazo Limite, Plano, Quantidade,Status da Linha ) originadas da tabela Ordem de Remessa, permitindo a edição in-line ou visualização rápida sem sair do contexto da Cotação.

---

##  3. Engenharia de Automação (Power Automate)

O backend foi desacoplado em dois fluxos de trabalho (micro-serviços) para garantir modularidade e facilitar o debug.



###  Fluxo I: Automação de Abertura de Ocorrências 

Este fluxo é um processo recorrente (Recurrence) que atua como um monitor de prazos, garantindo que nenhuma remessa fique atrasada sem o devido suporte.

#### 1. Gatilho e Filtragem Inicial
O fluxo inicia com uma recorrência diária e executa uma listagem de linhas na tabela **Ordem de Remessa** com um filtro OData rigoroso para performance:
* **Lógica do Filtro:** Busca apenas registros ativos (`statuscode eq 1`), que ainda estão pendentes (`dyn_statusdalinha eq 1`) e cujo prazo limite está entre hoje e os próximos 5 dias.
* **Expressão utilizada:** `dyn_prazolimite le '@{addDays(utcNow(), 5, 'yyyy-MM-dd')}' and dyn_prazolimite ge '@{utcNow('yyyy-MM-dd')}'`

#### 2. Processamento e Enriquecimento de Dados (Loop)
Para cada remessa encontrada, o fluxo entra em um "Aplicar a cada" (Apply to each) e realiza as seguintes etapas de busca para consolidar as informações:

* **Obter Cotação (Lookup):** Utiliza o ID `_dyn_cotacao_value` vindo da remessa para buscar detalhes da cotação pai (como o Nome do Cliente e a Lista de Preços).
* **Obter Produto (Lookup):** Utiliza o ID `_dyn_plano_value` para buscar metadados do produto, especificamente o **Código Fiscal / Código SAP**, que é essencial para o operacional.

#### 3. Criação da Ocorrência (Incident) com Dados Consolidados
A ação de "Adicionar uma nova linha" na tabela de **Ocorrências** é populada com dados cruzados de 3 tabelas diferentes:

* **Assunto Fixo:** Vinculado via OData ID à árvore de suporte: `/subjects(99999999-8888-8888-7777-000d3a378f36)`.
* **Título Dinâmico:** Montado com a estrutura: `ID da Cotação` + `Nome do Cliente` + `Solicitação: Envio de Remessa`.
* **Proprietário:** Atribuição direta para o time responsável via `/teams(GUID_DO_TIME)`.
* **Corpo do Argumento (Detalhado):**
    * **Remessa:** Injeta o Nome amigável da Remessa.
    * **Código Fiscal:** Injeta o Código SAP recuperado da tabela de Produtos.
    * **Lista de Preço:** Injeta o nome da lista recuperado da Cotação (usando `@OData.Community.Display.V1.FormattedValue`).
* **Rastreabilidade:** O campo lookup customizado `Carrier (Contas)` é preenchido com a referência da conta do cliente: `accounts(ID_DO_CLIENTE)`.

#### 4. Atualização de Status (State Management)
Após criar a ocorrência com sucesso, o fluxo executa uma atualização no registro de **Ordem de Remessa** original:
* **Ação:** Atualizar uma linha.
* **Alteração:** O campo `Status da Linha` é alterado para **"Em Processo" (Value 2)**. 
* **Objetivo:** Isso garante a integridade do processo, impedindo que o mesmo registro seja processado novamente na próxima execução do job.
### Fluxo II: Gatilho de Fechamento (State Machine)
Job automático que após a resolução de um chamado, verifica se a ocorrência criada pelo gatilho possui o guid da remessa salvo no campo: dyn_ordemderemessa da tabela OcorrÊncia, se sim, ela irá atualizar o status da linha da Remessa que teve seu chamado encerrado com status: Concluída.
Verifica a resolução de pendências externas para encerrar o ciclo de vida do registro pai.
* **OData Query:** `statecode eq 1 and _dyn_ordemderemessa_value ne null`
* **Atualizar uma linha da table Ordem remessa):**
  ** fim
  
##  4. Engine de Geração de Documentos Dinâmicos (Template Engine & Token Replacement)

A solução utiliza um fluxo de alta complexidade para a renderização de termos jurídicos/comerciais, baseando-se em um modelo de **Template HTML** armazenado no Dataverse e processado via substituição de tokens.

### A. Arquitetura do Fluxo (Pipeline de Dados)
O fluxo foi desenhado para ser agnóstico ao conteúdo, buscando a estrutura HTML de uma tabela de parametrização e injetando os dados em tempo de execução.

1.  **Contextualização (Data Fetching):**
    * **Obter Cotação/Termo:** O fluxo recupera o registro pai (`quote`) e o modelo de termo vinculado (`_dyn_tipo_termo_value`).
    * **Obter HTML:** Busca o código-fonte bruto (HTML/CSS) armazenado na coluna de configuração da tabela de Termos.
    * **Obter Cliente:** Recupera os metadados da conta (`customerid`) para personalização do documento.

2.  **Processamento de Itens (Looping & Logic):**
    * **Data Transformation:** Utilização de expressões para tratar campos nulos e formatação regional.
    * **Expressão de Tratamento de Data (SLA):**
        ```text
        if(empty(item()?['dyn_prazolimite']), '---', formatDateTime(convertTimeZone(item()?['dyn_prazolimite'], 'UTC', 'E. South America Standard Time'), 'dd/MM/yyyy'))
        ```
    * **Append Dinâmico:** Construção da estrutura `<tr>` injetando o `FormattedValue` (Display Name) dos campos de Lookup (`_dyn_plano_value` e `_dyn_cotacao_value`).

3.  **Algoritmo de Substituição (Token Replacement):**
    Para evitar a manipulação direta de strings complexas, utilizei uma cadeia de funções `replace` encadeadas (Pipeline de Composição) para injetar os dados nos placeholders do HTML:
    * `{{RAZAO_SOCIAL}}` ➔ Nome do Cliente (Conta).
    * `{{TABELA_COMPROMISSO}}` ➔ Variável de string contendo as linhas geradas no loop.
    * `{{OBSERVACOES_GERAIS}}` ➔ Tratado com a função `coalesce` para evitar falhas em campos de texto vazios.
    * `{{CIDADE_DATA}}` ➔ Dados de localidade do registro.

### B. Especificações Técnicas (Expressions & Functions)
* **Coalesce:** Utilizado para garantir que o fluxo não quebre caso as observações estejam em branco: `coalesce(outputs('Obter_Cotação')?['body/dyn_observacaodopedido'], '')`.
* **OData Formatted Value:** Extração de rótulos amigáveis de Lookups sem a necessidade de novos requests `Get Row`, otimizando o consumo de API.
* **Output Final:** O HTML processado é retornado para a interface (Power Apps/Cloud Flow) como uma string sanitizada e pronta para impressão ou conversão em PDF.



### C. Refatoração de Integridade Visual
* Inclusão de **Inline CSS** diretamente nas tags (`border`, `padding`, `text-align`) para garantir que o estilo seja preservado durante a conversão do navegador ou motor de PDF.
* Sincronização entre as colunas da Grid no Dataverse e as células da tabela HTML, removendo dados de controle interno e focando na transparência para o cliente final.
## 🛠️ Stack Técnica
* **Database:** Microsoft Dataverse.
* **Logic:** Power Automate (Cloud Flows).
* **Query Language:** OData Protocol.
* **Frontend:** Model-Driven Apps.
* **Formatting:** HTML5 / CSS3.


##  Conceitos de Desenvolvimento e Engenharia de Software Aplicados

A implementação seguiu princípios rigorosos de desenvolvimento para garantir que a solução seja resiliente, performática e de fácil manutenção.

* **Programação Defensiva (Null-Coalescing):** Utilização das funções `coalesce` e `if(empty())` em expressões lógicas para o tratamento de exceções em campos opcionais, prevenindo falhas de execução (*runtime errors*) durante a renderização de documentos.
* **Otimização de Payload e Performance:** Implementação de **Server-side Filtering** via protocolos OData e seleção de colunas específicas. Isso reduz a latência de rede e o overhead de processamento ao trafegar apenas o JSON estritamente necessário.
* **Abstração via Template Engine (Token Replacement):** Separação clara entre a camada de dados e a camada de apresentação. O uso de *placeholders* (tokens) no HTML permite que o layout seja atualizado sem a necessidade de alterar a lógica central do backend.
* **Integridade Referencial e GUIDs:** Gerenciamento de relacionamentos N:1 utilizando identificadores globais únicos (**GUIDs**). Essa prática elimina a dependência de nomes amigáveis (strings), que são voláteis, garantindo que o vínculo entre Cotação, Termos e Produtos seja imutável e preciso.
* **Desacoplamento de Processos:** Arquitetura baseada em gatilhos independentes para monitoramento de SLA e fechamento de ciclos, permitindo que cada micro-processo seja escalado ou corrigido sem afetar o fluxo principal de geração de documentos.







# Ajuste de Ribbon no Dynamics 365 (Ribbon Workbench)

## Objetivo

Documentar o processo de ajuste de botões na Ribbon do Dynamics 365 utilizando o plugin **Ribbon Workbench** (XrmToolBox), descrevendo a estrutura da Ribbon, boas práticas e o ajuste realizado na nomenclatura do botão **Qualificar Lead**.

---

## Ferramentas Utilizadas

* Power Apps
* XrmToolBox
* Plugin: Ribbon Workbench

---

## Seleção da Solution

Para alterar a Ribbon, é necessário selecionar no **Ribbon Workbench** a mesma **Solution** utilizada no Power Apps, referente à entidade que será ajustada.

Exemplos de solutions:

* `ribbon_lead`
* `ribbon_contas`
* `ribbon_incident`

Cada solution corresponde à Ribbon de uma entidade específica.
Sempre valide se a entidade correta foi selecionada antes de aplicar qualquer alteração.

---

## Estrutura da Ribbon

Ao abrir a Ribbon no Ribbon Workbench, são exibidas três linhas principais:

### Home

* Exibida na tela de **listagem de registros (views)**
* Exemplo: lista de Leads, lista de Contas
* Utilizada para ações em massa ou ações antes da abertura de um registro

### Subgrid

* Exibida dentro de formulários, nos **subgrids (listas relacionadas)**
* Exemplo: subgrid de Contatos dentro de uma Conta
* Os botões atuam somente sobre os registros exibidos naquele subgrid

### Form

* Exibida quando um **registro está aberto**
* Utilizada para ações diretas sobre o registro
* Exemplo: Qualificar Lead, Desqualificar Lead, Salvar, Fechar

> Normalmente, os ajustes são realizados na Ribbon **Form**, pois ela concentra os principais botões de negócio utilizados pelos usuários.

---

## Ajuste Realizado

Após selecionar a solution correta e a Ribbon do tipo **Form**, os botões foram posicionados conforme solicitado, respeitando a estrutura padrão do Dynamics.

Foi realizada a alteração da **Label (rótulo exibido ao usuário)** do botão padrão:

* **Qualificar Lead** → **Converter em Conta**

### Importante

* Apenas a **Label** foi alterada
* O **nome lógico** do botão não foi modificado
* A **lógica padrão** do sistema foi mantida

Ao clicar no botão **Converter em Conta**, o Dynamics executa exatamente o mesmo comportamento do botão padrão **Qualificar Lead**, sem impacto em regras de negócio, fluxos ou automações existentes.

---
## Comportamento de Duplicidade de Botões na Ribbon

Durante o ajuste de posicionamento de botões padrão na Ribbon, pode ocorrer do botão aparecer duplicado na interface do Dynamics, mesmo não aparentando duplicidade no Ribbon Workbench.

Isso acontece porque o botão padrão permanece vinculado ao seu grupo original, e ao ser arrastado para outro local, passa a ser exibido em mais de um grupo da Ribbon. O Ribbon Workbench exibe a estrutura lógica do botão, mas não evidencia todas as renderizações finais na interface.

Para evitar duplicidade visual, é necessário remover o botão do grupo original e mantê-lo apenas no novo grupo desejado.

Após a publicação, recomenda-se limpar o cache do navegador e validar o comportamento em diferentes registros e sessões.


---

## Boas Práticas Aplicadas

* Nenhum botão padrão foi removido
* Nenhuma lógica interna foi recriada
* Nenhum script customizado foi adicionado
* Apenas ajustes visuais e de posicionamento foram realizados

Essa abordagem garante:

* Estabilidade do ambiente
* Compatibilidade com futuras atualizações da Microsoft
* Preservação do fluxo de negócio padrão do Dynamics

---

## Observações Técnicas

* O cabeçalho do formulário possui limitações nativas
* Campos exibidos no cabeçalho tornam-se somente leitura
* Botões de Qualificar e Desqualificar Lead não são suportados nativamente no cabeçalho de forma editável
* A área recomendada para essas ações é a **Ribbon (barra superior)**

---

## Conclusão

Manter os botões padrão no local original da Ribbon e realizar apenas ajustes de nomenclatura via **Label** é a abordagem mais segura e recomendada, evitando impactos técnicos e retrabalho futuro.

---

Se quiser, posso:

* criar uma versão **ultra resumida** do README
* adicionar um **passo a passo com imagens (descrição dos prints)**
* ou padronizar o README com template corporativo

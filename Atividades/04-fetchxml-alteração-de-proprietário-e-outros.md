# FetchXML - Contas por CNPJ (Dynamics 365 / Dataverse)

Este `FetchXML` foi desenvolvido para **retornar todas as contas (entity: account)** cujo campo personalizado `CNPJ_CPF` corresponde a um dos valores informados.  
A consulta pode ser usada para **alterações em massa de proprietário** ou **atualização de campos personalizados** dentro do **XrmToolBox**.

---

## 🎯 Objetivo

Selecionar um conjunto específico de contas com base em seus CNPJs, permitindo posteriormente:

- Atualizar em massa o **proprietário (ownerid)**  
- Aplicar outras operações, como atualização de campos, etc.

---

## Estrutura do FetchXML

```sql
<fetch version="1.0" output-format="xml-platform" mapping="logical" distinct="true">
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="accountnumber" />
    <attribute name="ownerid" />
    <attribute name="CNPJ_CPF" /> --- ---- deve verificar no banco o NOME LÓGICO DO campo cnpj/cpf
    <filter type="or">
      <condition attribute="CNPJ_CPF" operator="eq" value="COLAR_AQUI_CNPJ_OU_CPF" />
    </filter>
  </entity>
</fetch>
```


# Explicação Técnica

## 1. Cabeçalho do Fetch

```sql
<fetch version="1.0" output-format="xml-platform" mapping="logical" distinct="true">

| Parâmetro | Descrição |
|-----------|-----------|
| `distinct="true"` | Evita duplicidade de registros |
| `mapping="logical"` | Usa os nomes lógicos dos atributos (não os exibidos na interface) |
| `version="1.0"` | Versão do formato FetchXML |
| `output-format="xml-platform"` | Formato de saída dos dados |
```
# 3. Atributos Retornados

```sql
<attribute name="accountid" />
<attribute name="accountnumber" />
<attribute name="ownerid" />
<attribute name="CNPJ_CPF" /> ---- deve verificar no banco o NOME LÓGICO DO campo cnpj/cpf
```

##  **Campos que serão retornados na consulta:**

| Campo | Descrição |
|-------|-----------|
| **`accountid`** | *Identificador único da conta* (usado para **updates** e **relacionamentos**) |
| **`accountnumber`** | *Número da conta*, geralmente o **código interno** da empresa |
| **`ownerid`** | *Proprietário atual* da conta (`usuário` ou `equipe`) |
| **`CNPJ_CPF`** | Campo **personalizado** que identifica o **CNPJ** da empresa |

### 🔧 Status dos Atributos:
- [x] `accountid` - **Implementado**
- [x] `accountnumber` - **Implementado**
- [x] `ownerid` - **Implementado** 
- [x] `CNPJ_CPF` - **Implementado**
---

# 4. Filtro (Condições)

```sql
<filter type="or">
  <condition attribute="CNPJ_CPF" operator="eq" value="COLAR_AQUI_CNPJ_OU_CPF" />
  <condition attribute="CNPJ_CPF" operator="eq" value="COLAR_AQUI_CNPJ_OU_CPF" />

</filter>
```

## **Características do Filtro:**

> **Nota importante:** O filtro usa `type="or"`, o que significa que **qualquer registro** que atenda a **pelo menos uma** das condições será retornado.

### Condições Aplicadas:

1. **CNPJ:** `COLAR_AQUI_CNPJ_OU_CPF`
2. **CNPJ:** `COLAR_AQUI_CNPJ_OU_CPF` 


### **Especificações Técnicas:**
- **Tipo de Filtro:** `OR`
- **Atributo Filtrado:** `CNPJ_CPF`
- **Operador:** `eq` (equal/igual)
- **Quantidade de Condições:** 5

---

### **Explicação Detalhada:**

Cada `<condition>` compara o valor do campo **`CNPJ_CPF`** com um número específico de CNPJ usando o operador **`eq`** (*equal*).

**Fluxo lógico:**
```
SE CNPJ_CPF = "COLAR_AQUI_CNPJ_OU_CPF1" OU
SE CNPJ_CPF = "COLAR_AQUI_CNPJ_OU_CPF2" OU  
SE CNPJ_CPF = "COLAR_AQUI_CNPJ_OU_CPF3" OU
SE CNPJ_CPF = "COLAR_AQUI_CNPJ_OU_CPF4" OU
SE CNPJ_CPF = "COLAR_AQUI_CNPJ_OU_CPF5"
ENTÃO retorna registro
```

[Documentação Completa](https://example.com/filter-docs) | [🔗 Exemplos Práticos](https://example.com/examples)

---
###Autor

Thiago Souza

Power Platform | Dynamics 365 | Automação de Processos
**Próximas Ações:**  - EM CONSTRUÇÃO..
- [ ] Validar formato dos CNPJs
- [ ] Testar performance da query
- [ ] Documentar resultados obtidos
- [ ] Revisar índices do campo CNPJ_CPF


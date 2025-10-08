# FetchXML - Contas por CNPJ (Dynamics 365 / Dataverse)

Este `FetchXML` foi desenvolvido para **retornar todas as contas (entity: account)** cujo campo personalizado `CNPJ_CPF` corresponde a um dos valores informados.  
A consulta pode ser usada para **alterações em massa de proprietário, inativação de contas** ou **atualização de campos personalizados** dentro do **XrmToolBox**.

---

## 🎯 Objetivo

Selecionar um conjunto específico de contas com base em seus CNPJs, permitindo posteriormente:

- Atualizar em massa o **proprietário (ownerid)**  
- **Inativar** contas  
- Aplicar outras operações, como alteração de status, atualização de campos, etc.

---

## 🧩 Estrutura do FetchXML

```xml
<fetch version="1.0" output-format="xml-platform" mapping="logical" distinct="true">
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="accountnumber" />
    <attribute name="ownerid" />
    <attribute name="CNPJ_CPF" />
    <filter type="or">
      <condition attribute="CNPJ_CPF" operator="eq" value="12345678000190" />
      <condition attribute="CNPJ_CPF" operator="eq" value="45987321000110" />
      <condition attribute="CNPJ_CPF" operator="eq" value="03222555000177" />
      <condition attribute="CNPJ_CPF" operator="eq" value="09876543000122" />
      <condition attribute="CNPJ_CPF" operator="eq" value="71102330000108" />
    </filter>
  </entity>
</fetch>
# 🧠 Explicação Técnica

## 1. Cabeçalho do Fetch

```xml
<fetch version="1.0" output-format="xml-platform" mapping="logical" distinct="true">

| Parâmetro | Descrição |
|-----------|-----------|
| `distinct="true"` | Evita duplicidade de registros |
| `mapping="logical"` | Usa os nomes lógicos dos atributos (não os exibidos na interface) |
| `version="1.0"` | Versão do formato FetchXML |
| `output-format="xml-platform"` | Formato de saída dos dados |

# 3. Atributos Retornados

```xml
<attribute name="accountid" />
<attribute name="accountnumber" />
<attribute name="ownerid" />
<attribute name="CNPJ_CPF" />
```

## 📊 **Campos que serão retornados na consulta:**

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

```xml
<filter type="or">
  <condition attribute="CNPJ_CPF" operator="eq" value="12345678000190" />
  <condition attribute="CNPJ_CPF" operator="eq" value="45987321000110" />
  <condition attribute="CNPJ_CPF" operator="eq" value="03222555000177" />
  <condition attribute="CNPJ_CPF" operator="eq" value="09876543000122" />
  <condition attribute="CNPJ_CPF" operator="eq" value="71102330000108" />
</filter>
```

## 🎯 **Características do Filtro:**

> **Nota importante:** O filtro usa `type="or"`, o que significa que **qualquer registro** que atenda a **pelo menos uma** das condições será retornado.

### 📋 Condições Aplicadas:

1. **CNPJ:** `12345678000190`
2. **CNPJ:** `45987321000110` 
3. **CNPJ:** `03222555000177`
4. **CNPJ:** `09876543000122`
5. **CNPJ:** `71102330000108`

### ⚙️ **Especificações Técnicas:**
- **Tipo de Filtro:** `OR`
- **Atributo Filtrado:** `CNPJ_CPF`
- **Operador:** `eq` (equal/igual)
- **Quantidade de Condições:** 5

---

### 💡 **Explicação Detalhada:**

Cada `<condition>` compara o valor do campo **`CNPJ_CPF`** com um número específico de CNPJ usando o operador **`eq`** (*equal*).

**Fluxo lógico:**
```
SE CNPJ_CPF = "12345678000190" OU
SE CNPJ_CPF = "45987321000110" OU  
SE CNPJ_CPF = "03222555000177" OU
SE CNPJ_CPF = "09876543000122" OU
SE CNPJ_CPF = "71102330000108"
ENTÃO retorna registro
```

[📖 Documentação Completa](https://example.com/filter-docs) | [🔗 Exemplos Práticos](https://example.com/examples)

---

**Próximas Ações:**
- [ ] Validar formato dos CNPJs
- [ ] Testar performance da query
- [ ] Documentar resultados obtidos
- [ ] Revisar índices do campo CNPJ_CPF

*Precisa de mais algum elemento específico?* ✅


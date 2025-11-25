# 📝 Roteiro: Desativar/Ativar Contas em Massa via XrmToolBox + FetchXML + DataBulk Updater - D365 Customer Service

## 1. Preparar o FetchXML

Use o **FetchXML Builder** no XrmToolBox para criar sua consulta.

### Exemplo 1: Selecionar contas por CPF/CNPJ

```xml
<fetch distinct="false" no-lock="true">
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="accountnumber" />
    <attribute name="name" />
    <attribute name="cpf_cnpj" />
    <attribute name="statecode" />
    <filter type="and">
      <condition attribute="cpf_cnpj" operator="in"> 
        <value>12.345.678/0001-01</value> 
        <value>23.456.789/0001-02</value>
        <value>12345678945</value> 
      </condition>
    </filter>
  </entity>
</fetch>
```

### Exemplo 2: Selecionar contas inativas

```xml
<fetch distinct="false" no-lock="true">
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="statecode" />
    <filter type="and">
      <condition attribute="statecode" operator="eq" value="1" /> <!-- 1 = Inativo -->
    </filter>
  </entity>
</fetch>
```

### Exemplo 3: Selecionar contas criadas nos últimos 30 dias

```xml
<fetch distinct="false" no-lock="true">
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="createdon" />
    <filter type="and">
      <condition attribute="createdon" operator="last-x-days" value="30" />
    </filter>
  </entity>
</fetch>
```

### Exemplo 4: Selecionar contas por proprietário

```xml
<fetch distinct="false" no-lock="true">
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="ownerid" />
    <filter type="and">
      <condition attribute="ownerid" operator="eq" value="GUID_DO_USUARIO" />
    </filter>
  </entity>
</fetch>
```

## Observações importantes

* Verifique o **nome lógico real do campo** que armazena o CNPJ/CPF no Dataverse (`cpf_cnpj` no exemplo).
* Adicione todos os valores que deseja filtrar dentro da tag `<condition>` usando `<value>`.

---

## 2. Executar o FetchXML no XrmToolBox

1. Abra o **XrmToolBox**.
2. Abra o **FetchXML Builder**.
3. Cole o FetchXML na janela de consulta.
4. Clique em **Execute** ou **Run** para listar os registros que serão afetados.
5. Verifique se os registros retornados estão corretos.

---

## 3. Enviar ao Bulk Data Updater

1. Após executar o FetchXML, clique no botão **Send** (enviar) no FetchXML Builder.
2. O FetchXML será enviado para o **Bulk Data Updater** (lado direito do XrmToolBox).
3. Na tela do Bulk Data Updater, você verá opções como:

   * **Update** → Atualiza campos específicos do registro.
   * **Assign** → Atribui a outro proprietário.
   * **Set State** → Altera o estado do registro (Ativo/Inativo).

---

## 4. Alterar o estado das contas

### Para inativar contas:

1. Escolha a opção **Set State**.
2. Selecione:

   * **State**: Inactive (1)
   * **Status**: Motivo apropriado (ex: Inativo = 2)

### Para ativar contas:

1. Escolha a opção **Set State**.
2. Selecione:

   * **State**: Active (0)
   * **Status**: Motivo apropriado (ex: Ativo = 1)

---

## 5. Executar a atualização em massa

1. Clique no botão **Start** ou **Run** no Bulk Data Updater.
2. Aguarde a execução terminar.
3. Verifique no **Dynamics 365** se os registros foram atualizados corretamente (`statecode` e `statuscode`).

---

## 6. Dicas finais

* Sempre **fazer um teste** com 2 ou 3 registros antes de rodar em massa.
* Certifique-se de que os CNPJs/CPFs estão corretos e no formato que o Dataverse reconhece.
* **Atenção:** alterar apenas `statuscode` **não inativa a conta**; o `statecode` precisa ser atualizado junto.

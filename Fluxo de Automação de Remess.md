### 🚀 Fluxo de Automação de Remessas (Finanças & Fulfillment)

```mermaid
graph TD
    %% Nós de Início e Fim
    Start([Criação da Remessa]) --> Fin(Gerar Ocorrência: Fila Finanças)
    
    %% Bloco Finanças
    Fin --> Dec1{Análise Financeira}
    Dec1 -- Reprovado --> StatusRep(Status: Crédito Reprovado)
    Dec1 -- Aprovado --> StatusApr(Status: Crédito Aprovado)

    %% Bloco Logística
    StatusApr --> Wait[Aguardar Prazo Limite]
    Wait --> Full(Gerar Ocorrência: Fila Fulfillment)
    Full --> StatusProc(Status: Em Processo de Entrega)
    StatusProc --> Dec2{Análise de Entrega}

    %% Resultados Finais
    Dec2 -- Falha --> StatusFail(Status: Não Entregue)
    Dec2 -- Sucesso --> StatusOK(Status: Entregue)

    %% Estilização (O "Front-end" do gráfico)
    classDef finance fill:#e1f5fe,stroke:#01579b,color:#01579b
    classDef logistics fill:#f3e5f5,stroke:#4a148c,color:#4a148c
    classDef success fill:#c8e6c9,stroke:#2e7d32,color:#1b5e20
    classDef error fill:#ffcdd2,stroke:#c62828,color:#b71c1c

    class Fin,Dec1 finance
    class Full,StatusProc,Dec2 logistics
    class StatusApr,StatusOK success
    class StatusRep,StatusFail error
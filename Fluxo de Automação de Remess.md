```mermaid
graph LR
    %% Organização em Subgrafos para não achatar
    subgraph SETOR_FINANCEIRO [Processo de Crédito]
        Start([Criação da Remessa]) --> Fin[Gerar Ocorrência:<br/>Fila Finanças]
        Fin --> Dec1{Análise<br/>Financeira}
        Dec1 -- Reprovado --> StatusRep[Status:<br/>Fin. Reprovado]
        Dec1 -- Aprovado --> StatusApr[Status:<br/>Fin. Aprovado]
    end

    %% Transição para o Meio
    StatusApr --> Prazo
    StatusRep --> Prazo

    subgraph MOTOR_DECISAO [Inteligência de Prazo e Status]
        Prazo{Chegou o<br/>Prazo Limite?} -- Sim --> ValidaStatus{O Status é<br/>'Fin. Aprovado'?}
        Prazo -- Não --> Aguarda[Aguardar Data]
        ValidaStatus -- Não --> NoFull[Fim: Bloqueio<br/>Financeiro]
    end

    %% Transição para Logística
    ValidaStatus -- Sim --> Full

    subgraph SETOR_LOGISTICO [Fulfillment e Entrega]
        Full[Gerar Ocorrência:<br/>Fila Entrega] --> StatusProc[Status: Em<br/>Processo de Entrega]
        StatusProc --> Dec2{Entrega<br/>Realizada?}
        Dec2 -- Sucesso --> StatusOK[Status:<br/>Entregue]
        Dec2 -- Falha --> StatusFail[Status:<br/>Não Entregue]
    end

    %% Estilização Técnica
    classDef finance fill:#e1f5fe,stroke:#01579b,color:#01579b
    classDef logistics fill:#f3e5f5,stroke:#4a148c,color:#4a148c
    classDef success fill:#c8e6c9,stroke:#2e7d32,color:#1b5e20
    classDef error fill:#ffcdd2,stroke:#c62828,color:#b71c1c
    classDef neutral fill:#f5f5f5,stroke:#616161,color:#616161

    class Fin,Dec1 finance
    class Full,StatusProc,Dec2 logistics
    class StatusApr,StatusOK success
    class StatusRep,StatusFail,NoFull error
    class Prazo,ValidaStatus neutral

graph LR
    %% Direção Inicial: Horizontal (LR)
    
    Start([Criação da Remessa]) --> Fin[Gerar Ocorrência: Fila Finanças]
    Fin --> Dec1{Análise Financeira}

    %% Se reprovado, desce para o "Fim" (Vertical)
    Dec1 -- Reprovado --> StatusRep[Status: Crédito Reprovado]
    
    %% Se aprovado, segue horizontal
    Dec1 -- Aprovado --> StatusApr[Status: Crédito Aprovado]

    %% O Fluxo "Sobe" para a análise de Prazo (Vertical)
    StatusApr --> Prazo{Chegou o Prazo Limite?}
    StatusRep --> Prazo

    %% A decisão do Prazo Limite desce para a Verificação de Status
    Prazo -- Sim --> ValidaStatus{O Status é <br/>'Crédito Aprovado'?}
    Prazo -- Não --> Aguarda[Aguardar Data]

    %% Lógica Final: Desce para o fechamento
    ValidaStatus -- Sim --> Full[Gerar Ocorrência: Fila Fulfillment]
    ValidaStatus -- Não --> NoFull[Processo Encerrado: <br/>Sem Envio de Mercadoria]

    Full --> StatusProc[Status: Em Processo de Entrega]
    StatusProc --> Dec2{Análise de Entrega}
    
    Dec2 -- Sucesso --> StatusOK[Status: Entregue]
    Dec2 -- Falha --> StatusFail[Status: Não Entregue]

    %% Estilização do "Front-end"
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

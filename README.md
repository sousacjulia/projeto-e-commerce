E‑Commerce – Modelo de Dados (ER/EER)

Projeto acadêmico de modelagem de dados para um sistema de e‑commerce (venda de produtos). O objetivo é transformar uma narrativa de requisitos em um diagrama ER/EER consistente e um esquema relacional pronto para implementação em MySQL.

    Baseado em exercício de modelagem: cadastro de clientes (PF/PJ), pedidos, produtos, estoque, fornecedores, pagamentos e entregas.

🔭 Objetivo

    Representar os principais processos do e‑commerce:
        cadastro de clientes (PF ou PJ);
        catálogo de produtos e fornecedores;
        criação de pedidos com múltiplos itens;
        controle de estoque por local;
        pagamentos (possivelmente múltiplos por pedido);
        entregas com status e código de rastreio.

🧭 Requisitos atendidos (da narrativa)

    Produto
        Produtos são vendidos por uma plataforma; podem ter vendedores/fornecedores distintos (marketplace).
        Cada pedido pode conter um ou mais produtos.

    Cliente
        Cliente pode se cadastrar com CPF (PF) ou CNPJ (PJ) — exclusivo, nunca os dois.
        Endereço do cliente influencia o valor do frete.
        Cliente pode comprar mais de um produto; existe período de carência para devolução.

    Pedido
        Criado por clientes e guarda informações de compra, endereço e status de entrega.
        Um ou mais produtos compõem o pedido.
        O pedido pode ser cancelado (tratado via status).

    Pagamento
        Um pedido pode ter uma ou mais formas de pagamento.

    Entrega
        Possui status e código de rastreio.

✅ Todos os pontos acima estão representados no modelo.
🗂️ Diagrama EER

Inclua o arquivo do diagrama no repositório e referencie aqui:

/docs/diagrams/ecommerce-eer.png

Diagrama EER

<img width="964" height="1099" alt="Diagrama - Ecommerce" src="https://github.com/user-attachments/assets/881440ac-9ec4-4600-b9df-72017dc29e21" />




🧱 Entidades & principais atributos
CLIENTE (entidade geral)

    idCliente (PK)
    nome, email, telefone

PF_CLIENTE (especialização)

    idClientePF (PK, FK → CLIENTE)
    cpf

PJ_CLIENTE (especialização)

    idClientePJ (PK, FK → CLIENTE)
    cnpj, razao_social

    Regra: uma conta é PF ou PJ. Implementação por exclusão (só existe linha em uma das tabelas filhas).

ENDERECO

    idEndereco (PK)
    idCliente (FK → CLIENTE)
    cidade, logradouro, numero, cep, complemento

PRODUTOS

    idProduto (PK)
    nome, categoria, valor

FORNECEDOR

    idFornecedor (PK)
    nome, razao_social, cnpj

FORNECEDOR_has_PRODUTOS (associativa – marketplace)

    idFornecedor (FK → FORNECEDOR)
    idProduto (FK → PRODUTOS)
    (opcional) preco_fornecedor, prazo_entrega_dias

    Obs.: Se o requisito for exatamente um fornecedor por produto, substitua a associativa por PRODUTOS.idFornecedor.

ESTOQUE

    idEstoque (PK)
    local

PRODUTOS_has_ESTOQUE (associativa)

    idProduto (FK → PRODUTOS)
    idEstoque (FK → ESTOQUE)
    quantidade

PEDIDO

    idPedido (PK)
    idCliente (FK → CLIENTE)
    idEndereco (FK → ENDERECO) – endereço de entrega faturado no pedido
    status (ex.: CRIADO, PAGO, ENVIADO, ENTREGUE, CANCELADO)
    descricao, frete, valor_total, data_pedido

Itens_Pedidos (detalhes do pedido)

    idProduto (FK → PRODUTOS)
    idPedido (FK → PEDIDO)
    quantidade, valor_unitario
    periodo_carencia (dias ou data limite p/ devolução)

Pagamento (1:N com Pedido)

    idPagamento (PK)
    idPedido (FK → PEDIDO)
    forma_pagamento (ex.: cartao, pix, boleto)
    valor
    data_pagamento

Entrega (1:1 com Pedido — ou 1:N se desejar parciais)

    idEntrega (PK)
    idPedido (FK → PEDIDO)
    status_entrega (ex.: separando, em_transporte, entregue)
    codigo_rastreio
    data_envio, data_prevista, data_entrega

🔗 Relacionamentos (cardinalidade)

    CLIENTE 1 — N ENDERECO
    CLIENTE 1 — N PEDIDO
    PEDIDO 1 — N Itens_Pedidos N — 1 PRODUTOS
    PEDIDO 1 — N PAGAMENTO (suporta múltiplas formas)
    PEDIDO 1 — 1 ENTREGA (ou 1 — N se habilitar entregas parciais)
    PRODUTOS N — N ESTOQUE (com quantidade na associativa)
    FORNECEDOR N — N PRODUTOS (marketplace)
    CLIENTE 1 — 1 PF_CLIENTE ou CLIENTE 1 — 1 PJ_CLIENTE (exclusivo)

🧠 Decisões de modelagem & suposições

    PF/PJ exclusivos: controlado em nível de aplicação e/ou constraints (trigger ou checagem) garantindo que um idCliente só exista em uma tabela filha.
    Frete: calculado com base no ENDERECO selecionado no momento da compra; o valor final fica gravado em PEDIDO.frete.
    Cancelamento: representado em PEDIDO.status.
    Devolução: gerenciada por Itens_Pedidos.periodo_carencia.
    Valores monetários: recomenda‑se DECIMAL(10,2) para campos de valor.

🏗️ Estrutura sugerida do repositório

.
├── docs/
│   └── diagrams/
│       └── ecommerce-eer.png
├── sql/
│   ├── schema.sql          # DDL completo (CREATE TABLE ...)
│   └── sample_data.sql     # Inserts de exemplo (opcional)
├── scripts/
│   └── queries.sql         # Consultas úteis de teste
└── README.md

▶️ Como usar (MySQL)

    Crie um banco: CREATE DATABASE ecommerce CHARACTER SET utf8mb4;
    Execute o sql/schema.sql com todas as tabelas na ordem correta.
    (Opcional) Rode sql/sample_data.sql para dados de teste.
    Execute as consultas de scripts/queries.sql.

    Se usar MySQL Workbench, mantenha e exporte o arquivo .mwb em docs/ para versionar o diagrama.

🔎 Consultas de exemplo

-- Total do pedido (itens + frete)
SELECT p.idPedido,
       SUM(i.quantidade * i.valor_unitario) + p.frete AS total
FROM Pedido p
JOIN Itens_Pedidos i ON i.idPedido = p.idPedido
GROUP BY p.idPedido;

-- Estoque disponível por produto
SELECT pr.idProduto, pr.nome,
       SUM(pe.quantidade) AS qtd_total
FROM Produtos pr
JOIN Produtos_has_Estoque pe ON pe.idProduto = pr.idProduto
GROUP BY pr.idProduto, pr.nome;

-- Pedidos e status de entrega
SELECT p.idPedido, e.status_entrega, e.codigo_rastreio
FROM Pedido p
LEFT JOIN Entrega e ON e.idPedido = p.idPedido;

-- Pagamentos por pedido
SELECT p.idPedido, pg.forma_pagamento, pg.valor
FROM Pedido p
JOIN Pagamento pg ON pg.idPedido = p.idPedido
ORDER BY p.idPedido;

✅ Checklist de conformidade

    Cliente PF ou PJ (exclusivo)
    Múltiplos endereços por cliente
    Pedido com múltiplos itens
    Período de carência por item
    Múltiplos pagamentos por pedido
    Entrega com status e código de rastreio
    Controle de estoque por local
    Produtos com fornecedores (marketplace opcional)


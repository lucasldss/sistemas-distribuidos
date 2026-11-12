# sistemas-distribuidos

# Central de Pedidos: Simulado de Sistemas Distribuídos

Simulação em Python de uma central de pedidos com **trabalhadores concorrentes**, um **servidor principal** e uma **réplica**. O projeto demonstra três conceitos de sistemas distribuídos: controle de concorrência (semáforo e lock), replicação de dados e failover.

- **Instituição:** Anhanguera Educacional
- **Disciplina:** Sistemas Distribuídos
- **Professora:** Prof.ª Dr.ª Simone Tatiane do Canto
- **Ambiente:** VS Code + Python (apenas biblioteca padrão)

## Comportamento esperado

| Requisito | Valor |
|---|---|
| Pedidos processados | 3 (números 101, 102 e 103) |
| Trabalhadores simultâneos | no máximo 2 |
| Estoque do principal | começa com 10 e termina com 7 |
| Estoque da réplica | 7 após a sincronização |
| Falha do principal | a consulta é atendida pela réplica |

## Como executar

1. Crie a pasta `simulado_sd` e salve o arquivo `central_pedidos.py` dentro dela.
2. Abra a pasta no VS Code e o terminal integrado.
3. Execute:

```bash
python central_pedidos.py
```

No Windows, se necessário:

```bash
py central_pedidos.py
```

## Como o código funciona

- **`processar_pedido(numero)`**: cada pedido roda em uma thread. O `semaforo` limita quantas threads trabalham ao mesmo tempo, e o `trava` (Lock) protege a atualização do estoque.
- **`sincronizar()`**: copia o estoque do servidor principal para a réplica.
- **`consultar_estoque()`**: responde com o principal se ele estiver online. Se estiver offline, usa a réplica (failover). Se ambos estiverem offline, retorna `INDISPONIVEL`.

## Correções realizadas

O código inicial tinha três erros, marcados por comentários no arquivo.

| Linha | Antes | Depois | Motivo |
|---|---|---|---|
| **A** | `MAXIMO_SIMULTANEO = 3` | `MAXIMO_SIMULTANEO = 2` | O requisito é no máximo 2 trabalhadores simultâneos. |
| **B** | `replica["estoque"] = replica["estoque"]` | `replica["estoque"] = principal["estoque"]` | A réplica copiava a si mesma e nunca recebia o valor do principal. |
| **C** | `if principal["online"]:` (segundo `if`) | `if replica["online"]:` | Com o principal offline a condição nunca era verdadeira, então a réplica não atendia. |

## Resultados

**Antes das correções**

```
=== CENTRAL DE PEDIDOS ===
Limite simultaneo: 3
Estoque inicial: 10
Pedidos: [101, 102, 103]
Estoque principal: 7
Replica antes: 10
Replica depois: 10
Consulta apos falha: INDISPONIVEL None
```

**Depois das correções**

```
=== CENTRAL DE PEDIDOS ===
Limite simultaneo: 2
Estoque inicial: 10
Pedidos: [101, 102, 103]
Estoque principal: 7
Replica antes: 10
Replica depois: 7
Consulta apos falha: REPLICA 7
```

## Análise técnica

**1. Qual o papel do semáforo?**
O semáforo limita quantas threads executam a seção de trabalho ao mesmo tempo. Aqui ele permite no máximo 2 trabalhadores simultâneos, e o terceiro pedido espera até que um dos outros termine.

**2. Por que a réplica precisava ser sincronizada?**
A réplica é uma cópia do principal e só serve como reserva se tiver os mesmos dados. Sem sincronização, ela mostraria 10 unidades (estoque desatualizado) e passaria informação errada caso assumisse o atendimento.

**3. O que é failover?**
É a troca automática para um servidor de reserva quando o principal falha, mantendo o serviço disponível. No código, a consulta passa a ser atendida por `REPLICA 7` quando o principal está offline.

**4. Por que o Lock ajuda a evitar problemas de concorrência?**
O Lock garante exclusão mútua: só uma thread por vez verifica e atualiza o estoque. Sem ele, duas threads poderiam ler o mesmo valor e sobrescrever uma à outra (condição de corrida), gerando estoque ou pedidos incorretos.

## Estrutura do projeto

```
simulado_sd/
├── central_pedidos.py
└── README.md
```

## Critérios de avaliação (1.000 pontos)

| Item | Pontos |
|---|---|
| Correção A | 200 |
| Correção B | 250 |
| Correção C | 250 |
| Execução e evidências | 150 |
| Justificativas técnicas | 150 |

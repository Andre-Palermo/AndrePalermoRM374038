# Tech Challenge

## 🎯 Objetivo

Descobrir causas da alta variabilidade do Net Promoter Score (NPS) entre diferentes perfis de consumidores mesmo com indicadores operacionais aparentemente semelhantes.

Investigar possiveis razões para que alguns clientes se tornam promotores da marca, enquanto outros se tornam detratores.

Quais fatores operacionais realmente influenciam/causam a satisfação do cliente e como a empresa pode agir de forma proativa para melhorar a experiência antes mesmo da aplicação da pesquisa de NPS.

Contexto atual: Atualmente, o NPS é coletado apenas após o encerramento da jornada de compra.

## 📊 Base de Dados

Dados históricos de pedidos, entregas e interações com o atendimento ao cliente. 

Arquivo: Base de dados NPS 

Exemplos de informações disponíveis: 

● Dados do pedido (valor, quantidade de itens, forma de pagamento); 
● Dados logísticos (tempo de entrega, atraso, tentativas); 
● Dados de atendimento (contatos, tempo de resolução); 
● Indicadores internos de negócio. 

## Dicionário de Dados

| Coluna | Descrição |
|--------|----------|
| customer_id | Identificador único do cliente |
| order_id | Identificador único do pedido |
| customer_age | Idade do cliente |
| customer_region | Região geográfica do cliente |
| customer_tenure_months | Tempo de relacionamento (meses) |
| order_value | Valor total do pedido |
| items_quantity | Quantidade de itens |
| discount_value | Valor de desconto |
| payment_installments | Número de parcelas |
| delivery_time_days | Tempo de entrega (dias) |
| delivery_delay_days | Dias de atraso |
| freight_value | Valor do frete |
| delivery_attempts | Tentativas de entrega |
| customer_service_contacts | Contatos com atendimento |
| resolution_time_days | Tempo de resolução |
| complaints_count | Número de reclamações |
| repeat_purchase_30d | Recompra em 30 dias (0/1) |
| csat_internal_score | Score interno |
| nps_score | Nota NPS (0 a 10) |

## 🔍 Análise Exploratória (EDA)

Principais insights Técnicos:

- Tamanho da da base de dados: 2500 linhas e 19 colunas
- Não há colunas com conteudos nulos
- Não há linhas duplicadas 
- Maioria das colunas do tipo numérica e somente uma coluna do tipo categórica (customer_region)
- Não foram encontrados conteúdos outliers

Ponto de atenção: 

- 158 linhas cuja coluna pontuacao_nps está zerada.
- 462 linhas cuja coluna pontuacao_satisfacao_interna está zerada.
- 106 linhas em que ambas colunas estão zeradas.

    Esses casos zerados podem significar:

    Não respondeu nenhuma pesquisa
    Dados faltantes codificados como 0(zero)
    Um comportamento real

    Isso pode enviesar análise de detratores, então vale considerar:

    Manter as linhas por ser um comportamento real
    Remover as linhas
    Tratar como missing e "inputar" valor que se enquadre como "neutro" (pontuação 7.5)

    Idealmente, seguindo as melhores praticas, um alinhamento com a área de negócios ou "data owners" é o recomendado para esclarecer esses casos.

    Por ora, a abordagem adotada deste tech challenge foi usar a abordagem de manter as linhas considerando a possibilidade de ser um comportamento real dos dados em produção.

Principais insights Negócio:

- NPS agregado final -66 (Péssimo)
- Quantidade de reclamações relacioandas ao atraso de entregas e demora na resolução para os problemas relatados pelo cliente durante sua experiência
- Quanto mais tempo levar para um cliente ter seu problema resolvido e/ou maior for o atraso na entrega a consequencia vai ser mais reclamações e menor vai ser sua pontuação CSAT (satisfação interna) e por consequência final menor vai ser sua pontuação NPS no pós-venda

## ⚙️ Metodologia

### Pré-processamento

- Encoding: Abordagem one-hot encoding para a coluna "regiao_cliente" para facilitar eventuais aplicações de modelos preditivos matemáticos. Coluna regiao_cliente possui 5 conteudos únicos, representados pelas 5(cinco) regioes do Brasil [Nordeste, Sul, Centro-Oeste, Norte, Sudeste]

### Modelagem e Resultados

- Foram implementados e exercitados 3(três) modelos:

    - REGRESSAO LINEAR (TARGET NUMERICO CONTINUO)

        🎯 Métricas
                RMSE: 1.69
                MAE: 1.33
                R²: 0.55

        🧠 Interpretação

            explica ~55% do comportamento do NPS
            erra em média ~1.3 pontos

        💥 Conclusão: modelo bom e utilizável

    - REGRESSAO LOGISTICA ( TARGET NPS CATEGORICO )

        🎯 Métricas
                Accuracy: 0.71
                ROC AUC: 0.84 💥
                Macro F1: 0.54

        🧠 Interpretação

            🟥 Detrator

                👉 291 acertos de 363

                recall ≈ 80%
                poucos falsos negativos

            🟡 Neutro

                👉 44 acertos de 98

                modelo ainda confunde bastante
                principalmente com promotor

                ⚠️ comportamento esperado

            🟢 Promotor

                👉 19 acertos de 39 (~49%)

                💥 grande evolução

        💥 Conclusão: modelo bom e utilizável
                    O modelo consegue diferenciar bem clientes insatisfeitos e apresenta boa capacidade geral de separação entre os níveis de satisfação (AUC 0.84), com maior dificuldade na identificação de perfis intermediários (neutros).

    - REGRESSAO LOGISTICA BINÁRIA ( TARGET NPS CATEGORICO )

        🎯 Métricas
                AUC: 0.877 💥 
                Accuracy: 0.80
                Recall (detrator): 0.79
                Precision (detrator): 0.92

        🧠 Interpretação

            ✔ separa muito bem clientes (AUC alto)
            ✔ identifica bem detratores
            ✔ erra pouco quando prevê detrator

            🟥 Detratores (classe 1)
                292 acertos
                78 perdidos

                👉 recall = 79%

                💥 Captura a maioria dos problemas

            🟩 Não detratores (classe 0)

                106 acertos
                24 falsos positivos

                👉 precision alto → modelo confiável

        💥 Conclusão: 
                modelo responde: “qual a chance desse cliente ser detrator?”
                modelo excelente para uso real

## 📈 Resultados

    Todos os modelos exercitados apresentam boas métricas, cabe à área de negócios a decisão de qual modelo implementar em produção ou atpe mesmo optar por implementar mais de um modelo exercitado neste tech challenge

## 🚀 Como reproduzir

    Executar o notebook no ambiente Collab mas antes fazer o upload do arquivo .csv que se encontra em data\dados_brutos

## 📌 Conclusão

De um ponto de vista de negócio, se ocorrer menos atraso nas entregas, ocorrer resolução mais rápida para os problemas do cliente e ocorrer melhor comunicação para o cliente das ações proativas do E-Commerce a consequencia deve ser menos reclamação, menos insatisfação e maior NPS

Melhorar operação = Melhorar NPS = Crescer a operação


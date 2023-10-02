# Estudo de precificação de cimento
Este projeto foi desenvolvido para realizar um estudo da precificação do cimento, entendendo como alguns fatores poderiam influenciar no preço do cimento. O estudo foi desenvolvido desde a coleta dos dados, construção de processos de pipeline de dados usando a plataforma de ETL DataPrep na nuvem em conjunto com a Google para carga dos dados no Bigquery,  até a análise final feita no Jupyter notebook.

As perguntas que desejamos responder são:

a) O preço do cimento esta positivamente correlacionado com o IPCA (Índice de Preços ao Consumidor Amplo)?

b) Existe uma relação de causalidade entre o consumo de cimento e a produção de cimento?
 - Esta pergunta pode ajudar a elucidar se o mercado se auto regula, produzindo algo próximo do que é consumido evitando excessos que poderiam fazer o preço do produto ter queda.
   
c) Há uma correlação negativa entre as admissões no setor de construção civil e o preço do cimento?

1. [Coleta dos dados](#coleta)
2. [Modelagem dos dados](#modelagem)
3. [File Descriptions](#files)
4. [Results](#results)
5. [Acknowledgments](#acknowledgments)
6. [Licensing and Author](#licensing)

## Coleta dos dados <a name="coleta"></a>

Foram coletados os seguintes dados através do datalake do governo hospedado no Bigquery da Google:

- <b>Índice Nacional de Preços ao Consumidor Amplo (IPCA)</b>: O IPCA é utilizado como indicador oficial do País desde 1985 para corrigir salários, aluguéis, taxa de câmbio, poupança, entre outros.  
- <b>Cadastro Geral de Empregados e Desempregados (CAGED)</b>: Foi criado como instrumento de acompanhamento e de fiscalização do processo de admissão e de dispensa de trabalhadores regidos pela CLT, com o objetivo de assistir os desempregados e de apoiar medidas contra o desemprego.
- <b>Cadastro Nacional de Obras (CNO)</b>: O cadastro nacional de obras (CNO) é uma base de dados de registro de obras de construção civil administrado pela Receita Federal.

Também foram coletados Câmara Brasileira da Indústria da Construção:

- Consumo de cimento por estado por mês e ano
- Produção de cimento por estado mês e ano
- Preço do cimento por estado mês e ano

Para coletar os dados levamos em consideração que o período amostral da menor amostra de dados que tinhamos, que no caso foi a tabela de cadastros nacional de obras, que abrangeu apenas dados do período de janeiro de 2018 até maio de 2021. Logo, este período foi utilizado para coletar as demais amostras. 

Foram utilizados os seguintes comandos SQL para coleta das amostras do datalake do governo hospedado na Google.

1) Índice Nacional de Preços ao Consumidor Amplo
- SELECT * FROM `basedosdados.br_ibge_ipca.mes_brasil`
   
2) Cadastro Geral de Empregados e Desempregados (CAGED)
Neste existem tabelas construídas com padrões diferentes, uma tabela com um padrão possui dados de 2007 a 2019. Outra tabela possui um padrão mais recente abrangendo dados de 2020 à 2023.

Como o objetivo era extrair dados através de um arquivo CSV, e o google limita downloads de até 1GB. Limitei os dados de forma a extrair apenas os dados dos trabalhadores da construção civil (que era o dado mais relevante para este trabalho) e particionei as tabelas para a extração uma por ano. Dessa forma foram utilizadas quatro consultas levando em consideração os dados apresentados anteriormente.

- SELECT * FROM `basedosdados.br_me_caged.microdados_antigos` WHERE subsetor_ibge = '15' AND ano = 2018;
- SELECT * FROM `basedosdados.br_me_caged.microdados_antigos` WHERE subsetor_ibge = '15' AND ano = 2019;

Onde subsetor_ibge = 15 => Setor de construção civil;

- SELECT * FROM `basedosdados.br_me_caged.microdados_movimentacao` WHERE cnae_2_secao = 'F' AND ano = 2020;
- SELECT * FROM `basedosdados.br_me_caged.microdados_movimentacao` WHERE cnae_2_secao = 'F' AND ano = 2021;

Onde cnae_2_secao = F => Setor de construção civil;

3) Cadastro Nacional de Obras (CNO)
- SELECT * FROM `basedosdados.br_me_cno.microdados_vinculo`;

## Modelagem dos dados<a name="modelagem"></a>

Através de todos estes dados foi construída uma única tabela que envolve dados por estados, mês e ano de cada uma das tabelas fontes.
Um print do catálogo dos dados pode ser visto neste repositório, com a descrição detalhada dos dados e domínios.

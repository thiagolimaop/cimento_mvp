# Estudo de precificação de cimento
Este projeto foi desenvolvido para realizar um estudo da precificação do cimento, entendendo como alguns fatores poderiam influenciar no preço do cimento. O estudo foi desenvolvido desde a coleta dos dados, construção de processos de pipeline de dados usando a plataforma de ETL DataPrep na nuvem em conjunto com a Google para carga dos dados no Bigquery,  até a análise final feita no Jupyter notebook.

As perguntas que desejamos responder são:

a) O preço do cimento esta positivamente correlacionado com o IPCA (Índice de Preços ao Consumidor Amplo)?

b) Existe uma relação de causalidade entre o consumo de cimento e a produção de cimento?
 - Esta pergunta pode ajudar a elucidar se o mercado se auto regula, produzindo algo próximo do que é consumido evitando excessos que poderiam fazer o preço do produto ter queda.
   
c) Há uma correlação negativa entre as admissões no setor de construção civil e o preço do cimento?

1. [Coleta dos dados](#coleta)
2. [Modelagem dos dados](#modelagem)
3. [Processo ETL](#etl_process)
4. [Análise de dados](#data_analitics)

## Coleta dos dados <a name="coleta"></a>

Foram coletados os seguintes dados através do datalake do governo hospedado no Bigquery da Google:

- <b>[Índice Nacional de Preços ao Consumidor Amplo](https://basedosdados.org/dataset/ea4d07ca-e779-4d77-bcfa-b0fd5ebea828?table=f1fd2eb7-467a-403b-8f1c-2de8eff354e6)</b>: O IPCA é utilizado como indicador oficial do País desde 1985 para corrigir salários, aluguéis, taxa de câmbio, poupança, entre outros.  
- <b>[Cadastro Geral de Empregados e Desempregados (CAGED)](https://basedosdados.org/dataset/562b56a3-0b01-4735-a049-eeac5681f056?table=2245875f-d1ef-490d-be29-4f8fb2191335)</b>: Foi criado como instrumento de acompanhamento e de fiscalização do processo de admissão e de dispensa de trabalhadores regidos pela CLT, com o objetivo de assistir os desempregados e de apoiar medidas contra o desemprego.
- <b>[Cadastro Nacional de Obras (CNO)](https://basedosdados.org/dataset/88580166-bd00-4f73-b86b-97f3a3515b36?table=157149ea-7191-4e3e-a5ae-47b2a1671d3b)</b>: O cadastro nacional de obras (CNO) é uma base de dados de registro de obras de construção civil administrado pela Receita Federal.

Também foram coletados Câmara Brasileira da Indústria da Construção:

- [Consumo de cimento por estado por mês e ano](http://www.cbicdados.com.br/menu/materiais-de-construcao/cimento)
- [Produção de cimento por estado mês e ano](http://www.cbicdados.com.br/menu/materiais-de-construcao/cimento)
- [Preço do cimento por estado mês e ano](http://www.cbicdados.com.br/menu/materiais-de-construcao/cimento)

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
Um print do catálogo dos dados pode ser visto neste repositório, através do nome "catalogo_dados.jpg" com a descrição detalhada dos dados e domínios.
A linhagem de dados gerada pelo google bigquery também pode ser vista através dad imagem "linhagem_dados.jpg".

## Processo ETL<a name="etl_process"></a>

Uma imagem do processo geral de pipeline de dados construido na ferramenta do DataPrep pode ser vista na imagem "pipeline_overview_dataprep.jpg". Nesta imagem a primeira coluna representa os arquivos fontes .csv carregados na ferramenta. A segunda coluna representa os processos. Cada processo foi documentado dentro da pasta ETL, alguns que fornecem processos bem semelhates foram colocados apenas uma vez. Por exemplo podemos observar na imagem "pipeline_overview_dataprep.jpg" uma caixinha com o nome amostra-2018-i, na pasta ETL, existe uma imagem de mesmo nome desta caixinha que representa os processos de tratamento de dados aplicado nesta caixa. Especificamente para esta última caixa não existe nenhum processo de tratamento de dados, pois foi apenas colhida uma nova amostra para se ter uma representatividade de todos os estados e meses representados nos dados.

A caixinha denominada caged por exemplo que tem uma imagem de mesmo nome mostra uma amostra dos dados, a distribuição dos dados, e do lado direito mostra as transformações dos dados que foram aplicadas. Neste caso em particular, esta caixa é formada pela união entre dois conjuntos de dados, amostra-2018-i e 
A caixinha denominada caged por exemplo que tem uma imagem de mesmo nome mostra uma amostra dos dados, a distribuição dos dados, e do lado direito mostra uma sessão denominada recipes com as transformações dos dados que foram aplicadas. Neste caso em particular, esta caixa é formada pela união entre dois conjuntos de dados, amostra-2018-i e amostra-2018-ii. Posteriormente foi aplicado um pivoteamento para contar a quantidade de admissões e demissões existentes. Posteriormente foram renomeadas as colunas pivoteadas.

## Análise de dados<a name="data_analitics"></a>
A análise de dados pode ser vista no arquivo "Estudo_Cimento.ipynb" com as observações do estudo e devidas conclusões.

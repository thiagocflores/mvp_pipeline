# mvp_pipeline
Pipeline de dados feito no Big Query utilizando tecnologia de nuvem da Google Cloud

# MVP – Análise de Interesse em Filmes Populares (2004–2024) com IMDb + Google Trends

## Objetivo

O objetivo deste MVP foi construir um pipeline de dados para analisar a relação entre a popularidade de filmes (medida via Google Trends) e seus dados de avaliação (via IMDb).  
Busquei responder às seguintes perguntas:

- Quais filmes populares entre 2004 e 2024 geraram maior interesse do público?
- Existe correlação entre a nota no IMDb e o interesse por busca?
- Certos gêneros têm picos de interesse em meses específicos?
- Existem filmes muito buscados com nota baixa ou muito bem avaliados com pouco interesse?

---

## Escolha da Plataforma

O projeto originalmente deveria ser desenvolvido na plataforma **Databricks**, porém, por já estar trabalhando com a infraestrutura do **Google Cloud**, optei por utilizar os serviços **BigQuery** e **Google Cloud Storage (GCS)**.

---

## Coleta dos Dados

### IMDb
- Utilizei os dados públicos já disponíveis no BigQuery (`imdb.title_basics` e `imdb.title_ratings`);
- Selecionei os 50 filmes mais votados entre 2004 e 2024, com mais de 250.000 votos.

### Google Trends
- Inicialmente tentei coletar via **Pytrends**, mas encontrei dificuldades devido a **possível bloqueio por parte da Google**;
- Também explorei datasets públicos no BigQuery, mas nenhum cobria os filmes desejados;
- Como alternativa, utilizei uma base de arquivos `.csv` exportados manualmente do Google Trends e armazenados no **GCS**, um por filme.

---

## Modelagem

O modelo de dados foi estruturado de forma **flat**:

- Tabela com metadados dos filmes (título, ano, gênero, nota, votos);
- Tabela com séries temporais de interesse por filme (coluna "Mês" + 50 colunas, uma por filme).

---

## Pipeline (ETL)

1. **Extração**
   - Consulta dos dados do IMDb direto no BigQuery;
   - Leitura dos 50 arquivos CSV do GCS, com tratamento de codificações (`utf-8`, `latin1`, `windows-1252`);

2. **Transformação**
   - Substituição de valores `<1` por `0.5` (Google Trends);
   - Padronização dos títulos dos filmes;
   - Conversão da coluna "Mês" em datetime;
   - Unificação dos dados de trends com os dados do IMDb;

3. **Carga e Exportação**
   - Geração de dois arquivos CSV:
     - `filmes_filtrados.csv`: metadados dos 50 filmes;
     - `trends_filtrados.csv`: evolução mensal do interesse por filme.

---

## Análises e Respostas

- **Quais filmes geraram maior interesse?**  
  Entre os 50 filmes selecionados, *Howl's Moving Castle*, *Ratatouille* e *Eternal Sunshine of the Spotless Mind* foram os 3 com maior média de interesse ao longo do período analisado, nessa ordem.  
  Analisando o interesse por gênero, as maiores médias são de **animação**, **aventura** e **comédia**, nessa ordem.

- **Existe correlação entre nota no IMDb e interesse de busca?**  
  Correlação **negativa fraca** de -0.14 → indica que nota alta nem sempre gera mais interesse, e vice-versa.

- **Há filmes bem avaliados com pouco interesse (ou o contrário)?**  
  Sim. O destaque negativo fica para o filme *The Kashmir Files*, que apesar da nota **8.5** e quase **600 mil votos**, tem uma média de busca de apenas **0.62** (em escala de 0 a 100).  
  Isso se justifica por ser um filme indiano, tendo se tornado um grande sucesso apenas no próprio país.

- **Gêneros têm comportamento sazonal?**  
  Sim. **Animação** e **comédia** têm seu pico em **dezembro e janeiro**, possivelmente impulsionado pelas férias escolares e pelas festas de fim de ano.  
  **Drama** tem pico em **fevereiro**, o que pode estar relacionado à temporada de premiações, especialmente o Oscar.  
  **Ação** tem um leve pico em **julho**, talvez impulsionado pelas férias de inverno.

---

## Limitações

- A coleta dos dados do Google Trends foi limitada a 50 filmes por questões práticas;
- Alguns arquivos exigiram tratamento especial de codificação;
- A análise se concentrou em filmes com grande volume de votos, excluindo títulos menos conhecidos.

---

## Trabalhos Futuros

- Substituir o processo manual de coleta de Trends por uma API customizada;
- Expandir o número de filmes e incluir variáveis como bilheteria ou premiações;
- Armazenar os resultados finais em uma tabela no BigQuery;
- Criar um dashboard interativo no Looker Studio ou Streamlit.

---

## Autoavaliação

Apesar de ter enfrentado muitas dificuldades no processo, por ser um jornalista tendo o primeiro contato com a Engenharia de Dados, acredito que o resultado final foi bastante satisfatório.  
Hoje já tenho um domínio bem maior de **Python**, **SQL** e da **BigQuery**. Reconheço que ainda sofro bastante pra fazer os códigos rodarem, mas com auxílio do Google, consegui resolver ou adaptar todos os problemas que tive ao longo do processo.

Em relação à resposta das perguntas iniciais, acredito que o trabalho teria uma precisão maior caso eu tivesse conseguido utilizar a API do Google Trends, o que não foi possível. De qualquer forma, consegui tirar ótimas conclusões através da análise dos dados disponíveis.

---

## Repositório

O código completo do pipeline, arquivos de entrada e saída, e notebooks com as análises estão disponíveis neste repositório:  
**[https://github.com/thiagocflores/mvp_pipeline](https://github.com/thiagocflores/mvp_pipeline)**

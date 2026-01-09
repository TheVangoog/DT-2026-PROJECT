ELT proces datasetu Food Trends v Snowflake

Tento repozitár obsahuje implementáciu ELT procesu v Snowflake a návrh dátového skladu (DWH) so schémou Star Schema nad datasetom:

Food Analytics and Food Trends Data in Web Recipes (Snowflake Marketplace)

Projekt sa zameriava na analýzu popularity potravín a ich kombinácií v receptoch počas prvého polroka 2024.

Obsah

Úvod a popis zdrojových dát

Návrh dimenzionálneho modelu

ELT proces v Snowflake

Vizualizácia dát

Záver

1. Úvod a popis zdrojových dát
Účel analýzy

Cieľom projektu je analyzovať:

trendy v používaní jednotlivých potravín,

najčastejšie kombinácie ingrediencií,

zmeny popularity v čase,

najpopulárnejšie potraviny podľa kategórií.

Biznis využitie

Dáta podporujú tieto procesy:

Food Industry Analytics – identifikácia trendujúcich potravín

Recipe Optimization – objavovanie populárnych kombinácií

Market Research – sezónne trendy v stravovaní

Menu Planning – strategické plánovanie ponuky

Zdrojové dáta

Database: FOOD_ANALYTICS_AND_FOOD_TRENDS_DATA_IN_WEB_RECIPES

Schema: PUBLIC

Zdrojové tabuľky
Tabuľka	Popis	Význam
FOODS	Katalóg potravín	Master zoznam potravín
TOP_FINAL	Mesačná popularita potravín	Trendy jednotlivých potravín
TOP_COOC_FINAL	Kombinácie potravín	Trendy food pairingov
ERD diagram zdrojových dát

2. Návrh dimenzionálneho modelu
Star Schema

Dimenzie
DIM_FOOD (SCD Type 1)

FOOD_KEY (PK)

FOODID (business key)

LABEL

CATEGORY

SR, URI

Zmeny sa prepisujú bez uchovávania histórie.
Vzťah: 1:N k faktovej tabuľke.

DIM_COOC_FOOD (SCD Type 1)

COOC_FOOD_KEY (PK)

COOC_FOOD_ID (business key)

COOC_LABEL

COOC_CATEGORY

Zmeny sa prepisujú bez histórie.
Vzťah: 1:N k faktovej tabuľke.

DIM_DATE (SCD Type 0)

DATE_KEY (PK)

YEAR

MONTH

QUARTER

MONTH_CODE

MONTH_NAME

Statická dimenzia.
Vzťah: 1:N k faktovej tabuľke.

DIM_TREND_TYPE (SCD Type 0)

TREND_TYPE_KEY (PK)

TREND_TYPE

DESCRIPTION

Statický číselník.
Vzťah: 1:N k faktovej tabuľke.

Faktová tabuľka FACT_FOOD_TRENDS

Primárny kľúč:

TREND_KEY

Cudzie kľúče:

FOOD_KEY → DIM_FOOD

COOC_FOOD_KEY → DIM_COOC_FOOD

DATE_KEY → DIM_DATE

TREND_TYPE_KEY → DIM_TREND_TYPE

Metriky:

POPULARITY_VALUE

SCORE

TOP_ORDER

RANK_IN_MONTH

PREV_POPULARITY

POPULARITY_CHANGE_PCT

CUMULATIVE_POPULARITY

Použité window functions:

ROW_NUMBER() – ranking v rámci mesiaca

LAG() – hodnota z predchádzajúceho mesiaca

SUM() OVER() – kumulatívna popularita

3. ELT proces v Snowflake
3.1 Extract

Dáta sú extrahované zo Snowflake Marketplace do staging schémy.

CREATE OR REPLACE TABLE STG_FOODS AS
SELECT * FROM FOOD_ANALYTICS_AND_FOOD_TRENDS_DATA_IN_WEB_RECIPES.PUBLIC.FOODS;

CREATE OR REPLACE TABLE STG_TOP_FINAL AS
SELECT * FROM FOOD_ANALYTICS_AND_FOOD_TRENDS_DATA_IN_WEB_RECIPES.PUBLIC.TOP_FINAL;

CREATE OR REPLACE TABLE STG_TOP_COOC_FINAL AS
SELECT * FROM FOOD_ANALYTICS_AND_FOOD_TRENDS_DATA_IN_WEB_RECIPES.PUBLIC.TOP_COOC_FINAL;

3.2 Transform

deduplikácia pomocou DISTINCT

čistenie dát pomocou COALESCE

transformácia M1–M6 na riadky (unpivot)

výpočet analytických metrík pomocou window functions

3.3 Load

vytvorenie dimenzií

vytvorenie faktovej tabuľky

výpočet rankingov, percentuálnych zmien a kumulatívnych súčtov

4. Vizualizácia dát

Dashboard obsahuje 5 vizualizácií:

Top produkty podľa popularity

Popularita podľa kategórií

Top 10 kombinácií potravín

Heatmapa popularity v čase

Distribúcia popularity kombinácií

Každá vizualizácia je vytvorená pomocou SQL dotazu nad DWH.

5. Záver

Projekt demonštruje kompletný ELT proces v Snowflake, návrh dimenzionálneho modelu a praktické využitie window functions pre analytické účely. Výsledný model umožňuje efektívnu analýzu trendov v oblasti potravín a receptov.

Autor

Ivan Timoshkin
# Escolha da base para o trabalho Spark ML (Classificação, Windows local)

## Contexto
Trabalho individual de Processamento de Dados Massivos (IESB). Tipo sorteado: **classificação**.
Métricas exigidas: AUC, F1, matriz de confusão, com atenção ao desbalanceamento.
Exigências da base: pública, >= 200 mil registros, fonte com link reproduzível, justificativa de dados massivos.
O notebook deve baixar a base da fonte original, sem subir dados ao GitHub.
Ambiente: Spark local no Windows, sem cluster.

## Recomendação principal: US DOT Flight Delays 2015 (Kaggle `usdot/flight-delays`)
- Tamanho: ~5,8 mi de voos, 31 colunas, ~565 MB de CSV em `flights.csv`. Roda em Spark local com `spark.driver.memory` de 4-8 GB.
- Alvo: `ATRASADO = ARRIVAL_DELAY >= 15` (definição oficial de atraso). Cerca de 18-20% de atrasados, desbalanceamento moderado.
- Pergunta de negócio: dado o que se sabe antes da decolagem, o voo vai chegar atrasado?
- Dados massivos: ~5,8 mi de linhas e relações de alta cardinalidade (companhia x origem x destino x horário), o que justifica particionamento e Pipeline distribuído.
- Engenharia de atributos (2 pts): StringIndexer/OneHotEncoder para companhia, origem e destino; hora cíclica (sin/cos); dia da semana; distância; feriados; VectorAssembler + StandardScaler dentro de um Pipeline.
- Armadilha de vazamento: excluir `ARRIVAL_DELAY`, `ARRIVAL_TIME`, `ELAPSED_TIME`, `AIR_TIME`, `WHEELS_ON`, `TAXI_IN`, `DIVERTED`, `CANCELLED` e as colunas `*_DELAY` de causa. Só usar informação conhecida na decolagem. Isso rende ponto no critério "ausência de vazamento".
- Modelos: LogisticRegression (referência) vs RandomForest/GBTClassifier, com CrossValidator ou TrainValidationSplit para hiperparâmetros. Usar `weightCol` para o desbalanceamento.
- Baseline de comparação: classificar tudo como "não atrasado" (acurácia ~80%, AUC 0,5).
- Divisão treino/teste: preferencialmente temporal (treino jan-set, teste out-dez), o que evita vazamento temporal.
- Download: Kaggle exige conta e token (`kagglehub` ou `kaggle datasets download -d usdot/flight-delays`). No notebook, ler `KAGGLE_USERNAME` e `KAGGLE_KEY` de variáveis de ambiente, nunca do repositório. Ao enviar ao professor, avisar que é preciso o `kaggle.json`.
- Obs.: o Kaggle pode ter mudado o formato do arquivo. Confirmar o nome e o schema ao baixar. O notebook deve definir o StructType à mão.

## Alternativas
1. **NYC Yellow Taxi (NYC TLC, parquet)**: sem login, URL direta (`https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_AAAA-MM.parquet`, ~3 mi de linhas por mês). Alvo possível: gorjeta >= 20%. Cuidado: só cartão registra gorjeta, o que enviesa o alvo. É a melhor opção se o professor não aceitar token do Kaggle.
2. **PaySim (Kaggle `ealaxi/paysim1`)**: ~6,3 mi de linhas, fraude (~0,13%). Atende bem à "atenção ao desbalanceamento", mas é sintética, as colunas de saldo vazam o alvo e os modelos chegam a AUC quase 1, o que dá pouca discussão de limitações.
3. **Home Credit Default Risk**: classificação com muitas categóricas, mas exige aceitar as regras da competição e juntar várias tabelas.

## Próximos passos (depois da escolha)
1. Confirmar a base (voos 2015 ou táxi NYC).
2. Criar `trabalho_sparkml.ipynb` seguindo as 8 seções do PDF: contextualização, ingestão com StructType, exploração e limpeza, features + Pipeline, modelagem, avaliação, `model.save()`, conclusão.
3. Verificar Java e PySpark instalados (`java -version`, `pip show pyspark`) e `HADOOP_HOME`/winutils no Windows, que costuma dar erro no save.
4. Criar `.gitignore` para dados e modelos, repositório no GitHub e envio do link até 3/11.

## Verificação
- Rodar o notebook do início ao fim em kernel limpo (Restart & Run All) com download automático.
- Conferir `df.count() >= 200000` e a lista de colunas sem vazamento.
- Conferir AUC, F1, matriz de confusão no teste e a existência do diretório salvo por `model.save()`, recarregado com `PipelineModel.load()`.

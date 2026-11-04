# Previsão de atrasos de voos com Spark ML

Trabalho de **Processamento de Dados Massivos** (IESB): classificação de atrasos de voos com PySpark e Spark ML sobre a base **US DOT Flight Delays 2015** (~5,8 milhões de voos).

- Notebook (com as saídas gravadas): [trabalho_sparkml.ipynb](trabalho_sparkml.ipynb)
- Fonte dos dados: <https://www.kaggle.com/datasets/usdot/flight-delays>

Os dados **não** estão neste repositório: o notebook os baixa do Kaggle com o `kagglehub`.

## Pergunta de negócio
Com as informações disponíveis, um voo vai chegar com **15 minutos ou mais de atraso**? Dois cenários:
- **A, antes da decolagem:** só informação de planejamento (companhia, rota, horário, dia da semana, feriados, distância).
- **B, na saída do portão:** o cenário A mais o atraso na partida (`DEPARTURE_DELAY`).

## O que o notebook faz
1. Contextualização e justificativa de dados massivos.
2. Ingestão com `StructType` manual e conversão para Parquet (592 MB para 142 MB).
3. Exploração e limpeza em Spark, incluindo a recuperação dos códigos IATA de outubro (que vinham como IDs numéricos) por votação nos próprios dados. Base limpa: 5.713.313 voos.
4. Prevenção de vazamento, **divisão temporal** (treino jan-set, teste out-dez) e engenharia de atributos em um `Pipeline` (`SQLTransformer`, `StringIndexer`, `OneHotEncoder`, `StandardScaler`, `VectorAssembler`).
5. Regressão logística e Random Forest, com pesos de classe, ajuste de hiperparâmetros em validação temporal e limiar escolhido na validação.
6. Avaliação no teste: AUC-ROC, AUC-PR, F1, matriz de confusão, importância de atributos e limitações.
7. Persistência do `PipelineModel` com `save()`.
8. Conclusão.

## Resultados no teste (outubro a dezembro, 1.414.267 voos)

| Cenário | Modelo | AUC-ROC | AUC-PR | Precisão | Recall | F1 (atrasado) |
|---|---|---|---|---|---|---|
| Referência (sempre pontual) | | 0,500 | 0,161 | 0 | 0 | 0 |
| A, antes da decolagem | Regressão logística | 0,606 | 0,216 | 0,209 | 0,590 | 0,308 |
| A | Random Forest | 0,612 | 0,221 | 0,207 | 0,607 | 0,309 |
| B, na saída do portão | Regressão logística | **0,928** | **0,852** | 0,842 | 0,730 | **0,782** |
| B | Random Forest | 0,909 | 0,797 | 0,766 | 0,772 | 0,769 |

Antes da decolagem, a informação de planejamento ordena o risco de atraso com ganho modesto sobre o acaso. Na saída do portão, o atraso na partida torna o problema bem previsível. As limitações e melhorias estão nas seções 6 e 8 do notebook.

## Como executar

Requisitos: Python 3.12 e Java 17 (JDK).

```powershell
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
```

No VS Code, abra o notebook, escolha o kernel do `.venv` e use *Run All* (cerca de 25 minutos num PC com 16 GB de RAM).

### Credenciais do Kaggle
Gere um token em *kaggle.com > Settings > API > Create New Token* e defina variáveis de ambiente (nunca coloque a chave no repositório):

```powershell
setx KAGGLE_USERNAME "seu_usuario"
setx KAGGLE_KEY "sua_chave"
```
Reabra o terminal/VS Code depois. Também é aceito `KAGGLE_API_TOKEN` (formato novo `KGAT_...`).

### Windows: winutils
Para gravar arquivos (Parquet e `model.save()`), o Spark no Windows precisa do `winutils.exe` e `hadoop.dll` do Hadoop 3.3.x em `HADOOP_HOME\bin`. O notebook procura em `%HADOOP_HOME%` ou em `C:\Users\<usuario>\hadoop`. Em Linux/macOS isso não é necessário.

### Memória
O notebook configura `spark.driver.memory` de 6 GB. Feche programas pesados antes de executar o treino.

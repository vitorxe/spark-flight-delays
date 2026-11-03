# Previsão de atrasos de voos com Spark ML

Trabalho de **Processamento de Dados Massivos** (IESB): classificação de atrasos de voos com PySpark e Spark ML sobre a base **US DOT Flight Delays 2015** (~5,8 milhões de voos).

- Notebook: [trabalho_sparkml.ipynb](trabalho_sparkml.ipynb)
- Fonte dos dados: <https://www.kaggle.com/datasets/usdot/flight-delays>

Os dados **não** estão neste repositório: o notebook os baixa do Kaggle com o `kagglehub`.

## Como executar

Requisitos: Python 3.12 e Java 17 (JDK).

```powershell
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
```

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
O notebook configura `spark.driver.memory` de 6 GB. Feche programas pesados antes de treinar os modelos.

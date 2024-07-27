# Análise de Séries Temporais com ARIMA

Este projeto realiza a análise de séries temporais de dados de energia usando o modelo ARIMA. A análise é feita a partir de um conjunto de dados disponível em um arquivo Excel, e o código inclui a remoção de outliers e a previsão de séries temporais.

## Descrição do Projeto

O objetivo deste projeto é analisar dados de diferentes tipos de geração de energia e carga, utilizando o modelo ARIMA para prever valores futuros. O projeto realiza as seguintes etapas:

1. **Carregamento dos Dados**: Leitura dos dados a partir de um arquivo Excel.
2. **Filtragem dos Dados**: Seleção dos dados referentes ao subsistema 'NORDESTE'.
3. **Pré-processamento dos Dados**: Conversão da coluna de data para o tipo datetime e definição do índice do DataFrame como a data.
4. **Remoção de Outliers**: Aplicação do método IQR (Interquartile Range) para remover outliers das séries temporais.
5. **Ajuste do Modelo ARIMA**: Configuração e ajuste do modelo ARIMA para prever os valores das séries temporais.
6. **Visualização dos Resultados**: Geração de gráficos para comparar as séries temporais sem outliers com as previsões do modelo ARIMA.

## Requisitos

Certifique-se de ter as seguintes bibliotecas instaladas:

- `pandas`
- `matplotlib`
- `seaborn`
- `statsmodels`
- `openpyxl`

Você pode instalar as dependências necessárias usando `pip`:

```bash
pip install pandas matplotlib seaborn statsmodels openpyxl
```

## Como Usar

1. **Carregar o Arquivo de Dados**: Substitua o caminho do arquivo `file_path` pela localização do seu arquivo Excel.

2. **Executar o Código**: Execute o código Python para carregar os dados, processá-los e gerar gráficos.

## Código

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from statsmodels.tsa.arima.model import ARIMA

# Função para remover outliers usando o método IQR
def remove_outliers_iqr(df, column):
    Q1 = df[column].quantile(0.25)
    Q3 = df[column].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    return df[(df[column] >= lower_bound) & (df[column] <= upper_bound)].copy()

# Carregar o arquivo Excel
file_path = '/kaggle/input/balanco-energia-subsistema-2023/BALANCO_ENERGIA_SUBSISTEMA_2023.xlsx'

# Ler o arquivo Excel e criar um DataFrame
df = pd.read_excel(file_path, engine='openpyxl')

# Filtrar o DataFrame para manter apenas as linhas onde nom_subsistema é 'NORDESTE'
df_nordeste = df[df['nom_subsistema'] == 'NORDESTE'].copy()

# Converter a coluna 'din_instante' para datetime
df_nordeste['din_instante'] = pd.to_datetime(df_nordeste['din_instante'])

# Definir a coluna 'din_instante' como o índice
df_nordeste.set_index('din_instante', inplace=True)

# Garantir que o índice tenha uma frequência definida (por exemplo, hourly)
df_nordeste = df_nordeste.asfreq('H')

# Definir o estilo dos gráficos
sns.set(style="whitegrid")

# Criar uma figura com múltiplos subplots
plt.figure(figsize=(14, 10))

# Definir o período para a média móvel
window_size = 7  # Você pode ajustar este valor conforme necessário

# Plotar cada série temporal em um subplot separado
for i, column in enumerate(['val_gerhidraulica', 'val_gertermica', 'val_gereolica', 'val_gersolar', 'val_carga', 'val_intercambio']):
    plt.subplot(3, 2, i + 1)
    
    # Remover outliers e fazer uma cópia explícita
    df_filtered = remove_outliers_iqr(df_nordeste, column)
    
    # Ajustar o modelo ARIMA
    try:
        # Configurar e ajustar o modelo ARIMA
        model = ARIMA(df_filtered[column].dropna(), order=(5, 1, 0))  # Ajuste os parâmetros (p, d, q) conforme necessário
        model_fit = model.fit()
        
        # Prever a série temporal
        df_filtered['arima_prediction'] = model_fit.predict(start=0, end=len(df_filtered)-1, dynamic=False)
        
        # Plotar a série temporal sem outliers e a previsão ARIMA
        plt.plot(df_filtered.index, df_filtered[column], label=f'{column} (sem outliers)', alpha=0.7)
        plt.plot(df_filtered.index, df_filtered['arima_prediction'], label=f'{column} (ARIMA)', color='red', linestyle='--')
        plt.title(column)
        plt.xlabel('Data')
        plt.ylabel('Valor')
        plt.legend()
        
    except Exception as e:
        print(f"Erro ao ajustar ARIMA para {column}: {e}")

# Ajustar o layout e exibir os gráficos
plt.tight_layout()
plt.show()
```

## Observações

- **Frequência do Índice**: Certifique-se de que o índice do DataFrame tenha uma frequência definida para evitar problemas durante a previsão com ARIMA.
- **Parâmetros do ARIMA**: Ajuste os parâmetros `(p, d, q)` do modelo ARIMA conforme necessário para obter os melhores resultados.

## Licença

Este projeto está licenciado sob a [Licença MIT](https://opensource.org/licenses/MIT). Veja o arquivo LICENSE para mais detalhes.

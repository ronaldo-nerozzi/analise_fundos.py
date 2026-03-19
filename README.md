# ============================================================
# Análise Exploratória de Fundos de Investimento — CVM
# Autor: Ronaldo Nerozzi
# GitHub: github.com/ronaldo-nerozzi
# Descrição: Análise exploratória com Pandas e visualizações
# ============================================================

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Configurações visuais
sns.set_theme(style="darkgrid")
plt.rcParams["figure.figsize"] = (12, 6)
plt.rcParams["font.size"] = 12

# ============================================================
# 1. CARREGAMENTO DOS DADOS
# Fonte: Dados públicos CVM
# URL: https://dados.cvm.gov.br/dados/FI/DOC/INF_DIARIO/DADOS/
# ============================================================

# Simulação de dados para demonstração
# Em produção, substitua por: df = pd.read_csv("inf_diario_fi_AAAAMM.csv", sep=";")
dados = {
    "CNPJ_FUNDO":    ["11.111.111/0001-11"] * 6 + ["22.222.222/0001-22"] * 6,
    "DT_COMPTC":     pd.date_range("2024-01-01", periods=6, freq="MS").tolist() * 2,
    "VL_TOTAL":      [1_200_000, 1_350_000, 1_100_000, 1_500_000, 1_450_000, 1_600_000,
                      800_000,   850_000,   900_000,   870_000,   920_000,   980_000],
    "VL_PATRIM_LIQ": [1_000_000, 1_150_000,   950_000, 1_300_000, 1_250_000, 1_400_000,
                        700_000,   740_000,   780_000,   760_000,   800_000,   850_000],
    "NR_COTST":      [120, 135, 110, 150, 145, 160,
                       80,  85,  90,  87,  92,  98],
    "VL_CAPTC_LIQ":  [50_000, 80_000, -30_000, 120_000, 90_000, 110_000,
                       20_000, 30_000,  15_000,  -5_000, 25_000,  40_000],
    "NOME_FUNDO":    ["Fundo Alpha FI"] * 6 + ["Fundo Beta FI"] * 6,
}

df = pd.DataFrame(dados)
df["DT_COMPTC"] = pd.to_datetime(df["DT_COMPTC"])

print("=" * 60)
print("ANÁLISE EXPLORATÓRIA — FUNDOS DE INVESTIMENTO")
print("=" * 60)

# ============================================================
# 2. VISÃO GERAL DOS DADOS
# ============================================================
print("\n📋 Primeiras linhas do dataset:")
print(df.head())

print("\n📊 Informações gerais:")
print(df.info())

print("\n📈 Estatísticas descritivas:")
print(df[["VL_TOTAL", "VL_PATRIM_LIQ", "NR_COTST", "VL_CAPTC_LIQ"]].describe().round(2))

# ============================================================
# 3. PATRIMÔNIO LÍQUIDO — Comparativo entre fundos
# ============================================================
print("\n💰 Patrimônio Líquido médio por fundo:")
patrim_medio = (
    df.groupby("NOME_FUNDO")["VL_PATRIM_LIQ"]
    .mean()
    .reset_index()
    .rename(columns={"VL_PATRIM_LIQ": "Patrimônio Médio (R$)"})
    .sort_values("Patrimônio Médio (R$)", ascending=False)
)
print(patrim_medio.to_string(index=False))

# ============================================================
# 4. EVOLUÇÃO DO PATRIMÔNIO — Gráfico de linha
# ============================================================
fig, ax = plt.subplots()
for nome, grupo in df.groupby("NOME_FUNDO"):
    ax.plot(grupo["DT_COMPTC"], grupo["VL_PATRIM_LIQ"] / 1_000_000,
            marker="o", label=nome)

ax.set_title("Evolução do Patrimônio Líquido (R$ milhões)", fontsize=14, fontweight="bold")
ax.set_xlabel("Mês")
ax.set_ylabel("Patrimônio Líquido (R$ mi)")
ax.legend()
plt.tight_layout()
plt.savefig("grafico_patrimonio.png", dpi=150)
plt.show()
print("\n✅ Gráfico salvo: grafico_patrimonio.png")

# ============================================================
# 5. CAPTAÇÃO LÍQUIDA — Barras por mês
# ============================================================
captacao = df.groupby(["DT_COMPTC", "NOME_FUNDO"])["VL_CAPTC_LIQ"].sum().reset_index()

fig, ax = plt.subplots()
fundos = captacao["NOME_FUNDO"].unique()
x = range(len(captacao["DT_COMPTC"].unique()))
datas = captacao["DT_COMPTC"].unique()
largura = 0.35

for i, fundo in enumerate(fundos):
    valores = captacao[captacao["NOME_FUNDO"] == fundo]["VL_CAPTC_LIQ"].values / 1_000
    offset = [xi + i * largura for xi in x]
    bars = ax.bar(offset, valores, largura, label=fundo)

ax.set_title("Captação Líquida Mensal (R$ mil)", fontsize=14, fontweight="bold")
ax.set_xlabel("Mês")
ax.set_ylabel("Captação Líquida (R$ mil)")
ax.set_xticks([xi + largura / 2 for xi in x])
ax.set_xticklabels([d.strftime("%b/%Y") for d in datas], rotation=45)
ax.legend()
ax.axhline(0, color="black", linewidth=0.8, linestyle="--")
plt.tight_layout()
plt.savefig("grafico_captacao.png", dpi=150)
plt.show()
print("✅ Gráfico salvo: grafico_captacao.png")

# ============================================================
# 6. NÚMERO DE COTISTAS — Evolução
# ============================================================
print("\n👥 Número de cotistas — último período:")
ultimo_periodo = df[df["DT_COMPTC"] == df["DT_COMPTC"].max()]
print(ultimo_periodo[["NOME_FUNDO", "NR_COTST"]].to_string(index=False))

# ============================================================
# 7. CORRELAÇÃO entre variáveis numéricas
# ============================================================
fig, ax = plt.subplots(figsize=(8, 6))
corr = df[["VL_TOTAL", "VL_PATRIM_LIQ", "NR_COTST", "VL_CAPTC_LIQ"]].corr()
sns.heatmap(corr, annot=True, fmt=".2f", cmap="Blues", ax=ax)
ax.set_title("Mapa de Correlação — Variáveis dos Fundos", fontsize=13, fontweight="bold")
plt.tight_layout()
plt.savefig("grafico_correlacao.png", dpi=150)
plt.show()
print("✅ Gráfico salvo: grafico_correlacao.png")

print("\n" + "=" * 60)
print("✅ Análise concluída com sucesso!")
print("=" * 60)
# analise_fundos.py
Análise de fundos python 

# Detecção de Fraudes em Cartão de Crédito com MLOps

**Links do Projeto:**
* [Repositório de Dados e Código no DagsHub](https://dagshub.com/fernando-marconi/credit-card-fraud)
* [Painel de Rastreamento de Experimentos no MLflow](https://dagshub.com/fernando-marconi/credit-card-fraud.mlflow)

Este projeto simula um cenário real de detecção de fraudes bancárias. O objetivo central é equilibrar a segurança financeira (bloqueio de fraudes) com a experiência do usuário (redução de falsos positivos). Apliquei princípios de MLOps para garantir a reprodutibilidade e a governança de cada etapa do desenvolvimento.

---

## Destaques Técnicos

Diferente de abordagens estritamente acadêmicas, este repositório utiliza ferramentas de nível empresarial para gerenciar o ciclo de vida do modelo:

* **DVC (Data Version Control):** Utilizado para o versionamento do dataset `creditcard.csv`, mantendo o arquivo pesado fora do Git e armazenado de forma segura no storage do DagsHub.
* **MLflow:** Empregado para o rastreamento sistemático de cada experimento, permitindo a comparação de métricas e hiperparâmetros entre diferentes modelos.
* **Engenharia de Features:** Aplicação de tratamento estatístico, incluindo escalonamento robusto e limpeza de anomalias.

---

## 1. O Problema de Negócio

O conjunto de dados contém transações de cartões de crédito europeus onde as fraudes representam apenas **0,17%** do total.

Neste contexto, a acurácia é uma métrica enganosa. Se o modelo classificar todas as transações como legítimas, ele atingirá 99,8% de acurácia, mas falhará em seu propósito de negócio. O foco estratégico deste projeto foi otimizar o **Recall** para minimizar o prejuízo financeiro, monitorando a **Precisão** para evitar o atrito com clientes legítimos.

---

## 2. Pipeline de Desenvolvimento

### Tratamento e Limpeza de Dados
* **RobustScaler:** Utilizado para normalizar as variáveis `Time` e `Amount`. Este método é resistente a outliers agressivos:
  $$x_{scaled} = \frac{x_i - Q_2(x)}{Q_3(x) - Q_1(x)}$$
* **Análise de Correlação:** Através de um conjunto de dados balanceado, identificamos que as variáveis **V14, V12 e V10** possuem forte correlação negativa com a classe de fraude.
* **Remoção de Outliers (IQR):** Aplicamos o método do Intervalo Interquartil para remover ruídos extremos nas features de maior importância:
  $$IQR = Q_3 - Q_1$$
  $$Limite\_Inferior = Q_1 - (1.5 \times IQR)$$

### Experimentação com Balanceamento
Foram testadas duas abordagens principais, documentadas no MLflow:
1. **Under-sampling:** Elevado Recall (92%), porém gerou mais de 2.000 falsos positivos, o que inviabilizaria a operação de suporte do banco.
2. **SMOTE (Over-sampling):** Criação de dados sintéticos da classe minoritária, reduzindo drasticamente os alarmes falsos para apenas 14, com um Recall inicial de 82%.

---

## 3. Otimização e Resultados Finais

O diferencial técnico deste projeto foi o ajuste do **Limiar de Decisão (Threshold)**. Ao reduzir o limiar padrão de 0.5 para **0.3**, alcançamos o ponto ideal de equilíbrio para o negócio:

| Métrica | Resultado Final (Threshold 0.3) |
| :--- | :--- |
| **Recall (Classe 1)** | **89%** |
| **Falsos Positivos** | **35** |
| **Precisão (Classe 1)** | **71%** |

**Conclusão:** O modelo final captura 89% das fraudes reais, gerando apenas 35 bloqueios indevidos em um universo de mais de 56.000 transações de teste.

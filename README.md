# Modelo Agregado de Pequeno Porte – BCB

Ferramenta interativa para projeção de IPCA baseada em **Funções de Resposta ao Impulso (FRIs)** do Banco Central do Brasil.

Desenvolvido com base na metodologia do **Relatório de Inflação (RI) de junho de 2024** do BCB, este modelo permite simular impactos de mudanças na Selic, câmbio, petróleo e commodities sobre a inflação.

---

## 📊 Metodologia de Cálculo

### Estrutura Base

O modelo projeta o IPCA para **8 trimestres** (2T26 a 1T28) a partir de uma **trajetória de Selic observada**:

```
Trimestres:  2T26  3T26  4T26  1T27  2T27  3T27  4T27  1T28
Projeção:     3,6%  3,7%  3,8%  3,9%  3,4%  3,3%  3,3%  3,2%
```

**Horizonte (T+N)**: Quantos trimestres até o horizonte de interesse
- T+6 = 3T27 (última reunião Copom abr/26)
- T+7 = 4T27 (próxima reunião abr/26)

### Equação Principal

```
IPCA_projetado(T+N) = IPCA_base(T+N) + Σ Impactos
```

Onde:

$$\text{IPCA} = \text{Base BCB} + \text{Imp}_\text{Selic} + \text{Imp}_\text{Câmbio} + \text{Imp}_\text{Brent} + \text{Imp}_\text{Commodities} + \text{Imp}_{Focus2026} + \text{Imp}_{Focus2027}$$

---

## 🔧 Funções de Resposta ao Impulso (FRIs)

As FRIs quantificam o impacto acumulado **em 4 trimestres** de cada choque sobre a inflação (em p.p.).

### 1️⃣ **Selic (Gráfico 1b do RI Jun/24)**

Resposta da inflação a um **choque de 100 bps** na Selic

| T+0 | T+1 | T+2 | T+3 | T+4 | T+5 | T+6 | T+7 | T+8 | T+9 | T+10 |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|------|
| 0,0 | -0,10 | -0,15 | -0,22 | -0,27 | -0,29 | -0,31 | -0,30 | -0,28 | -0,24 | -0,20 |

**Interpretação**: Um aumento de 100 bps na Selic reduz o IPCA em até -0,31 p.p. (pico em T+6)

**Cálculo do impacto**:
```
Imp_Selic = Σ(Δ Selic em impulsos) × FRI_selic(T+N-t)
```

---

### 2️⃣ **Câmbio (Gráfico 3b do RI Jun/24)**

Resposta ao **choque de 10% de depreciação cambial** (R$ enfraquecido)

| T+0 | T+1 | T+2 | T+3 | T+4 | T+5 | T+6 | T+7 | T+8 | T+9 | T+10 |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|------|
| 0,0 | 0,20 | 0,35 | 0,58 | **0,90** | 0,80 | 0,67 | 0,40 | 0,31 | 0,28 | 0,24 |

**Interpretação**: Uma depreciação de 10% do R$ aumenta o IPCA em até **+0,90 p.p.** (pico em T+4)

**Cálculo do impacto**:
```
Var_Cambio(%) = (Taxa_nova - Taxa_base) / Taxa_base × 100
Imp_Câmbio = (Var_Cambio / 10) × FRI_cambio(T+N)
```

---

### 3️⃣ **Brent USD (RPM Jun/25)**

Resposta ao **choque de 10% de apreciação do petróleo** (em USD)

| T+0 | T+1 | T+2 | T+3 | T+4 | T+5 | T+6 | T+7 | T+8 | T+9 | T+10 |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|------|
| 0,0 | 0,10 | 0,30 | 0,40 | 0,50 | **0,40** | 0,30 | 0,20 | 0,20 | 0,19 | 0,19 |

**Interpretação**: Alta de 10% no Brent aumenta o IPCA em até +0,50 p.p. (pico em T+4)

**Cálculo do impacto**:
```
Var_Brent(%) = (Preço_novo - Preço_base) / Preço_base × 100
Imp_Brent = (Var_Brent / 10) × FRI_brent(T+N)
```

---

### 4️⃣ **Índice de Commodities Agrícolas (Gráfico 8 do RI Jun/24)**

Resposta ao **choque de 10%** em preços de commodities

| T+0 | T+1 | T+2 | T+3 | T+4 | T+5 | T+6 | T+7 | T+8 | T+9 | T+10 |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|------|
| 0,0 | 0,25 | 0,50 | 0,70 | **0,90** | 0,65 | 0,50 | 0,40 | 0,35 | 0,30 | 0,25 |

**Cálculo**:
```
Var_Commodities(%) = (Índice_novo - Índice_base) / Índice_base × 100
Imp_Commodities = (Var_Commodity / 10) × FRI_comm(T+N)
```

---

### 5️⃣ **Focus 2026 e 2027 (Pesos Fixos)**

Impacto direto das projeções de inflação do mercado (Focus - Banco Central)

**Metodologia**:
```
Imp_Focus2026 = (Projeção_2026 - Base_2026) × 0,30  (peso 30%)
Imp_Focus2027 = (Projeção_2027 - Base_2027) × 0,40  (peso 40%)
```

---

## 📈 Componentes do Modelo

### Entradas Principais

| Campo | Descrição | Padrão |
|-------|-----------|--------|
| **s0** | Selic atual (%) | 14,75% |
| **cx0** | Câmbio P.O. ($R) | 5,40 |
| **cx1** | Câmbio esperado ($R) | 5,25 |
| **br0** | Brent P.O. (US$) | 75 |
| **br1** | Brent esperado (US$) | 75 |
| **cm0** | IC-Br P.O. (índice) | 83 |
| **cm1** | IC-Br esperado | 83 |
| **f26_0** | Focus IPCA 2026 base | 4,10% |
| **f26_1** | Focus IPCA 2026 novo | 4,31% |
| **f27_0** | Focus IPCA 2027 base | 3,80% |
| **f27_1** | Focus IPCA 2027 novo | 3,84% |
| **proj_cop** | Projeção Copom (277ª) | 3,30% |

### Saídas

- **IPCA Projetado**: Para cada horizonte (T+6 até T+16)
- **Decomposição de Impactos**: Contribuição individual de cada fator
- **Selic Necessária**: Qual regime manteria o IPCA na meta (3%)
- **Tabela Trimestral**: Trajetória completa com bases e cenários

---

## 🎮 Como Usar

### Modelo Completo
1. Insira valores atuais de câmbio, taxas de juros e commodities
2. Defina expectativas futuras para cada variável
3. Escolha a **próxima reunião** do Copom (define horizonte)
4. Clique em **"🔄 Atualizar dados do BCB"** para buscar dados em tempo real
5. Veja a projeção atualizar automaticamente

### Modelo Suavizado
- Mesmo modelo, mas com **FRI Brent reduzida** (×0,5)
- Reflete menor pass-through de petróleo

### Cenários Selic
- Teste diferentes trajetórias de Selic
- Simule cortes ou altas mantendo cambio constante

---

## 🌐 Integração com APIs do BCB

O botão **"🔄 Atualizar dados do BCB"** busca em tempo real:

| API | Dados | Campo |
|-----|-------|-------|
| **PTAX** | Câmbio dólar | `cx1` |
| **ExpectativasMercadoSelic** | Selic mediana | `s0` |
| **ExpectativasMercadoAnuais** | Focus 2026/2027 | `f26_1`, `f27_1` |

---

## 📚 Referências

- **Relatório de Inflação (RI), junho de 2024** – Banco Central do Brasil
- Cálculos baseados em **Funções de Resposta ao Impulso** (impulse response functions)
- Horizontes de política monetária conforme RPM (Reunião de Política Monetária)

---

## 💡 Notas Técnicas

### Meta de Inflação
- Meta: **3,0% a.a.**
- Intervalo: 1,5% a 4,5%

### Períodos Considerados
- Base: janeiro de 2026 a dezembro de 2027
- 16 períodos mensais (jan/26 ~ dez/27)

### Convenção Trimestral
- Trimestre = 3 meses (T-1 = jan-fev-mar, etc)
- T+N = N trimestres à frente

---

**Versão**: 1.0 | **Atualizado**: Abril 2026
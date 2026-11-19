# 📊 Data Sources & Engineering Benchmarks: Predictive Cost & Schedule Risk

This repository predicts cost overruns, schedule delays, and contractor failure risks across heavy civil infrastructure megaprojects. The analytical models are calibrated against audited historical infrastructure portfolios and macroeconomic indices from authoritative government agencies.

---

## 1. Federal Highway Administration (FHWA) Major Projects & GAO Audits
* **Authoritative Source:** U.S. Department of Transportation (USDOT) - Federal Highway Administration (FHWA)
* **Official Access Portal:** [FHWA Major Projects Program Directory](https://www.fhwa.dot.gov/programadmin/mega/) & [GAO Infrastructure Audits](https://www.gao.gov/)
* **Format:** Tabular reports / CSV / PDF Audited Financial Statements
* **Domain Scope:** Civil megaprojects with total estimated cost $\ge \text{US\$ 100 Million}$ (interstates, viaducts, transit corridors, multimodal tunnels).
* **Licensing:** U.S. Federal Government Open Data (Open & Free for public use).

### Key Features & Model Drivers
| Feature | Data Type | Physical / Business Meaning |
| :--- | :--- | :--- |
| `initial_authorized_cost_musd` | Float | Baseline budget authorized at Record of Decision (ROD) |
| `audited_final_cost_musd` | Float | Actual audited cost at commercial close |
| `cost_growth_pct` | Float | Target variable: $(Cost_{final} - Cost_{initial}) / Cost_{initial} \times 100$ |
| `planned_duration_months` | Integer | Original schedule critical path duration |
| `actual_duration_months` | Integer | Actual calendar duration to substantial completion |
| `schedule_delay_months` | Integer | Target variable: $Duration_{actual} - Duration_{planned}$ |
| `procurement_delivery_method` | Categorical | `Design-Bid-Build` (DBB), `Design-Build` (DB), `CMGC`, `P3` (Concession) |
| `environmental_clearance_tier` | Categorical | `EIS` (Environmental Impact Statement), `EA`, `CatEx` |
| `structural_complexity_index` | Float (1-10) | Geological risk, water crossings, urban density score |

---

## 2. Obrasgov.br - Ministério da Gestão e da Inovação em Serviços Públicos
* **Authoritative Source:** Governo Federal do Brasil - Ministério da Gestão e da Inovação (MGI)
* **Official Access Portal:** [Portal Brasileiro de Dados Abertos - Obrasgov.br](https://dados.gov.br/dados/conjuntos-dados/obrasgov)
* **API Endpoint:** `https://api.obrasgov.gestao.gov.br/v1/obras`
* **Format:** REST API (JSON) / Bulk CSV
* **Volume:** 700.000+ contratos públicos de obras civis e infraestrutura cadastrados em território nacional.
* **Licensing:** Open Data Brasil (Decreto nº 8.777/2016).

### Key Features & Variables
* `id_obra`: Identificador único do empreendimento federal.
* `nm_municipio` / `uf`: Localização geográfica do canteiro de obras.
* `tipo_intervencao`: Tipo de serviço (Construção, Reforma, Ampliação, Restauração).
* `vl_investimento_previsto`: Orçamento base de referência licitado ($R\$$).
* `vl_contratado`: Valor contratado junto ao consórcio/empreiteira ($R\$$).
* `vl_pago`: Desembolso acumulado liquidado até a última medição ($R\$$).
* `perc_execucao_fisica`: Percentual físico medido pela fiscalização ($0.0\% - 100.0\%$).
* `dt_inicio_prevista` / `dt_conclusao_prevista`: Cronograma contratual original.
* `situacao`: Status operacional (`Em Execução`, `Paralisada`, `Concluída`).
* `motivo_paralisacao`: Classificação de causa de paralisação (`Orçamentária`, `Abandono de Contratada`, `Licenciamento Ambiental`, `Ação Judicial`).

---

## 3. Banco Central do Brasil (SGS) & FGV IBRE - Séries Macroeconômicas
* **Authoritative Source:** Banco Central do Brasil (BACEN) & Fundação Getulio Vargas (FGV IBRE)
* **Official Access Portal:** [BACEN Sistema Gerenciador de Séries Temporais (SGS)](https://dadosabertos.bcb.gov.br/)
* **Format:** REST API / JSON / CSV
* **Purpose:** Feature engineering para choques macroeconômicos exógenos de suprimentos e inflação setorial.

### Mapeamento de Séries Temporais Oficiais
| Série SGS | Indicador | Descrição e Aplicação |
| :--- | :--- | :--- |
| `7458` | **INCC-M Geral** | Índice Nacional de Custo da Construção (base mensal) |
| `7461` | **INCC Materiais e Equipamentos** | Pressão inflacionária em insumos industriais |
| `7462` | **INCC Mão de Obra** | Reajustes de convenções coletivas da construção pesada |
| `433` | **IPCA** | Inflação oficial de referência para reajuste monetário |
| `21619` | **Custo Unitário Básico (CUB/m²)** | Referência de custo paramétrico por metro quadrado |

---

## Automated Ingestion Script Example
```python
import urllib.request
import json
import pandas as pd

def fetch_bacen_sgs_series(series_code: int, start_date: str = "01/01/2018") -> pd.DataFrame:
    """Fetches official macroeconomic construction cost index directly from BACEN SGS API."""
    url = f"https://api.bcb.gov.br/dados/serie/bcdata.sgs.{series_code}/dados?formato=json&dataInicial={start_date}"
    req = urllib.request.Request(url, headers={"User-Agent": "CivilDataPlatform/1.0"})
    with urllib.request.urlopen(req) as resp:
        data = json.loads(resp.read().decode())
    df = pd.DataFrame(data)
    df['data'] = pd.to_datetime(df['data'], format='%d/%m/%Y')
    df['valor'] = pd.to_numeric(df['valor'])
    return df
```

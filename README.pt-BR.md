# Estudo de Caso: Dunder Mifflin RevOps

[![CI](https://github.com/anacartola/dunder-mifflin-revops/actions/workflows/ci.yml/badge.svg)](https://github.com/anacartola/dunder-mifflin-revops/actions/workflows/ci.yml) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[English](README.md) | **Português** | [Français](README.fr.md) | [简体中文](README.zh-CN.md)

Um exercício autocontido de analytics de RevOps: um conjunto de dados sintético de
CRM no estilo SaaS B2B, três perguntas abertas de um CRO e uma solução resolvida.

Feito para ser **baixado e tentado**, não apenas lido. Tente primeiro, depois compare.

---

## O cenário

A Dunder Mifflin Paper Company está reformulando como sua organização de vendas faz
forecast. Você acaba de entrar como analista de dados de RevOps. O CRO, Michael
Scott, faz três perguntas numa reunião de segunda-feira:

1. *Erramos as bookings do Q1 em 18%. O que estourou o forecast?*
2. *Onde os deals estão travando, e por quê?*
3. *Com o pipe de hoje, estamos no caminho para o próximo trimestre?*

O briefing é deliberadamente ambíguo. Parte do exercício é decidir o que as
perguntas realmente significam e então dizer isso em voz alta. Todo entregável deve
declarar suas premissas e ressalvas explicitamente.

> Uma das três perguntas não pode ser respondida como foi feita. Descobrir qual, e
> conseguir provar isso, é o ponto.

**Limite sugerido: 90 minutos** para uma primeira passada.

---

## Os dados

Quatro tabelas em `data/`, ~700 deals ao longo de 2024 e do primeiro semestre de 2025.

| Arquivo | Linhas | Granularidade |
|---|---|---|
| `deals_meta.csv` | 700 | uma linha por deal, estado atual |
| `deals_snapshot.csv` | 8.321 | uma linha por deal por semana, histórico do pipeline |
| `owners.csv` | 13 | vendedores, segmentos, gestores, tempo de casa |
| `targets.csv` | 104 | meta trimestral de bookings por vendedor |

```
deals_meta      deal_id, owner_id, created_date, close_date, deal_stage,
                forecast_category, record_type, deal_type, deal_order_type,
                account_industry, account_region, account_size, deal_source,
                deal_amount

deals_snapshot  deal_id, snapshot_date, stage, forecast_category, amount,
                close_date, owner_id

owners          owner_id, name, email, role_start_date, role_end_date, role,
                manager, segment

targets         quarter, owner_id, segment, target_amount
```

Estágios: `Prospecting → Qualification → Proposal → Negotiation → Closed Won / Closed Lost`

Os dados têm as imperfeições que você esperaria de uma exportação real de CRM. Não
são bugs, são o exercício.

---

## Findings (Resultados)

*O exercício é mais divertido se você tentar primeiro — desenvolvimento completo em [`notebooks/revops_analysis.ipynb`](notebooks/revops_analysis.ipynb).*

<details>
<summary><b>Resumo executivo — clique para expandir</b></summary>

<br>

**Método.** Métricas padrão de RevOps (win rate, pipeline ponderado com pesos por categoria calibrados pelo histórico, coverage, velocidade por estágio) mais uma camada de rigor: intervalos de confiança de Wilson, teste qui-quadrado de independência (com verificação por permutação), limites de controle por estágio (SPC) para o limiar de stall, estatísticas cientes de assimetria, e um Pareto — com moldura DMAIC.

**Baseline.** 700 deals, 109 fechados ganhos. Win rate **48,7%** (IC 95% 42–55%). O deal típico é a **mediana ~US$ 58,5k**, não a média ~US$ 70k (o valor é assimétrico à direita, skew 2,6); ciclo ~90 dias. O win rate parece maior em Cross-Sell (59%) e menor em New Client (43%) — mas os intervalos de confiança se sobrepõem, então o ranking é *sugestivo, não estabelecido*.

**Q1 — "Erramos o Q1 em 18%. O que estourou o forecast?" → Não pode ser respondida como foi feita.**
A tabela `targets` está corrompida: as metas trimestrais brutas inflam exponencialmente, chegando a ~**93.000×** a base de 2024-Q1 até 2025-Q4. Sem meta confiável, o "miss de 18%" não pode ser verificado; uma reconstrução a partir do único trimestre calibrado mostra uma diferença de ~**9%**. Esta é a pergunta que o briefing não sustenta.

**Q2 — "Onde os deals estão travando, e por quê?" → Um problema de processo, não de pessoas ou canais — testado, não afirmado.**
115 de 445 deals abertos estão travados. Um teste qui-quadrado de independência **não encontra associação significativa** entre travamento e gestor, canal, tamanho de conta ou segmento (p = 0,30–0,90; premissas de contagem esperada atendidas; teste de permutação concorda) — evidência de um *processo* quebrado, não de um vendedor fraco ou canal ruim. (O tipo de deal está no limiar, p ≈ 0,08: New Client pode travar um pouco mais.) O travamento também depende do estágio: a mediana de permanência vai de **1 semana em Negotiation a 5 em Prospecting**, então um "4 semanas" achatado rotula errado — um limite de controle por estágio é a correção. Sobre onde agir, um Pareto mostra que **4 de 8 gestores concentram ~80% do valor travado**.

**Q3 — "Com o pipe de hoje, estamos no caminho para o próximo trimestre?" → Bookings vão bem; o sinal de pipeline não é confiável neste export.**
As bookings do Q2 2025 estão adiantadas (projetando ~**US$ 1,70M** vs a meta **reconstruída** de **US$ 1,17M** (ver Q1), **+46%**). O pipeline aberto *parece* cair **63%** nas últimas quatro semanas (US$ 8,46M → US$ 3,12M) — mas isso coincide exatamente com o fim da janela do export: a cobertura por snapshot cai de ~130 para **95 deals/semana** e **nenhum deal novo entra nas últimas ~3 semanas**. O "colapso" é, em grande parte, um **artefato dos dados truncados**, não um declínio comprovado. Veredito honesto: as bookings estão no ritmo; o indicador antecedente é ilegível perto do corte — um segundo sinal que os dados não sustentam por completo.

</details>

---

## Como executar

O projeto fixa a versão do Python (`.python-version`) e suas dependências
(`pyproject.toml` / `poetry.lock`), então um clone novo roda igual em qualquer máquina.

**Com [pyenv](https://github.com/pyenv/pyenv) + [Poetry](https://python-poetry.org/) (recomendado):**

```bash
git clone <este-repo>
cd dunder-mifflin-revops

pyenv install 3.12        # pule se o 3.12 já estiver instalado; obedece o .python-version
poetry env use 3.12       # cria o venv no interpretador fixado
poetry install            # instala as dependências travadas (inclui o JupyterLab)
poetry run jupyter lab notebooks/revops_analysis.ipynb
```

No Windows, os comandos do pyenv são os mesmos com o [pyenv-win](https://github.com/pyenv-win/pyenv-win).

**Sem o Poetry (pip puro):** crie um virtualenv no Python 3.12 e
`pip install -r requirements.txt`, depois abra o JupyterLab.

Sem conta em nuvem, sem autenticação, sem chaves de API. Tudo é lido de `data/`.
As saídas geradas são escritas em `output/` e estão no gitignore.

A cada push, o CI executa o notebook inteiro num runner limpo (o badge **CI**
acima) — então "baixar e rodar em qualquer lugar" é verificado automaticamente, não só prometido.

---

## Estrutura do repositório

```
data/                    quatro CSVs, a entrada
notebooks/
  revops_analysis.ipynb  solução resolvida
output/                  tabelas geradas, no gitignore
```

O notebook segue Ask → Prepare → Process → Analyse → Share (Perguntar → Preparar →
Processar → Analisar → Compartilhar). A seção Process é onde vivem as premissas, e é
a seção que vale a pena questionar.

---

## Procedência dos dados e licença

O conjunto de dados é **sintético**. Nenhuma empresa, cliente, deal ou pessoa real
aparece nele. Os nomes são personagens de *The Office* (EUA), usados como rótulos
reconhecíveis para que um achado por vendedor seja mais fácil de discutir do que
"owner_9". Os endereços de e-mail usam o TLD `.invalid`, reservado pela RFC 2606, e
nunca podem resolver para uma caixa de correio real.

*The Office*, a Dunder Mifflin e os nomes dos personagens são propriedade de seus
respectivos detentores de direitos e são usados aqui apenas como rótulos em um
conjunto de dados educacional, gratuito e não comercial. Nenhuma afiliação ou
endosso está implícito.

O **código** deste repositório é licenciado sob a [Licença MIT](LICENSE). Os
**dados e o conteúdo escrito** estão sob [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — livres para
compartilhar e adaptar com atribuição.

# Étude de cas : Dunder Mifflin RevOps

[English](README.md) | [Português](README.pt-BR.md) | **Français** | [简体中文](README.zh-CN.md)

Un exercice d'analytique RevOps autonome : un jeu de données CRM synthétique de type
SaaS B2B, trois questions ouvertes posées par un CRO, et une solution corrigée.

Conçu pour être **téléchargé et tenté**, pas seulement lu. Essayez d'abord, puis comparez.

---

## Le scénario

Dunder Mifflin Paper Company repense la façon dont son organisation commerciale
établit ses prévisions. Vous venez d'arriver comme analyste de données RevOps. Le
CRO, Michael Scott, pose trois questions lors d'une réunion du lundi :

1. *Nous avons manqué les bookings du T1 de 18 %. Qu'est-ce qui a fait dérailler la prévision ?*
2. *Où les deals stagnent-ils, et pourquoi ?*
3. *Compte tenu du pipe actuel, sommes-nous sur la bonne voie pour le trimestre prochain ?*

Le brief est délibérément ambigu. Une partie de l'exercice consiste à décider ce que
les questions signifient réellement, puis à le dire explicitement. Chaque livrable
doit énoncer ses hypothèses et ses réserves de façon explicite.

> L'une des trois questions ne peut pas recevoir de réponse telle qu'elle est posée.
> Trouver laquelle, et pouvoir le prouver, c'est là tout l'enjeu.

**Durée suggérée : 90 minutes** pour une première passe.

---

## Les données

Quatre tables dans `data/`, ~700 deals sur 2024 et le premier semestre 2025.

| Fichier | Lignes | Granularité |
|---|---|---|
| `deals_meta.csv` | 700 | une ligne par deal, état actuel |
| `deals_snapshot.csv` | 8 321 | une ligne par deal et par semaine, historique du pipeline |
| `owners.csv` | 13 | commerciaux, segments, managers, ancienneté |
| `targets.csv` | 104 | objectif trimestriel de bookings par commercial |

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

Étapes : `Prospecting → Qualification → Proposal → Negotiation → Closed Won / Closed Lost`

Les données présentent les aspérités que l'on attend d'un véritable export CRM. Ce ne
sont pas des bugs, c'est l'exercice.

---

## Exécution

Le projet fige sa version de Python (`.python-version`) et ses dépendances
(`pyproject.toml` / `poetry.lock`), de sorte qu'un clone récent s'exécute de la même
manière sur n'importe quelle machine.

**Avec [pyenv](https://github.com/pyenv/pyenv) + [Poetry](https://python-poetry.org/) (recommandé) :**

```bash
git clone <ce-depot>
cd dunder-mifflin-revops

pyenv install 3.12        # ignorer si 3.12 est déjà installé ; respecte le .python-version
poetry env use 3.12       # crée le venv sur l'interpréteur figé
poetry install            # installe les dépendances verrouillées (JupyterLab inclus)
poetry run jupyter lab notebooks/revops_analysis.ipynb
```

Sous Windows, les commandes pyenv sont les mêmes avec [pyenv-win](https://github.com/pyenv-win/pyenv-win).

**Sans Poetry (pip simple) :** créez un virtualenv sur Python 3.12 et
`pip install -r requirements.txt`, puis lancez JupyterLab.

Pas de compte cloud, pas d'authentification, pas de clés d'API. Tout est lu depuis
`data/`. Les sorties générées sont écrites dans `output/` et sont ignorées par git.

---

## Structure du dépôt

```
data/                    quatre CSV, l'entrée
notebooks/
  revops_analysis.ipynb  solution corrigée
output/                  tables générées, ignorées par git
```

Le notebook suit Ask → Prepare → Process → Analyse → Share (Comprendre → Préparer →
Traiter → Analyser → Partager). La section Process est celle où résident les
hypothèses, et c'est la section qui mérite d'être remise en question.

---

## Provenance des données et licence

Le jeu de données est **synthétique**. Aucune entreprise, aucun client, aucun deal ni
aucune personne réelle n'y figure. Les noms sont des personnages de *The Office* (US),
utilisés comme étiquettes reconnaissables afin qu'un constat au niveau d'un commercial
soit plus facile à évoquer que « owner_9 ». Les adresses e-mail utilisent le TLD
`.invalid` réservé par la RFC 2606 et ne peuvent jamais correspondre à une boîte aux
lettres réelle.

*The Office*, Dunder Mifflin et les noms des personnages sont la propriété de leurs
ayants droit respectifs et ne sont utilisés ici que comme étiquettes dans un jeu de
données pédagogique, gratuit et non commercial. Aucune affiliation ni aucun
endossement n'est sous-entendu.

Le jeu de données, le notebook et le texte de ce dépôt sont diffusés pour un usage
éducatif gratuit.

# Troop AI-setup

De setup waarmee wij met Claude Code werken: twee plugins uit de marketplace en vijf
eigen skills. De plugins werken meteen. De werkpakket-skills bevatten placeholders die
per repo ingevuld moeten worden; zie [Aanpassen per repo](#aanpassen-per-repo).

## 1. Plugins installeren

In Claude Code:

```
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail

/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

- **ponytail** (https://github.com/DietrichGebert/ponytail) laat de agent de kleinste
  wijziging kiezen die het probleem oplost, en elk antwoord eindigen met wat er niet is
  gedaan en welk risico je moet weten.
- **agent-skills** (https://github.com/addyosmani/agent-skills) levert de vaste stappen:
  specificeren, plannen, incrementeel bouwen met tests, reviewen en de pre-launch check.
  De skills hieronder roepen dit pakket aan, dus zonder deze plugin werken ze niet.

## 2. Skills installeren

Globaal, voor elk project:

```
cp -R skills/* ~/.claude/skills/
```

Of per project, alleen voor deze repo:

```
mkdir -p .claude/skills && cp -R skills/* .claude/skills/
```

| Skill | Wat het doet | Per repo instellen |
| --- | --- | --- |
| `/pipeline-dev` | Eén feature van idee tot pull request in één run, met twee akkoordmomenten. | nee |
| `/pr-check` | Een bestaande PR beoordelen, samen beslissen wat je fixt, en eindigen met GO of NO-GO. | nee |
| `/ticket` | Korte tickets in een Notion-database: titel, story, omschrijving, acceptatiecriteria. | nee |
| `/wp` | Maakt één werkpakket: eigen worktree en branch, een spec en een taakplan, en één akkoordmoment voordat er code komt. | ja |
| `/wp-run` | Bouwt goedgekeurde werkpakketten af tot en met een PR, met code review, security review en ponytail-review. | ja |

## 3. Aanpassen per repo

`/wp` en `/wp-run` bevatten placeholders. Zoek op `<<` en vul ze in:

| Placeholder | Wat erin moet |
| --- | --- |
| `<<PROJECT>>` | De naam van het project. |
| `<<INTEGRATIEBRANCH>>` | De branch waar pull requests naartoe gaan, bijvoorbeeld `main` of `acceptatie`. |
| `<<POORTEN-BRON>>` | De plek waar de testcommando's staan die CI ook draait, bijvoorbeeld `README.md` § Testen. Zet ze op één plek, niet in de skill zelf: twee kopieën lopen uit elkaar. |
| `<<VALKUIL-1..3>>` | De drie dingen die in dit systeem echt misgaan. Haal ze uit je laatste incidenten en reviewbevindingen, niet uit een lijstje met principes. |
| `<<CI-WORKFLOW>>` | De naam van de workflow die de tests op een PR draait. |

Verder nodig in de repo:

- `.claude/constitution.md` met de harde regels. Zie `templates/constitution.md`.
- `CLAUDE.md` met conventies, architectuur, bouwvolgorde en valkuilen.
- In `.gitignore`: `.claude/worktrees/` en `.claude/wp-logs/`. Die blijven lokaal.
- `gh` ingelogd, voor de pull request en voor het nummeren van nieuwe werkpakketten.

Reken op een halve dag samen instellen voordat de eerste werkpakketrun zinvol is.

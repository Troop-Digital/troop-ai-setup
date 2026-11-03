# Constitution — de onwrikbare regels

Zeven regels die voor **elke** agent gelden, in elke fase, in elk werkpakket. Dit bestand is de
samenvatting die elke subagent meekrijgt. Wijzigen mag alleen de eigenaar. Een werkpakket,
review-verdict of suggestie die één van deze regels oprekt, is daarmee automatisch een
productbeslissing en gaat naar hem of haar.

Regel 1 tot 3 vul je zelf in: de drie dingen die in dít systeem echt misgaan, afgeleid uit je
laatste incidenten en reviewbevindingen. Schrijf het mechanisme op, niet de bedoeling. Kun je een
regel niet nalopen in een diff, dan kun je hem niet handhaven.

1. **<<EIGEN REGEL 1>>** Bijvoorbeeld de isolatie tussen klanten: elke query op klantdata loopt via
   één vast mechanisme, en een pad zonder dat filter is een blocker.

2. **<<EIGEN REGEL 2>>** Bijvoorbeeld welke data nooit in een log terechtkomt.

3. **<<EIGEN REGEL 3>>** Bijvoorbeeld het commando dat nooit op een live database draait, of de
   sleutel die bepaalt of data dubbel binnenkomt.

Regel 4 tot 7 gelden voor elk project waarin een agent werkt:

4. **Code en commentaar zijn Engels, spaarzaam, en alleen over het waarom.** Eén zin, nooit een
   verhaal, nooit review-historie.

5. **Tests worden niet aangepast om een poort groen te krijgen.** Een rode test meldt een defect in
   de feature-code, niet in zichzelf.

6. **Nooit mergen, nooit deployen.** De PR is het overdrachtspunt; mergen is een mensenbeslissing.

7. **Blijf binnen de scope-bestandslijst van het pakket.** De buiten-scope-lijst is een verbod, geen
   suggestie — ook niet "even meenemen". De reviewers controleren erop.

> Orchestrators: geef dit bestand mee in de prompt van elke subagent die je start.
> Het is klein genoeg om altijd mee te gaan. Houd het onder de twee kilobyte.

## Wat je mechanisch afdwingt

Wat je kunt blokkeren, blokkeer je. Zet in `.claude/settings.json` onder `permissions.deny` in elk
geval de destructieve databasecommando's, `git merge`, `gh pr merge`, `git push --force`,
`git reset --hard` en `rm -rf`. Een deny-lijst doet meer dan een zin in een instructiebestand.

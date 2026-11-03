---
name: wp-run
description: Voer goedgekeurde werkpakketten één voor één uit tot en met een PR naar de integratiebranch — bouwen met /build auto, daarna code review, security review en ponytail-review, met een logboek per pakket. Gebruik voor "draai wp-015" of "werk deze vier pakketten af".
argument-hint: wp-015 [wp-016 wp-017 ...]
disable-model-invocation: true
---

# Werkpakketten uitvoeren

Eén pakket tegelijk, in de opgegeven volgorde, elk tot en met een eigen PR. Nooit twee
pakketten parallel. Een pakket is af of expliciet geblokkeerd voordat het volgende start.

Lees eerst `.claude/constitution.md` en `CLAUDE.md`.

## Vooraf, één keer

1. Zoek per argument de worktree en branch via `git worktree list`. Die lijst is het
   register; er is geen apart registerbestand. Ontbreekt de worktree of `SPEC.md`, dan is
   dat pakket niet goedgekeurd: meld het en sla het over. Lees `tasks/todo.md`: afgevinkte
   taken zijn al gebouwd, dus een hervatting begint bij de eerste openstaande taak.
2. Lees de bestandslijst uit elke `SPEC.md`. Raken twee pakketten in de rij hetzelfde
   bestand, **stop en leg het voor** — ze forken allemaal van dezelfde basisbranch, dus
   overlap levert conflicten op die hij bij het mergen moet uitzoeken.
3. Herhaal de rij met per pakket de branch en het aantal openstaande taken, en begin dan.

## Per pakket

Delegeer elk pakket aan één subagent met een eigen context. De orchestrator bouwt niet
mee en houdt alleen het logboek bij; zonder die scheiding loopt de sessie na twee
pakketten vol. Geef de subagent mee: het worktree-pad, `SPEC.md`, `tasks/plan.md`,
`.claude/constitution.md`, `CLAUDE.md`, de basisbranch en het logboekpad. Lees daarna
zijn rapport en controleer de PR zelf.

1. **Baseline.** In de worktree moet `git status --porcelain` schoon zijn en het
   fork-punt bestaan. Kopieer de dependency-mappen die de poorten nodig hebben, symlink
   ze nooit. Kopieer geen `.env`; tests draaien tegen de testconfiguratie van het project.
   Een rode baseline is een setupkwestie die je apart rapporteert, geen reden voor
   scopegroei.
2. **Bouwen.** Invoke `agent-skills:build` met `auto`. de opdrachtgever's akkoord op spec én
   taakplan, met de commitref in het logboek, ís de enkele menselijke poort die die skill
   vraagt: geef dat expliciet mee zodat hij niet opnieuw om goedkeuring vraagt. De skill
   doet per taak rood-groen, de suite en een eigen commit. Verzwak geen bestaande test
   (regel 5) en blijf binnen de bestandslijst (regel 7).
3. **Code review.** Invoke `code-review` met `high --fix` op de branch-diff. Draai daarna
   de poorten opnieuw en commit de fixes apart. Start `ultra` of `max` nooit zelf:
   die zijn van de opdrachtgever en worden apart gefactureerd.
4. **Security review.** Invoke `security-review`. Verwerk de bevindingen binnen scope,
   test opnieuw, commit apart.
5. **Ponytail-review.** Invoke `ponytail:ponytail-review`. Dat rapport verandert zelf
   niets: neem over wat binnen scope valt en log de rest.
6. **Documentatie.** Werk `documentation/<feature>/<feature>.md` bij, en `README.md` of
   `CLAUDE.md` alleen wanneer het gedrag of de conventie daar beschreven staat.
7. **PR.** Commit met expliciete paden, push de branch en open één PR naar
   `<<INTEGRATIEBRANCH>>`.
   Zoek eerst een bestaande PR voor deze branch, zodat hervatten geen tweede oplevert.
   Body: probleem, aanpak, tests, de drie reviews met hun uitkomst, resterende suggesties
   en de Notion-link.
8. **CI.** Controleer de checks op de laatste PR-commit, en of die commit de PR-head is.
   <<CI-WORKFLOW>> draait de tests op elke PR; een deploy-job hoort op een PR
   overgeslagen te worden. Pending, rode of onbereikbare CI is geen afgerond pakket: leg
   de status
   met reden vast en meld het pakket als niet gereed. Gebruik begrensde wachttijden met
   voortgangsberichten, geen eeuwige retrylus.

## De poorten

De lijst staat in <<POORTEN-BRON>> en is daar leidend, zodat hij niet op twee plekken
uit elkaar loopt. Groen betekent elke poort uit die lijst, gedraaid in de worktree-root.
Hoe je één test los draait, staat op diezelfde plek.

## Logboek

Append na elke fase een regel in `.claude/wp-logs/wp-NNN.md`: fase, uitkomst,
testaantallen, reviewbevindingen per ronde, open acties, PR-nummer en de gecontroleerde
CI-status. Houd daarnaast `.claude/wp-logs/queue-<datum>.md` bij met één regel per
pakket: status, PR, blokkade. Beide zijn gitignored en horen niet in een PR.

Een pakket is af bij volledige scope, groene poorten, nul blokkerende bevindingen,
bijgewerkte documentatie en groene CI. Al het andere is geblokkeerd.

## Grenzen

- Stopt een pakket, dan stopt de rij daar. Niet doorgaan naar het volgende en niet stil
  om de blokkade heen bouwen.
- **Nooit mergen, nooit deployen** (regel 6). De PR is het overdrachtspunt; een push naar
  `<<INTEGRATIEBRANCH>>` raakt een echte omgeving.
- Nooit `git add -A`: alleen expliciete paden, zodat logs en werkbestanden buiten de PR
  blijven.
- Scopevreemde problemen apart vastleggen, niet meenemen. Geen herstel-PR mergen om de
  basis te verversen.
- Zet toolrechten of de sandbox nooit uit om de run te laten doorlopen.
- Notion lezen hoort bij de opdracht. Schrijven in Notion, tickets, comments of berichten
  versturen vereist een aparte opdracht van de opdrachtgever.

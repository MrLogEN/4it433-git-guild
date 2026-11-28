# Šablony pravidel pro GitHub

## Vysvětlení

Doporučený postup je **pracovní větev → `dev` → `main`**. Změny z pracovních větví se integrují přes pull request (PR) do `dev`; připravené změny se pak z `dev` posílají přes PR do `main`.

| Šablona | Co vynucuje |
| --- | --- |
| [`main-protection.json`](main-protection.json) | Blokuje force push do `main` a vyžaduje schválený PR pro změny. |
| [`dev-protection.json`](dev-protection.json) | Vyžaduje schválený PR pro změny v `dev`. |
| [`commit-message-format.json`](commit-message-format.json) | Kontroluje zprávy commitů na všech větvích. Vyžaduje, aby cílový repozitář podporoval **Restrict commit metadata**. |

Obě šablony ochrany větví vyžadují schválení od code ownera a schválení posledního pushnutí, které změnilo obsah PR, někým jiným než jeho autorem. Jedno schválení může splnit obě podmínky, pokud ho po posledním pushnutí udělí code owner, který tento push neprovedl.

### Formát zpráv commitů

```regex
^(ingest|bronze|silver|dq|gov|sablony|ci|infra|docs|milnik)(\([a-z0-9_-]+\))?!?: \S.{0,70}$
```

Platné příklady: `docs: upřesni pravidla` a `silver(model_1)!: změň schéma`. Po `: ` musí popis začínat znakem jiným než mezera či jiný bílý znak a může mít nejvýše 71 znaků. Zpráva má jeden řádek; šablona připouští koncový znak nového řádku, který může přidat Git. Stejný formát použij i pro zprávu merge nebo squash commitu, pokud ji GitHub při sloučení vytváří.

### Import do cílového repozitáře

1. Před aktivací pravidel připrav větve `main` a `dev` a v obou nastav `.github/CODEOWNERS`. Pokud má být schválení od code ownera povinné pro každý PR, pokryj všechny soubory, například výchozím pravidlem `* @uzivatel`, kde má uvedený uživatel právo zápisu do repozitáře. Praktické je určit alespoň dva code ownery, aby mohl schválit i někdo jiný, když jeden z nich změny připravuje.
2. V cílovém repozitáři otevři **Settings → Rules → Rulesets → New ruleset → Import a ruleset**. Nahraj každý JSON samostatně, zkontroluj cílové větve a pravidla ulož. Šablony mají nastavený stav **Active**, takže po uložení začnou platit.

Ruleset pro `main` kontroluje cílovou větev PR. Pokud má být `dev` jediným povoleným zdrojem PR do `main`, přidej samostatnou povinnou kontrolu zdrojové větve.

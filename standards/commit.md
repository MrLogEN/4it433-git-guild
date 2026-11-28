# Konvence commit zpráv

Každý commit v repozitáři má zprávu ve tvaru **`oblast: popis`**. Oblast říká, které části datové platformy se změna týká, popis říká, co se změnilo. Z historie commitů je pak na první pohled vidět, co se dělo v ingestu, v Bronze nebo v Silver, a CI umí zprávy zkontrolovat.

Konvence vychází z [Conventional Commits](https://www.conventionalcommits.org/cs/), ale je jednodušší a přizpůsobená našemu projektu: místo typu změny (`feat`, `fix`, …) uvádíme **oblast**, protože u datové platformy je důležitější *kde* se něco změnilo než *jaký druh* změny to byl. Druh změny vyjadřuje první slovo popisu.

## Formát

```
oblast(předmět)!: popis

tělo (volitelné)
```

| Část | Povinná | Význam |
| --- | --- | --- |
| `oblast` | ano | Jedna z oblastí v tabulce níže, malými písmeny. |
| `(předmět)` | ne | Konkrétní tabulka, zdroj nebo notebook, např. `(cestujici)`, `(golemio)`. Malými písmeny, bez diakritiky, bez mezer. |
| `!` | ne | Změna, která rozbije navazující práci: změna schématu tabulky, přejmenování sloupce, změna rozhraní helperu. |
| `popis` | ano | Co se změnilo. Pravidla viz níže. |
| `tělo` | ne | Proč a jak. Odděluje se prázdným řádkem. Používá se hlavně u commitů z GitHubu nebo z lokálního Gitu. |

## Oblasti

| Oblast | Kdy ji použít | Příklady položek |
| --- | --- | --- |
| `ingest` | Načítání dat ze zdrojů: SQL databáze, API, otevřená data, CSV, průběžný sběr | Copy joby, pipeline, notebooky pro API |
| `bronze` | Ukládání surových dat do Bronze, technické sloupce, kontrola počtů | `nb_bronze_*` |
| `silver` | Čištění, typy, deduplikace, číselníky, propojení, historizace (SCD2) | `nb_silver_*` |
| `dq` | Pravidla kvality dat, karanténa, DQ report | DQ notebooky, pravidla |
| `gov` | Governance: datový slovník, klasifikace sloupců, pseudonymizace, matice rolí, datové kontrakty | `docs/slovnik/`, kontrakty |
| `sablony` | Kostra notebooku, sdílený notebook `nb_common`, šablona README a PR | `nb_common`, `.github/pull_request_template.md` |
| `ci` | GitHub Actions a kontrolní skripty | `.github/workflows/`, `ci/` |
| `infra` | Nastavení Fabricu: lakehouse, workspace, connections, prostředí | `lh_*`, nastavení workspace |
| `docs` | Dokumentace, která nespadá do jiné oblasti | `README.md`, `docs/` |

Když se změna týká víc oblastí, rozdělte ji na víc commitů. Ve Fabricu jde v panelu Source control zaškrtnout jen některé položky, takže to obvykle není problém. Pokud rozdělit nejde, zvolte oblast, které se změna týká nejvíc.

## Pravidla pro popis

1. **Česky, s diakritikou.**
2. **Začíná podstatným jménem děje:** *přidání*, *oprava*, *úprava*, *odstranění*, *načtení*, *sjednocení*, *přejmenování* apod. Stejný styl používá zadání (`bronze: načtení všech tabulek`).
3. **Malým písmenem na začátku, bez tečky na konci.**
4. **Nejvýš 72 znaků** pro celý první řádek včetně oblasti.
5. **Konkrétně:** z popisu musí být jasné, co se změnilo, i bez otevření diffu. Ne „úpravy“, „fix“ nebo „wip“.

Typické první slovo podle druhu změny:

| Druh změny | Slovo |
| --- | --- |
| Nová funkčnost | přidání, zavedení, načtení |
| Oprava chyby | oprava |
| Změna chování | úprava, změna, sjednocení |
| Úklid bez změny chování | refaktoring, přejmenování, zjednodušení |
| Odstranění | odstranění |

## Příklady

### Dobře

```
ingest(golemio): přidání denního stažení poloh vozidel
ingest: načtení všech tabulek ze SQL_4IT433 přes Copy job
bronze: přidání sloupců _source, _ingested_at a _run_id
bronze(podnet_cestujiciho): uložení sloupců TIME jako textu
silver(cestujici): převod jmen psaných VELKÝMI na Velké první
silver(prepravni_kontrola): sjednocení druhu dopravy na BUS/TRAM/METRO/TROLLEYBUS
silver!: přejmenování person_id na person_sk v kontrolách
dq: přidání příznaku pro události s časem v budoucnosti
dq(refund): oprava detekce FAILED vratek po COMPLETED
gov: klasifikace sloupců tabulky cestujici
gov: pravidla pseudonymizace osobních údajů
sablony: přidání write_bronze() do nb_common
sablony!: změna signatury log_run() na log_run(run_id, tabulka, pocet)
ci: kontrola prefixů tabulek v noteboocích
ci: přepnutí kontroly tajných klíčů na blokující
infra: vytvoření lakehouse lh_team_12
docs: doplnění výsledků milníku 1 do README
```

### Špatně a proč

| Zpráva | Problém | Správně |
| --- | --- | --- |
| `update` | Chybí oblast, popis nic neříká | `silver(platba): oprava typu částky na decimal` |
| `Silver: Načtení tabulek.` | Velká písmena a tečka | `silver: načtení tabulek z Bronze` |
| `silver: fix` | Anglicky a nekonkrétně | `silver(refund): oprava deduplikace vratek` |
| `silver: upravil jsem čištění cestujících` | Sloveso v minulém čase místo podstatného jména | `silver(cestujici): úprava čištění jmen` |
| `bronze + silver: nové tabulky` | Dvě oblasti v jednom commitu | dva commity: `bronze: …` a `silver: …` |
| `feat: přidání DQ` | Typ z Conventional Commits místo oblasti | `dq: přidání pravidel pro tabulku platba` |
| `silver(Cestující): oprava` | Předmět s velkým písmenem a diakritikou | `silver(cestujici): oprava duplicit` |
| `wip` | Rozpracovaná práce se necommituje do `dev` | commit až po dokončeném kroku |

## Tělo zprávy

Tělo je volitelné. Napište ho, když z popisu není zřejmé **proč** se změna dělá nebo **jaký má dopad**. Hodí se hlavně u změn označených `!`.

```
silver!: přejmenování person_id na person_sk v kontrolách

Cizí klíč na cestujícího se v tabulce prepravni_kontrola jmenoval
person_id, ale v tabulce cestujici je to person_sk. Sjednoceno na
person_sk. Dotazy, které spojují kontroly s cestujícími, je potřeba
upravit.
```

Ve Fabricu se do pole pro commit zprávu píše obvykle jen první řádek. Tělo se přidává hlavně u commitů z GitHubu nebo z lokálního Gitu.

## Commity z Fabricu

1. Otevřete panel **Source control** ve workspace.
2. Zaškrtněte jen položky, které patří k jedné oblasti.
3. Napište zprávu podle této konvence.
4. Commit jde do větve `dev`.

Commitujte po každém ucelém kroku, ne po každé drobné úpravě a ne až na konci milníku.

## Pull Requesty a merge commity

- **Název PR** dodržuje stejný formát jako commit, např. `silver: čištění všech tabulek pro milník 1`. Když PR obsahuje víc oblastí, použijte tu hlavní nebo `milnik: …` u PR, který uzavírá milník.
- **Merge commity** vytvořené GitHubem (`Merge pull request #12 from …`) a revert commity (`Revert "…"`) se nepřepisují a kontrola je přeskakuje.

## Kontrola v CI

CI kontroluje první řádek každého commitu v PR tímto regulárním výrazem:

```
^(ingest|bronze|silver|dq|gov|sablony|ci|infra|docs|milnik)(\([a-z0-9_-]+\))?!?: \S.{0,70}$
```

Přeskakuje commity začínající `Merge ` a `Revert "`. Kontrola zatím jen **varuje**, PR neblokuje. Na blokující ji přepneme, až si na konvenci zvykne většina týmů.

Když CI nahlásí chybu u commitu, který je už v `dev`, nepřepisujte historii. Další commity pište správně a v PR to uveďte do popisu.

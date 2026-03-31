# CLAUDE.md — HOI4 Custom Buildings Mod

## Progetto

Mod per **Hearts of Iron IV** che aggiunge nuovi edifici custom al gioco. Il progetto è in fase attiva di sviluppo: nuovi edifici, tecnologie, eventi, decisioni e focus tree potranno essere aggiunti incrementalmente in questa stessa cartella.

**Lingua principale del modder:** Italiano (ma il codice e i commenti sono in inglese per convenzione HOI4).
**Localizzazione:** Sempre doppia — `english` + `italian`.

---

## Stato attuale della mod

### Edifici implementati (v1.2.0)

| ID interno | Nome IT | Tipo slot | Max Lvl | Costo base | Extra/lvl |
|---|---|---|---|---|---|
| `heavy_machinery_factory` | Fabbrica Macchinari Pesanti | Non-shared | 3 | 4000 | +600 |
| `forced_labor_camp` | Campo di Lavori Forzati | Non-shared | 2 | 2500 | +400 |
| `refugee_center` | Centro di Accoglienza | Non-shared | 2 | 3000 | +500 |
| `research_center` | Centro di Ricerca | Non-shared | 2 | 6000 | +1500 |
| `naval_aerospace_complex` | Complesso Navale-Aeronautico | Non-shared | 3 | 14500 | +2750 |

> **v1.2.0 — Cambiamenti:** Tutti i costi dimezzati. Merge di `naval_parts_factory` + `aircraft_parts_factory` + `naval_tech_center` in `naval_aerospace_complex`. Bilanciamento potenziato su tutti gli edifici.

### File esistenti

```
custom_buildings/
├── descriptor.mod                                    # Descriptor interno mod
├── custom_buildings.mod                              # Descriptor utente (con path=)
├── README.md                                         # Istruzioni installazione
├── claude.md                                         # QUESTO FILE
├── common/
│   └── buildings/
│       └── zzz_custom_buildings.txt                  # Definizioni 7 edifici
└── localisation/
    ├── english/
    │   └── custom_buildings_l_english.yml             # Localizzazione EN
    └── italian/
        └── custom_buildings_l_italian.yml             # Localizzazione IT
```

### Cosa MANCA ancora

- **Icone edifici**: servono 5 icone 46x46px aggiunte allo sprite strip `GFX_buildings_strip`. I frame usati sono 17-21 (frame 22-23 non più usati dopo il merge). Serve anche un file `interface/custom_buildings.gfx` per aggiornare `noOfFrames = 21`.
- **Tecnologie**: attualmente tutti gli edifici sono disponibili da subito. Il modder potrebbe volere tech per sbloccarli.
- **Focus tree**: nessun focus tree collegato.
- **Eventi**: nessun evento collegato.
- **Decisioni**: nessuna decisione collegata.
- **Modelli 3D**: nessun modello sulla mappa (solo `show_on_map = 1` placeholder).

---

## Decisioni di design prese

1. **Slot propri (non-shared)** — gli edifici custom NON competono con fabbriche civili/militari per gli slot condivisi. Ogni edificio ha il suo cap indipendente.
2. **Disponibili da subito** — nessuna tech necessaria per costruirli. Se si aggiungono tech in futuro, aggiungere `hide_if_missing_tech = yes` alla definizione del building.
3. **Trade-off deliberati** — Il Campo di Lavori Forzati ha malus manpower/resistenza. Il Centro di Accoglienza costa consumer goods e stabilità. Mantenere questo pattern per nuovi edifici.
4. **Bilanciamento potenziato (v1.2.0)** — I bonus sono stati aumentati significativamente. Il Centro di Ricerca al 15% è molto forte — non aggiungere altri edifici con `research_speed_factor` senza bilanciare. Il `Complesso Navale-Aeronautico` è volutamente costoso (14500 base) perché combina tre edifici.
5. **Naming convention** — ID interni in snake_case inglese. Localizzazione sempre in entrambe le lingue.
6. **File prefix `zzz_`** — Il file buildings usa il prefisso `zzz_` per essere valutato DOPO i file vanilla, evitando conflitti.

---

## Guida rapida al modding HOI4 — Riferimento tecnico

### Struttura generale del codice

Il linguaggio di scripting HOI4 è basato su coppie `attributo = argomento`. I blocchi si aprono con `{` e si chiudono con `}`. I commenti iniziano con `#`. Non esiste commento multilinea.

```
my_thing = {
    attribute = value
    nested_block = {
        another = thing
    }
}
```

**Regola d'oro:** ogni `{` deve avere il suo `}`. L'indentazione non conta per il parser ma è fondamentale per la leggibilità.

### Encoding file

- **File .txt** (buildings, events, tech, ecc.): **UTF-8 senza BOM**
- **File .yml** (localizzazione): **UTF-8 CON BOM** (i primi 3 byte devono essere `EF BB BF`)
- **Immagini**: DDS formato ARGB8 senza mipmaps (o TGA/PNG in alcuni casi)

Per aggiungere il BOM a un file .yml con Python:
```python
BOM = b'\xef\xbb\xbf'
with open('file_l_english.yml', 'rb') as f:
    content = f.read()
if not content.startswith(BOM):
    with open('file_l_english.yml', 'wb') as f:
        f.write(BOM + content)
```

### Buildings — Sintassi completa

I buildings vanno in `common/buildings/*.txt` dentro un blocco `buildings = { ... }`.

```
buildings = {
    my_building = {
        # === COSTI ===
        base_cost = 10000                    # Costo base in IC (5 IC/giorno per fabbrica civile)
        per_level_extra_cost = 1500          # Costo aggiuntivo per ogni livello già costruito
        infrastructure_construction_effect = yes  # Bonus velocità da infrastruttura

        # === VISUALIZZAZIONE ===
        value = 5                            # HP base per bombardamento + valore per conferenze di pace
        icon_frame = 17                      # Frame nello sprite strip GFX_buildings_strip
        show_on_map = 1                      # Quanti modelli 3D mostrare sulla mappa
        has_destroyed_mesh = no              # Modello distrutto separato
        damage_factor = 0.5                  # Moltiplicatore danno da bombardamento
        show_modifier = yes                  # Mostra modifier nel tooltip

        # === SLOT ===
        level_cap = {
            # Per shared building (compete con fabbriche):
            # shares_slots = yes
            
            # Per non-shared state building:
            state_max = 3
            
            # Per provincial building:
            # province_max = 5
        }

        # === MODIFIER PAESE (si stackano per ogni livello globalmente) ===
        country_modifiers = {
            # enable_for_controllers = { TAG1 TAG2 }  # Opzionale: limita a certi paesi
            modifiers = {
                research_speed_factor = 0.03
                # Qualsiasi country modifier valido
            }
        }

        # === MODIFIER STATO (si applicano solo nello stato dove è costruito) ===
        state_modifiers = {
            local_building_slots_factor = 0.10
            # Qualsiasi state modifier valido
        }

        # === EFFETTI SPECIALI (simulano building vanilla) ===
        # military_production = 0.5          # Aggiunge fabbriche militari frazionali
        # general_production = 0.5           # Aggiunge fabbriche civili frazionali
        # naval_production = 0.5             # Aggiunge cantieri navali frazionali
        # infrastructure = yes               # Marca come infrastruttura
        # refinery = yes                     # Marca come raffineria
        # nuclear_reactor = yes              # Marca come reattore nucleare
        # radar = yes                        # Marca come stazione radar

        # === OPZIONALI ===
        # only_costal = yes                  # Solo province/stati costieri
        # disabled_in_dmz = yes              # Disabilitato in zone demilitarizzate
        # allied_build = yes                 # Modifier si applicano anche agli alleati
        # is_buildable = no                  # Non costruibile manualmente
        # hide_if_missing_tech = yes         # Nascosto se tech non ricercata
    }
}
```

### Localizzazione — Formato

File `.yml`, encoding UTF-8-BOM, nella cartella `localisation/<lingua>/`.
Il nome file DEVE finire con `_l_<lingua>.yml` (es. `custom_buildings_l_english.yml`).

La prima riga (dopo il BOM) deve essere `l_english:` o `l_italian:`.

```yaml
l_english:
 my_building:0 "My Building Name"
 my_building_desc:0 "Description shown in tooltip."
 my_building_plural:0 "My Buildings"
 
 # Modifier automatici per ogni building (il gioco li crea da solo):
 modifier_production_speed_my_building_factor:0 "§Y$my_building$§! construction speed"
 modifier_production_speed_my_building_factor_desc:0 "Modifies the speed of §Y$my_building$§! construction."
 modifier_state_production_my_building_factor:0 "§Y$my_building$§! construction speed"
 modifier_state_production_my_building_factor_desc:0 "Modifies the speed of §Y$my_building$§! construction in this state."
```

**Convenzioni testo:**
- `§Y...§!` = testo giallo (usato per nomi importanti)
- `§G...§!` = testo verde (bonus positivi)
- `§R...§!` = testo rosso (malus negativi)
- `$key$` = inserisce il valore di un'altra chiave di localizzazione
- `\n` = a capo
- `[TAG.GetName]` = nome dinamico del paese

### Tecnologie — Sintassi per sblocco building

Se in futuro si aggiungono tech, vanno in `common/technologies/*.txt`:

```
technologies = {
    naval_parts_tech = {
        enable_building = {
            building = naval_parts_factory
            level = 3                        # Sblocca fino a livello 3
        }
        research_cost = 2
        start_year = 1936
        
        path = {
            leads_to_tech = advanced_naval_parts_tech
            research_cost_coeff = 1
        }
        
        folder = {
            name = electronics_folder        # O un folder custom
            position = { x = 0 y = 6 }
        }
        
        categories = { industry }
        
        ai_will_do = {
            factor = 1
        }
    }
}
```

### Eventi — Sintassi base

File in `events/*.txt`:

```
country_event = {
    id = custom_buildings.1
    title = custom_buildings.1.t
    desc = custom_buildings.1.desc
    picture = GFX_report_event_generic_factory

    trigger = {
        has_country_flag = some_flag
    }

    mean_time_to_happen = {
        days = 30
    }

    option = {
        name = custom_buildings.1.a
        add_political_power = 50
    }
}
```

### Decisioni — Sintassi base

File in `common/decisions/*.txt`:

```
my_decision_category = {
    my_decision = {
        icon = generic_industry

        available = {
            num_of_civilian_factories > 10
        }

        visible = {
            always = yes
        }

        cost = 50                            # Political power
        days_remove = 30                     # Durata timer

        remove_effect = {
            random_owned_controlled_state = {
                add_building_construction = {
                    type = research_center
                    level = 1
                    instant_build = yes
                }
            }
        }

        ai_will_do = {
            factor = 1
        }
    }
}
```

### National Focus — Sintassi base

File in `common/national_focus/*.txt`:

```
focus_tree = {
    id = custom_focus_tree
    country = {
        factor = 0
        modifier = {
            add = 10
            tag = TAG
        }
    }
    default = no

    focus = {
        id = TAG_build_research_center
        icon = GFX_focus_research
        cost = 10                            # 10 = 70 giorni
        x = 5
        y = 0

        completion_reward = {
            every_owned_state = {
                limit = {
                    is_core_of = ROOT
                    infrastructure > 3
                }
                add_building_construction = {
                    type = research_center
                    level = 1
                    instant_build = yes
                }
            }
        }
    }
}
```

### Modifier utili per nuovi edifici

**Country modifiers (globali):**
- `research_speed_factor` — velocità ricerca
- `production_speed_buildings_factor` — velocità costruzione tutti gli edifici
- `production_factory_efficiency_gain_factor` — guadagno efficienza fabbriche
- `consumer_goods_factor` — % fabbriche destinate a consumer goods
- `stability_factor` — modifier stabilità
- `war_support_factor` — modifier supporto di guerra
- `political_power_factor` — guadagno potere politico
- `production_speed_<building_id>_factor` — velocità costruzione specifico edificio
- `industry_repair_factor` — velocità riparazione
- `industrial_capacity_factory` — capacità industriale fabbriche militari
- `industrial_capacity_dockyard` — capacità industriale cantieri navali
- `global_building_slots_factor` — slot costruzione globali
- `trade_opinion_factor` — opinione commerciale
- `civilian_factory_use` — fabbriche civili usate (flat, non %)

**State modifiers (locali):**
- `local_building_slots_factor` — slot costruzione nello stato
- `local_factories_factor` — output fabbriche nello stato
- `local_resources_factor` — risorse locali
- `local_manpower` — manpower locale (moltiplicatore)
- `recruitable_population_factor` — popolazione reclutabile
- `monthly_population` — crescita popolazione mensile
- `non_core_manpower` — manpower da stati non-core
- `resistance_target` — target resistenza
- `compliance_growth` — crescita conformità
- `local_ship_build_speed_factor` — velocità produzione navi locale
- `arms_factory_output_factor` — output fabbriche militari locale
- `state_resources_factor` — risorse dello stato (moltiplicatore)
- `attrition_for_controller` — attrizione per controllore
- `local_intel_to_enemies` — intel dato ai nemici

### Descriptor mod (.mod)

Servono DUE file:
1. `descriptor.mod` — dentro la cartella della mod (SENZA `path =`)
2. `custom_buildings.mod` — nella cartella `mod/` dell'utente (CON `path =`)

```
name = "Custom Buildings - Industrial & Social"
path = "mod/custom_buildings"            # SOLO nel file utente
version = "1.0.0"
supported_version = "1.15.*"
tags = {
    "Gameplay"
    "Buildings"
}
```

Se si aggiungono `replace_path`, vanno in ENTRAMBI i file.

### Sprite / Icone (interface/*.gfx)

Per aggiungere icone custom, creare `interface/custom_buildings.gfx`:

```
spriteTypes = {
    spriteType = {
        name = "GFX_buildings_strip"
        textureFile = "gfx/interface/buildings/building_icon_strip.dds"
        noOfFrames = 21      # Vanilla = 16, noi aggiungiamo frame 17-21
    }
}
```

Le icone sono 46x46 pixel ciascuna, allineate orizzontalmente in un'unica strip DDS.

### Debug

Avviare il gioco con `-debug` nelle opzioni di lancio Steam per:
- Ricaricare file modificati senza riavviare
- Vedere error.log automaticamente
- Accedere al Nudger dal menu principale
- Vedere ID province/stati al passaggio del mouse

Error log in: `Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log`

---

## Idee per espansioni future (dal modder)

Il modder ha menzionato questi edifici come punto di partenza. Possibili aggiunte:
- Più livelli di edifici esistenti sbloccabili via tech
- Edifici militari (campo addestramento, deposito armi, centro comando)
- Edifici di intelligence (centro spionaggio, stazione radio)
- Sistema di eventi legati alla presenza di certi edifici
- Decisioni per costruzione rapida o conversione edifici
- Focus tree dedicato alla "politica industriale"
- Effetti negativi a catena (es. troppi campi di lavoro = evento rivolta)

---

## Checklist per aggiungere un nuovo edificio

1. [ ] Aggiungere definizione in `common/buildings/zzz_custom_buildings.txt`
2. [ ] Aggiungere localizzazione in `localisation/english/custom_buildings_l_english.yml`
3. [ ] Aggiungere localizzazione in `localisation/italian/custom_buildings_l_italian.yml`
4. [ ] Verificare che il file .yml abbia il BOM (UTF-8-BOM)
5. [ ] Assegnare un `icon_frame` non in conflitto con altri edifici
6. [ ] Aggiornare `noOfFrames` nel file GFX se si usano frame nuovi
7. [ ] Aggiornare questo `claude.md` con i dettagli del nuovo edificio
8. [ ] Testare con `-debug` e verificare error.log
9. [ ] Aggiornare `README.md` e `version` nei descriptor

---

## Convenzioni di questo progetto

- **Un file per tipo**: tutti i buildings nello stesso file, tutta la loc EN in un file, ecc.
- **Commenti abbondanti**: ogni sezione separata con header `# ====`
- **Prefix `zzz_`**: per file che devono essere valutati dopo vanilla
- **Prefix `custom_buildings.`**: per event ID, per evitare collisioni
- **Versione**: aggiornare in entrambi i `.mod` e nel `README.md`
- **Testare sempre** con `-debug` prima di rilasciare

# Custom Buildings Mod - Industrial & Social
## Hearts of Iron IV

---

## Contenuto della Mod

Questa mod aggiunge **5 nuovi edifici** al gioco, tutti con slot propri (non-shared),
disponibili da subito senza tecnologie necessarie.

### 1. Fabbrica Pezzi Navali (`naval_parts_factory`)
| Proprietà | Valore |
|---|---|
| Costo base | 9000 IC |
| Costo extra/livello | +1500 IC |
| Max livelli | 3 per stato |
| Tipo slot | Non-shared |

**Bonus per livello:**
- +5% velocità produzione navi (stato)
- +5% output fabbriche militari (stato)
- +3% velocità costruzione cantieri navali (paese)
- -2% costo design navale (paese)

---

### 2. Fabbrica Macchinari Pesanti (`heavy_machinery_factory`)
| Proprietà | Valore |
|---|---|
| Costo base | 8000 IC |
| Costo extra/livello | +1200 IC |
| Max livelli | 3 per stato |
| Tipo slot | Non-shared |

**Bonus per livello:**
- +10% slot costruzione (stato)
- +2% velocità costruzione globale (paese)
- +5% velocità riparazione (paese)

---

### 3. Campo di Lavori Forzati (`forced_labor_camp`)
| Proprietà | Valore |
|---|---|
| Costo base | 5000 IC |
| Costo extra/livello | +800 IC |
| Max livelli | 2 per stato |
| Tipo slot | Non-shared |

**Bonus per livello:**
- +5% output fabbriche (stato)
- +3% guadagno efficienza fabbrica (paese)
- -1% beni di consumo necessari (paese)

**Malus per livello:**
- -15% manpower reclutabile (stato)
- +5% resistenza locale (stato)

---

### 4. Centro di Accoglienza (`refugee_center`)
| Proprietà | Valore |
|---|---|
| Costo base | 6000 IC |
| Costo extra/livello | +1000 IC |
| Max livelli | 2 per stato |
| Tipo slot | Non-shared |

**Bonus per livello:**
- +5% crescita popolazione mensile (stato)
- +10% manpower locale (stato)
- +2% manpower da stati non-core (stato)

**Malus per livello:**
- +2% beni di consumo necessari (paese)
- -1% stabilità (paese)

---

### 5. Centro di Ricerca (`research_center`)
| Proprietà | Valore |
|---|---|
| Costo base | 12000 IC |
| Costo extra/livello | +3000 IC |
| Max livelli | 2 per stato |
| Tipo slot | Non-shared |

**Bonus per livello:**
- +3% velocità di ricerca (paese)

---

## Installazione

### Metodo 1 - Manuale
1. Copia la cartella `custom_buildings/` dentro:
   - **Windows:** `C:\Users\<utente>\Documents\Paradox Interactive\Hearts of Iron IV\mod\`
   - **Linux:** `~/.local/share/Paradox Interactive/Hearts of Iron IV/mod/`
   - **Mac:** `~/Documents/Paradox Interactive/Hearts of Iron IV/mod/`
2. Copia il file `custom_buildings.mod` nella stessa cartella `mod/`
3. Avvia il launcher e attiva la mod

### Struttura file della mod
```
mod/
├── custom_buildings.mod                              ← Descriptor utente
└── custom_buildings/
    ├── descriptor.mod                                ← Descriptor mod
    ├── common/
    │   └── buildings/
    │       └── zzz_custom_buildings.txt              ← Definizioni edifici
    └── localisation/
        ├── english/
        │   └── custom_buildings_l_english.yml        ← Testi inglese
        └── italian/
            └── custom_buildings_l_italian.yml        ← Testi italiano
```

---

## Icone degli edifici (DA FARE)

I file di codice usano `icon_frame` da 17 a 21. Devi aggiornare lo sprite strip
delle icone edifici:

1. Apri `gfx/interface/buildings/building_icon_strip.dds` dal gioco vanilla
2. Espandi l'immagine aggiungendo 5 nuove icone da 46x46 pixel a destra
3. Aggiorna `noOfFrames` nel file GFX:

```
# interface/custom_buildings.gfx
spriteTypes = {
    spriteType = {
        name = "GFX_buildings_strip"
        textureFile = "gfx/interface/buildings/building_icon_strip.dds"
        noOfFrames = 21    # Vanilla è 16, aggiungiamo 5 frame
    }
}
```

**NOTA:** Se non vuoi creare icone custom, puoi cambiare i valori `icon_frame`
nel file buildings per riutilizzare icone vanilla esistenti (1-16).

---

## Note sul bilanciamento

- Il **Centro di Ricerca** è intenzionalmente costoso (12000 + 3000/livello)
  perché +3% research speed per livello è molto potente se stackato.
  Con 10 stati a livello 2 = +60% research speed.

- Il **Campo di Lavori Forzati** ha un rapporto rischio/beneficio deliberato:
  il -15% recruitable population per livello è significativo.

- Il **Centro di Accoglienza** compensa il bonus manpower con costi in
  consumer goods e stabilità, simulando il peso economico.

- Tutti i valori sono facilmente modificabili nei file .txt.

---

## Compatibilità
- Versione gioco: 1.15.x
- Non sovrascrive file vanilla (aggiunge solo file nuovi)
- Compatibile con la maggior parte delle mod
- Cambia il checksum (no achievements, no multiplayer vanilla)

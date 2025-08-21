# 🎬 Kino-Schichtplaner (OR-Tools)

Dieses Projekt erstellt automatisch einen **fairen Schichtplan** für ein Kino (Mi–So, 14/17/20 Uhr) mithilfe von **Google OR-Tools (CP-SAT)**.  
Es berücksichtigt **Pflichtbesetzungen, Verfügbarkeiten, Urlaub, Präferenzen und Fairness** und gibt einen vollständigen Plan mit Statistiken aus.

---

## 🚀 Features

- Automatische **Dienstplan-Erstellung** mit OR-Tools
- **Pflichtbesetzung** je Tag/Slot (Normal & Ferienwochen)
- **Urlaub & Unverfügbarkeiten** (harte Sperren)
- **Wunschschichten** (weiche Präferenzen)
- **Wunschpaare** inkl. Min/Max-Prozentband und Bonus
- **Fairness-Regeln** (Lastverteilung, Slot-Typ-Fairness)
- **Erholungsregeln** (Back-to-back-Einsätze verhindern)
- **Wochen- & Tageslimits** (max. 2 Einsätze pro Woche/Tag)
- **Audit & Statistiken**: prüft Pflichtdeckung, zeigt Verteilungen

---

## 📦 Installation

```bash
git clone https://github.com/<dein-user>/kino-schichtplaner.git
cd kino-schichtplaner
pip install -r requirements.txt
```

## 📊 Architektur & Datenfluss

```mermaid
flowchart TD
    subgraph ARCHITEKTUR["Kino-Schichtplaner – Architektur & Datenfluss"]
      A[Konfiguration<br/>(Personal, Urlaub, Pflicht, Wünsche, Paare, Frequenz, Tuning)] --> B
      B[Modellierung (CP-SAT)<br/>Binärvariablen, Paar-Variablen, Loads] --> C{Harte Regeln}
      B --> D((Zielfunktion))

      C --> C1[Pflichtdeckung exakt]
      C --> C2[Urlaub & Unverfügbarkeiten]
      C --> C3[Inkompatibilitäts-Paare]
      C --> C4[Tagesregeln max 2/Tag]
      C --> C5[Back-to-back-Verbot Mi+Do/Do+Fr]
      C --> C6[Erholung Fr20→Sa14, Sa20→So14]
      C --> C7[Wochenende: 14==17 nur wenn gebraucht]
      C --> C8[Wochenlimit max 2/Woche]
      C --> C9[Pro-rata Fairness (Urlaub)]
      C --> C10[Wunschpaar-Prozente MIN/MAX]
      C --> C11[Frequenz-Regeln Fensterweise]

      D --> D1[+ W_WISH · Wünsche]
      D --> D2[- W_SPREAD · Spread]
      D --> D3[- W_SLOTDEV · Slot-Abweichungen]
      D --> D4[+ W_PAIR · Paare]

      subgraph LOOP["Solver-Loop mit Tuning"]
        L1[Attempt 0..N<br/>spread_caps, slot_cap_add_list variieren] --> L2[Seeds 1..k<br/>random_seed wechseln]
        L2 --> L3{FEASIBLE/OPTIMAL?}
        L3 -- nein --> L1
      end

      C & D --> LOOP
      LOOP -->|ja| E[Plan & Berichte]
      E --> O1[Finaler Plan]
      E --> O2[Audit Pflichtdeckung]
      E --> O3[Verteilung gesamt]
      E --> O4[Verteilung je Slottyp]
      E --> O5[Wunschpaare-Statistik]
    end

# Texture Generator — Design Document

## Descrizione

Applicazione web single-page per la generazione automatica di texture geometriche basate su una griglia di quadrati con connessioni angolari. Sviluppata come progetto per il workshop universitario di Rocco Modugno.

## Regole di generazione

1. **Griglia**: N×N quadrati separati equidistantemente
2. **Angoli attivi**: ogni angolo può essere attivo o inattivo; se attivo genera 1-10 linee verso i 2 angoli più vicini del quadrato adiacente nella direzione esterna
3. **Connessioni**: ogni angolo attivo si collega ai 2 angoli più vicini del quadrato più vicino (laterale o diagonale) nella sua direzione
4. **Vincolo orizzontale**: il numero di angoli attivi (0-4) di un quadrato deve essere diverso da quello dei quadrati adiacenti a sinistra e destra
5. **Bordi**: i quadrati sul bordo non hanno wrap — si collegano solo verso i quadrati interni esistenti

## Parametri UI

| Parametro            | Tipo            | Default | Range   |
| -------------------- | --------------- | ------- | ------- |
| Griglia N×N          | input number ×2 | 5       | 2-20    |
| Dimensione quadrato  | slider          | 40px    | 10-100  |
| Spaziatura (× lato)  | slider          | 0.5     | 0.1-2.0 |
| Linee min            | slider          | 1       | 1-10    |
| Linee max            | slider          | 3       | 1-10    |
| Quadrati pieni/vuoti | toggle          | vuoti   | —       |

## Colori

- Sfondo: bianco (#FFFFFF)
- Linee: nero (#000000)
- Quadrati: nero (#000000) se pieni, solo contorno se vuoti

## Esportazione

- **PNG**: export dal Canvas 2D
- **SVG**: generazione seriale del DOM SVG equivalente

## Tecnologia

Singolo file HTML standalone. CSS inline, JS vanilla. Zero dipendenze.

## Layout UI

```
┌─────────────────────────────────────────────┐
│  TEXTURE GENERATOR                          │
├─────────────────────┬───────────────────────┤
│                     │  Griglia: [5] x [5]  │
│    [  PREVIEW  ]    │  Dim. quadrato: [===] │
│    [   CANVAS  ]    │  Spaziatura: [===]   │
│                     │  Linee min: [===]    │
│                     │  Linee max: [===]    │
│                     │  Pieni / Vuoti [▼]  │
│                     │                      │
│                     │  [Genera] [PNG][SVG] │
└─────────────────────────────────────────────┘
```

## Algoritmo

1. Genera matrice N×N di quadrati, ognuno con 4 angoli (TL, TR, BL, BR)
2. Assegna a ogni quadrato un numero `k` (0-4) di angoli attivi, rispettando il vincolo orizzontale
3. Per ogni angolo attivo, determina il quadrato adiacente nella direzione esterna e connettiti ai suoi 2 angoli più vicini
4. Assegna a ogni connessione una molteplicità random tra `linee_min` e `linee_max`
5. Renderizza su Canvas 2D (per preview e PNG) e generazione SVG

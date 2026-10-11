# SQL en mode *filtre i ordre*: `WHERE` + `ORDER BY` + `LIMIT`

Ara que domineu `SELECT` i `FROM`, toca aprendre a:
- **Filtrar** files amb `WHERE`
- **Ordenar** resultats amb `ORDER BY`
- **Restringir** quantes files surten amb `LIMIT` (i `OFFSET`)

---

## 1) `WHERE`
Serveix per **escollir** quines files passen el tall.

Operadors que pots usar (sense funcions):
- Comparació: `=`, `<>` o `!=`, `<`, `<=`, `>`, `>=`
- Rangs: `BETWEEN a AND b` *(inclou els extrems)*
- Conjunt: `IN (valor1, valor2, ...)` · Negació: `NOT IN (...)`
- Nulls: `IS NULL`, `IS NOT NULL`
- Patrons de text: `LIKE` amb `%` (qualsevol seqüència) i `_` (un sol caràcter)
- Lògica: `AND`, `OR`, `NOT` · Parèntesis per prioritat

**Exemples (BD Pokémon)**
```sql
-- Tots els Pokémon de nivell 50
SELECT nom, nivell
FROM pokemon
WHERE nivell = 50;
```

| nom       | nivell |
| --------- | -----: |
| Gengar    |     50 |
| Alakazam  |     50 |
| Snorlax   |     50 |
| Gyarados  |     50 |
| Arcanine  |     50 |
| Dragonite |     50 |

(6 rows)

```sql
-- Nivell entre 20 i 40 (inclosos)
SELECT nom, nivell
FROM pokemon
WHERE nivell BETWEEN 20 AND 40;
```

| nom        | nivell |
| ---------- | -----: |
| Ivysaur    |     24 |
| Wartortle  |     26 |
| Charmeleon |     28 |
| Raichu     |     30 |
| Gardevoir  |     32 |
| Pikachu    |     35 |
| Machoke    |     36 |
| Ninetales  |     38 |
| Poliwrath  |     39 |
| Cloyster   |     40 |

(10 rows)

```sql
-- Pokémon de Kanto o Johto
SELECT nom, regio
FROM pokemon
WHERE regio IN ('Kanto', 'Johto');
```

| nom        | regio |
| ---------- | ----- |
| Bulbasaur  | Kanto |
| Charmander | Kanto |
| Squirtle   | Kanto |
| Pikachu    | Kanto |
| Eevee      | Kanto |
| Chikorita  | Johto |
| Cyndaquil  | Johto |
| Totodile   | Johto |
| Togetic    | Johto |
| Ampharos   | Johto |

(10 rows)

```sql
-- Noms que comencen per 'Pi'
SELECT nom
FROM pokemon
WHERE nom LIKE 'Pi%';
```

| nom       |
| --------- |
| Pikachu   |
| Pichu     |
| Pidgey    |
| Pidgeotto |
| Pidgeot   |
| Piplup    |
| Pinsir    |
| Pineco    |
| Pignite   |
| Piloswine |

(10 rows)

```sql
-- Sense entrenador assignat
SELECT nom
FROM pokemon
WHERE id_entrenador IS NULL;
```

| nom        |
| ---------- |
| Onix       |
| Jigglypuff |
| Ditto      |
| Lapras     |
| Snorlax    |
| Dratini    |
| Abra       |
| Magnemite  |
| Cubone     |

(9 rows)
```sql
-- Combinat: prioritat amb parèntesis
SELECT nom, nivell, regio
FROM pokemon
WHERE (regio = 'Kanto' OR regio = 'Johto') AND nivell >= 30;
```

| nom        | nivell | regio |
| ---------- | -----: | ----- |
| Charizard  |     78 | Kanto |
| Blastoise  |     71 | Kanto |
| Venusaur   |     45 | Kanto |
| Raichu     |     30 | Kanto |
| Ninetales  |     38 | Kanto |
| Typhlosion |     60 | Johto |
| Meganium   |     55 | Johto |
| Feraligatr |     58 | Johto |
| Ampharos   |     34 | Johto |
| Skarmory   |     33 | Johto |

(10 rows)

> **Truc**: `AND` s’avalua abans que `OR`. Si tens dubtes, **posa parèntesis**.

> Cada nova condició ha d'incloure la columna i què es compara, si fas ```(regio = 'Kanto' OR 'Johto')``` donarà error de sintaxi.

---

## 2) `ORDER BY`
Ordena el resultat **després** del `WHERE`.

Sintaxi bàsica:
```sql
SELECT ...
FROM ...
WHERE ...
ORDER BY columna1 [ASC|DESC], columna2 [ASC|DESC];
```

- `ASC` = ascendent (per defecte) · `DESC` = descendent
- Pots ordenar per **més d’una** columna (clau secundària, terciària…).
- Pots ordenar per **àlies** definides al `SELECT`.
- També es pot ordenar per **posició** (`ORDER BY 2`) però no és gaire recomanable.

**Exemples**
```sql
-- Més febles primer
SELECT nom, nivell
FROM pokemon
ORDER BY nivell ASC;
```

| nom        | nivell |
| ---------- | -----: |
| Charmander |      8 |
| Squirtle   |     10 |
| Bulbasaur  |     12 |
| Ivysaur    |     24 |
| Wartortle  |     26 |
| Charmeleon |     28 |
| Raichu     |     30 |
| Pikachu    |     35 |
| Blastoise  |     71 |
| Charizard  |     78 |

(10 rows)

```sql
-- Els més forts al principi i, en empat, ordre alfabètic
SELECT nom, nivell
FROM pokemon
ORDER BY nivell DESC, nom ASC;
```

| nom        | nivell |
| ---------- | -----: |
| Charizard  |     78 |
| Blastoise  |     71 |
| Venusaur   |     45 |
| Pikachu    |     35 |
| Raichu     |     30 |
| Charmeleon |     28 |
| Wartortle  |     26 |
| Ivysaur    |     24 |
| Bulbasaur  |     12 |
| Squirtle   |     10 |
| Charmander |      8 |

(10 rows)

```sql
-- Ordenar per àlies
SELECT nom, (nivell + 1) AS nivell_proper
FROM pokemon
WHERE nivell >= 10
ORDER BY nivell_proper DESC, nom;
```

| nom        | nivell_proper |
| ---------- | ------------: |
| Charizard  |            79 |
| Blastoise  |            72 |
| Venusaur   |            46 |
| Pikachu    |            36 |
| Raichu     |            31 |
| Charmeleon |            29 |
| Wartortle  |            27 |
| Ivysaur    |            25 |
| Bulbasaur  |            13 |
| Squirtle   |            11 |

(10 rows)

> L’ordre afecta **només** la presentació del resultat; no canvia la BD.

---

## 3) `LIMIT` (i `OFFSET`)
Restringeix **quantes files** mostra la consulta. Ideal per “primeres N”.

> **Compatibilitat ràpida**: `LIMIT` és típic de **PostgreSQL/MySQL/SQLite**. En SQL Server s’usa `TOP` i a l’estàndard modern hi ha `FETCH FIRST`. Mantindrem `LIMIT` per simplicitat.

**Exemples**
```sql
-- Les 5 primeres files
SELECT nom, nivell
FROM pokemon
ORDER BY nivell DESC
LIMIT 5;
```

| nom       | nivell |
| --------- | -----: |
| Charizard |     78 |
| Blastoise |     71 |
| Venusaur  |     45 |
| Pikachu   |     35 |
| Raichu    |     30 |

(5 rows)

```sql
-- Saltar-ne 5 i agafar-ne 5 (paginació)
SELECT nom, nivell
FROM pokemon
ORDER BY nivell DESC
LIMIT 5 OFFSET 5;
```

| nom        | nivell |
| ---------- | -----: |
| Charmeleon |     28 |
| Wartortle  |     26 |
| Ivysaur    |     24 |
| Bulbasaur  |     12 |
| Squirtle   |     10 |

(5 rows)

> Normalment es combina `ORDER BY` + `LIMIT` per a resultats **deterministes** (si no ordenes, el “top N” pot variar).

---

## 4) Combo bàsic
```sql
SELECT nom, nivell, regio
FROM pokemon
WHERE regio = 'Kanto' AND nivell >= 30
ORDER BY nivell DESC, nom
LIMIT 10;
```

| nom       | nivell | regio |
| --------- | -----: | ----- |
| Charizard |     78 | Kanto |
| Blastoise |     71 | Kanto |
| Venusaur  |     45 | Kanto |
| Ninetales |     38 | Kanto |
| Pikachu   |     35 | Kanto |
| Raichu    |     30 | Kanto |

(6 rows)

---

## 5) Errors típics
- Escriure `WHERE` **després** d’`ORDER BY`
- Usar `=` amb `NULL` (cal `IS NULL` / `IS NOT NULL`)
- Oblidar-te dels **parèntesis** en expressions amb `AND` + `OR`
- Fer `LIMIT` sense `ORDER BY` quan vols “els *més grans/recents*”

---

## XSheet (resum del resum)
- `WHERE` = **filtra** files (comparació, rangs, conjunts, patrons, nulls, lògica)
- `ORDER BY` = **endreça** el resultat (ASC/DESC, múltiples columnes, àlies)
- `LIMIT`/`OFFSET` = **dosifica** files (paginació)
- Posa **parèntesis** si combines `AND` i `OR`
- Per resultats reproductibles: **`ORDER BY` + `LIMIT`**

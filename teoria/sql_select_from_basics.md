# 🎮 SQL Bàsic (modo “zen”): `SELECT` + `FROM` + àlies

Aquesta és la **xuleta minimalista** per començar a moure’t per SQL. Només el nucli: **què vull** (`SELECT`) i **d’on** ho trec (`FROM`). I uns **àlies** per fer-ho més maco.

---

## 1) `SELECT` — Què vols veure?

```sql
SELECT columna1, columna2, ...
FROM taula;
```

- Pots llistar **una o moltes** columnes.
- Pots posar **valors literals** (textos o números “de pega”) per proves.

**Exemples**
```sql
-- Només una columna
SELECT nom
FROM pokemon;
```

| nom        |
|------------|
| Bulbasaur  |
| Ivysaur    |
| Venusaur   |
| Charmander |
| Charmeleon |
| Charizard  |
| Squirtle   |
| Wartortle  |
| Blastoise  |
| Pikachu    |

(10 rows)

```sql
-- Múltiples columnes
SELECT id_pokemon, nom, nivell
FROM pokemon;
```

|id_pokemon |    nom     | nivell|
|----------:|------------|------:|
|         1 | Bulbasaur  |     12|
|         2 | Ivysaur    |     24|
|         3 | Venusaur   |     45|
|         4 | Charmander |      8|
|         5 | Charmeleon |     28|
|         6 | Charizard  |     78|
|         7 | Squirtle   |     10|
|         8 | Wartortle  |     26|
|         9 | Blastoise  |     71|
|        25 | Pikachu    |     35|

(10 rows)

```sql
-- Expressions i literals
SELECT nom, nivell + 1 AS nivell_proper, 'Kanto' AS regio_defecte
FROM pokemon;
```
|    nom      | nivell_proper  | regio_defecte |
|-------------|---------------:|---------------|
|Bulbasaur    |             13 | Kanto         |
|Ivysaur      |             25 | Kanto         |
|Venusaur     |             46 | Kanto         |
|Charmander   |              9 | Kanto         |
|Charmeleon   |             29 | Kanto         |
|Charizard    |             79 | Kanto         |
|Squirtle     |             11 | Kanto         |
|Wartortle    |             27 | Kanto         |
|Blastoise    |             72 | Kanto         |
|Pikachu      |             36 | Kanto         |

(10 rows)

```sql
-- El clàssic “dona-m’ho tot”
SELECT *
FROM pokemon;
```
| id_pokemon |    nom     | nivell | regio | id_entrenador |
|-----------:|------------|-------:|-------|--------------:|
|          1 | Bulbasaur  |     12 | Kanto |            101|
|          2 | Ivysaur    |     24 | Kanto |            101|
|          3 | Venusaur   |     45 | Kanto |            101|
|          4 | Charmander |      8 | Kanto |            102|
|          5 | Charmeleon |     28 | Kanto |            102|
|          6 | Charizard  |     78 | Kanto |            102|
|          7 | Squirtle   |     10 | Kanto |            103|
|          8 | Wartortle  |     26 | Kanto |            103|
|          9 | Blastoise  |     71 | Kanto |            103|
|         25 | Pikachu    |     35 | Kanto |           NULL|

(10 rows)


---

## 2) `FROM` — D’on ho vols?

```sql
SELECT ...
FROM taula;
```

- Indica **la taula** (o vista) origen de les dades.
- **Pots** fer servir **àlies de taula** (va molt bé!).

**Exemples**
```sql
-- Àlies de taula (curtets i elegants)
SELECT p.nom, p.nivell
FROM pokemon AS p;

-- L’AS és opcional
SELECT p.nom, p.nivell
FROM pokemon p;
```

> 💡 L’àlies de taula és útil per **abreujar** (`pokemon` → `p`) i per **qualificar** columnes quan tens noms semblants.

---

## 3) Àlies de columna — posa noms “humans”

Dóna un **nom bonic** a allò que surt al resultat.  
Això **no canvia** la base de dades, només el **capçal** del resultat.

```sql
-- Amb AS
SELECT nom AS nom_pokemon, nivell AS lvl
FROM pokemon;

-- Sense AS (curt i comú)
SELECT nom nom_pokemon, nivell lvl
FROM pokemon;
```
| nom_pokemon | lvl |
|-------------|----:|
| Bulbasaur   |  12 |
| Ivysaur     |  24 |
| Venusaur    |  45 |
| Charmander  |   8 |
| Charmeleon  |  28 |
| Charizard   |  78 |
| Squirtle    |  10 |
| Wartortle   |  26 |
| Blastoise   |  71 |
| Pikachu     |  35 |

(10 rows)


---

## 4) `DISTINCT` — Sense duplicats

Treure **valors únics** d’una o més columnes.

```sql
-- Tipus únics
SELECT DISTINCT tipus
FROM tipus_pokemon;
```
|    tipus   |
|------------|
| Normal     |
| Foc        |
| Aigua      |
| Planta     |
| Elèctric   |
| Gel        |
| Roca       |
| Terra      |
| Volador    |
| Psíquic    |
| Fantasma   |
| Drac       |

(12 rows)

```sql
-- Parelles úniques
SELECT DISTINCT regio, ciutat
FROM gimnas;
```
|  regio   |   ciutat  |
|----------|-----------|
| Kanto    | Pewter    |
| Kanto    | Cerulean  |
| Kanto    | Vermilion |
| Kanto    | Celadon   |
| Johto    | Goldenrod |
| Johto    | Ecruteak  |
| Hoenn    | Petalburg |
| Hoenn    | Mauville  |
| Sinnoh   | Oreburgh  |
| Sinnoh   | Hearthome |
| Unova    | Nimbasa   |
| Galar    | Motostoke |

(12 rows)

> ⚠️ Recorda que `DISTINCT` s’aplica a la **combinació** de columnes llistades.

---

## 5) Trucs ràpids de pro (sense sortir del tema)

- **Ordre de les columnes**: el resultat **respecta** l’ordre que poses al `SELECT`.  
- **Duplicar una columna amb nom diferent** (per comparacions visuals):  
```sql
  SELECT nom, nom AS nom_copia
  FROM pokemon;
```
|     nom      |  nom_copia  |
|--------------|-------------|
| Bulbasaur    | Bulbasaur   |
| Ivysaur      | Ivysaur     |
| Venusaur     | Venusaur    |
| Charmander   | Charmander  |
| Charmeleon   | Charmeleon  |
| Charizard    | Charizard   |
| Squirtle     | Squirtle    |
| Wartortle    | Wartortle   |
| Blastoise    | Blastoise   |
| Pikachu      | Pikachu     |

(10 rows)


- **Comentar la teva query** (el profe t’estimarà més):
  ```sql
  -- Aquest SELECT mostra només noms i nivells
  SELECT nom, nivell
  FROM pokemon; 
  ```

---

## XSheet (resum del resum)

- `SELECT` **què** veus · `FROM` **d’on** ho treus  
- **Àlies** de taula i columna (`AS` opcional)  
- `DISTINCT` per **valors únics**  

# JOINS en SQL
A l’hora d’agafar dades de **més d’una taula**, els **JOIN** són la solució.
Aquí expliquem els imprescindibles: **INNER (JOIN), LEFT, RIGHT** i fem una passada ràpida pels altres.

> **Esquema que farem servir als exemples**
> - ```pokemon(id_pokemon, nom, nivell, regio, id_entrenador)```
> - ```entrenador(id_entrenador, nom_entrenador)``` *(hi ha un entrenador sense cap Pokémon!)*
>
> Dades (resum):
> - **POKÉMON**: Bulbasaur(101), Ivysaur(101), Venusaur(101), Charmander(102), Charmeleon(102), Charizard(102), Squirtle(103), Wartortle(103), Blastoise(103), Pikachu(NULL)
> - **ENTRENADORS**: 101-Red, 102-Blue, 103-Green, 104-Yellow (sense Pokémon)

---

## Com s’uneixen les taules (molt important)

Sempre que feu un JOIN, **indiqueu per quin camp s’ajunten**:
```sql
... JOIN entrenador e
     ON p.id_entrenador = e.id_entrenador
```

- La **clau forana** està a ```pokemon.id_entrenador```
- La **clau primària** està a ```entrenador.id_entrenador```
- Sense un ```ON``` correcte, el resultat serà **cartesià** (i caòtic).

---

## 1) ```INNER JOIN``` (o ```JOIN``` “normal”): només els que fan parella

Torna **només** les files que **tenen coincidència** a banda i banda.

```sql
SELECT p.nom       AS nom_pokemon,
       p.nivell,
       e.nom_entrenador
FROM pokemon AS p
INNER JOIN entrenador AS e
  ON p.id_entrenador = e.id_entrenador;
```

**Resultat (simulat)** — *(Pikachu queda fora perquè té ```id_entrenador = NULL```)*

| nom_pokemon | nivell | nom_entrenador |
| ----------- | -----: | -------------- |
| Bulbasaur   |     12 | Red            |
| Ivysaur     |     24 | Red            |
| Venusaur    |     45 | Red            |
| Charmander  |      8 | Blue           |
| Charmeleon  |     28 | Blue           |
| Charizard   |     78 | Blue           |
| Squirtle    |     10 | Green          |
| Wartortle   |     26 | Green          |
| Blastoise   |     71 | Green          |

(9 rows)

---

## 2) ```LEFT JOIN```: tots els de l’esquerra (encara que no tinguin parella)

Torna **tots els Pokémon**, i si no tenen entrenador, posa ```NULL```.

```sql
SELECT p.nom       AS nom_pokemon,
       p.nivell,
       e.nom_entrenador
FROM pokemon AS p
LEFT JOIN entrenador AS e
  ON p.id_entrenador = e.id_entrenador;
```

**Resultat (simulat)** — *(ara apareix **Pikachu** amb entrenador ```NULL```)*

| nom_pokemon | nivell | nom_entrenador |
| ----------- | -----: | -------------- |
| Bulbasaur   |     12 | Red            |
| Ivysaur     |     24 | Red            |
| Venusaur    |     45 | Red            |
| Charmander  |      8 | Blue           |
| Charmeleon  |     28 | Blue           |
| Charizard   |     78 | Blue           |
| Squirtle    |     10 | Green          |
| Wartortle   |     26 | Green          |
| Blastoise   |     71 | Green          |
| **Pikachu** |     35 | **NULL**       |

(10 rows)

---

## 3) ```RIGHT JOIN```: tots els de la dreta (encara que no tinguin parella)

Torna **tots els entrenadors**, i si no tenen Pokémon, posa ```NULL`` al costat Pokémon.

```sql
SELECT e.nom_entrenador,
       p.nom   AS nom_pokemon,
       p.nivell
FROM pokemon AS p
RIGHT JOIN entrenador AS e
  ON p.id_entrenador = e.id_entrenador;
```

**Resultat (simulat)** — *(apareix **Yellow** sense cap Pokémon)*

| nom_entrenador | nom_pokemon |   nivell |
| -------------- | ----------- | -------: |
| Red            | Bulbasaur   |       12 |
| Red            | Ivysaur     |       24 |
| Red            | Venusaur    |       45 |
| Blue           | Charmander  |        8 |
| Blue           | Charmeleon  |       28 |
| Blue           | Charizard   |       78 |
| Green          | Squirtle    |       10 |
| Green          | Wartortle   |       26 |
| Green          | Blastoise   |       71 |
| **Yellow**     | **NULL**    | **NULL** |

(10 rows)

> **LEFT vs RIGHT**: tria segons **quina banda vols “completa”**.
> `LEFT JOIN` = conserva **tots els de l’esquerra** (`FROM ...`), `RIGHT JOIN` = **tots els de la dreta**.

---

## 4) Altres `JOIN` (només per cultura general)

- `FULL OUTER JOIN` → No existeix directament a MariaDB.
    - Es pot simular amb `LEFT JOIN` **UNION** `RIGHT JOIN` i eliminant duplicats.
- `CROSS JOIN` → Producte cartesià (cada Pokémon amb cada entrenador). Rarament útil sense un `WHERE`.
- `NATURAL JOIN` → Uneix per **columnes amb el mateix nom** automàticament. Evita’l en projectes seriosos: és fràgil (un canvi de nom i… boom).
- `SELF JOIN` → Una taula es relaciona amb ella mateixa (ex.: arbre d’evolucions).

```sql
-- Exemple conceptual (no el fem servir avui):
SELECT p1.nom AS base, p2.nom AS evolucio
FROM pokemon p1
JOIN pokemon p2
  ON p1.id_pokemon = p2.id_previa;
```

---

## 5) BONUS: JOIN + `WHERE` + `ORDER BY` + `LIMIT`

**“Digue’m els 5 Pokémon més forts amb entrenador, i el nom del seu entrenador”**

```sql
SELECT p.nom       AS nom_pokemon,
       p.nivell,
       e.nom_entrenador
FROM pokemon AS p
JOIN entrenador AS e
  ON p.id_entrenador = e.id_entrenador
WHERE p.nivell >= 20               -- filtre
ORDER BY p.nivell DESC, p.nom ASC  -- ordre consistent
LIMIT 5;                           -- n resultats
```

**Resultat (simulat)**

| nom_pokemon | nivell | nom_entrenador |
| ----------- | -----: | -------------- |
| Charizard   |     78 | Blue           |
| Blastoise   |     71 | Green          |
| Venusaur    |     45 | Red            |
| Charmeleon  |     28 | Blue           |
| Wartortle   |     26 | Green          |

(5 rows)

---

## Checklist d’examen (JOINS)

- [ ] He posat el `ON` **correcte** (`p.id_entrenador = e.id_entrenador`).
- [ ] Sé què **conserva** cada JOIN:
    - `INNER` → només parelles que **matxegen**
    - `LEFT` → tots els de l’**esquerra** (+ dreta si matxaquen)
    - `RIGHT` → tots els de la **dreta** (+ esquerra si matxaquen)
- [ ] A MariaDB **no hi ha** `FULL OUTER JOIN` nadiu.
- [ ] Puc afegir `WHERE`, `ORDER BY`, `LIMIT` a gust per refinar el resultat.
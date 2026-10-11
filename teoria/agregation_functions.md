# **Funcions d’Agregació en SQL**

Les **funcions d’agregació** en SQL són per resumir dades: et permeten comptar, sumar, calcular mitjanes, trobar màxims i mínims, i molt més. Quan les combines amb **GROUP BY** i operacions aritmètiques o funcions a columnes, pots fer anàlisis complets.

> **Esquema**  
> - ```pokemon(id_pokemon, nom, nivell, regio, id_entrenador, capturat, experiencia)```  
> - ```entrenador(id_entrenador, nom_entrenador, data_naixement, pokedollars)```  
> 
> **Dades (resum)**:  
> - **Pokémon**: Bulbasaur (nivell 12, id_entrenador 101, capturat: 2025-01-15, experiencia: 1000), Ivysaur (nivell 24, id_entrenador 101, experiencia: 2000), Venusaur (nivell 45, id_entrenador 101, experiencia: 4500), Charmander (nivell 8, id_entrenador 102, experiencia: 800), Charmeleon (nivell 28, id_entrenador 102, experiencia: 2800), Charizard (nivell 78, id_entrenador 102, experiencia: 7800), Squirtle (nivell 10, id_entrenador 103, experiencia: 900), Wartortle (nivell 26, id_entrenador 103, experiencia: 2600), Blastoise (nivell 71, id_entrenador 103, experiencia: 7100), Pikachu (nivell 35, id_entrenador NULL, experiencia: 3500).  
> - **Entrenadors**: Red (101, 1995-05-10, pokedollars: 5000), Blue (102, 1996-08-22, pokedollars: 8000), Green (103, 1998-12-01, pokedollars: 6000), Yellow (104, pokedollars: 2000, sense Pokémon).  
> 
> **Nota**: Avui és **19 d’octubre de 2025, 12:12 PM CEST**.

---

## **Què són les funcions d’agregació?**

Les funcions d’agregació processen un conjunt de files i retornen **un sol valor** per grup (o per tot el conjunt si no hi ha ```GROUP BY```). Les més comunes són:

- ```COUNT```: Compta files o valors no nuls.
- ```SUM```: Suma valors numèrics.
- ```AVG```: Calcula la mitjana.
- ```MAX```: Troba el valor màxim.
- ```MIN```: Troba el valor mínim.

**Important**: 
- S’usen sovint amb **GROUP BY** per analitzar grups.
- Ignoren els valors **NULL** (excepte en algun cas amb ```COUNT```).

---

## 1) **COUNT: Compteig**

Compta files o valors no nuls en una columna.

### Exemple: Quants Pokémon té cada entrenador?

```sql
SELECT e.nom_entrenador,
       COUNT(p.id_pokemon) AS total_pokemon
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:

| nom_entrenador | total_pokemon |
|----------------|---------------|
| Red            | 3             |
| Blue           | 3             |
| Green          | 3             |
| Yellow         | 0             |

**Variants**:
- ```COUNT(p.id_pokemon)```: Compta només files amb ```id_pokemon``` no nul.
- ```COUNT(*)```: Compta **totes** les files del grup, incloses les amb nuls.

**Error Típic 1: ```COUNT(*)``` vs ```COUNT(columna)```**
```sql
SELECT e.nom_entrenador, COUNT(*) AS total
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat (erroni)**:

| nom_entrenador | total |
|----------------|-------|
| Red            | 3     |
| Blue           | 3     |
| Green          | 3     |
| Yellow         | 1     | -- Error! Yellow no té Pokémon, però compte 1 fila

**Solució**: Usa ```COUNT(columna)``` (ex.: ```COUNT(p.id_pokemon)```) quan vols comptar només valors no nuls.

---

## 2) **SUM: Suma**

Suma valors numèrics d’una columna.

### Exemple: Quin és el total d’experiència dels Pokémon de cada entrenador?

```sql
SELECT e.nom_entrenador,
       SUM(p.experiencia) AS experiencia_total
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:

| nom_entrenador | experiencia_total |
|----------------|------------------|
| Red            | 7500             | -- (1000 + 2000 + 4500)
| Blue           | 11400            | -- (800 + 2800 + 7800)
| Green          | 10600            | -- (900 + 2600 + 7100)
| Yellow         | NULL             | -- Sense Pokémon

**Error Típic 2: Usar SUM amb dades no numèriques**
```sql
SELECT e.nom_entrenador, SUM(p.nom) -- Error! nom no és numèric
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Solució**: Assegura’t que la columna (ex.: ```p.experiencia```) sigui numèrica (INT, DECIMAL, etc.).

---

## 3) **AVG: Mitjana**

Calcula la mitjana dels valors d’una columna.

### Exemple: Quin és el nivell mitjà dels Pokémon de cada entrenador?

```sql
SELECT e.nom_entrenador,
       AVG(p.nivell) AS nivell_mitja
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:

| nom_entrenador | nivell_mitja |
|----------------|--------------|
| Red            | 27.0         | -- (81 / 3)
| Blue           | 38.0         | -- (114 / 3)
| Green          | 35.7         | -- (107 / 3)
| Yellow         | NULL         | -- Sense Pokémon

**Error Típic 3: Oblidar que AVG ignora NULLs**
Si una columna té valors NULL, ```AVG``` només considera els valors no nuls, cosa que pot donar resultats inesperats.

**Solució**: Usa ```COALESCE``` per gestionar NULLs:
```sql
SELECT e.nom_entrenador,
       AVG(COALESCE(p.nivell, 0)) AS nivell_mitja
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

---

## 4) **MAX i MIN**

- ```MAX```: Retorna el valor més alt.
- ```MIN```: Retorna el valor més baix.

### Exemple: Quin és el nivell màxim i mínim dels Pokémon de cada entrenador?

```sql
SELECT e.nom_entrenador,
       MAX(p.nivell) AS nivell_max,
       MIN(p.nivell) AS nivell_min
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:

| nom_entrenador | nivell_max | nivell_min |
|----------------|------------|------------|
| Red            | 45         | 12         |
| Blue           | 78         | 8          |
| Green          | 71         | 10         |
| Yellow         | NULL       | NULL       |

**Error Típic 4: Assumir que MAX/MIN funcionen igual amb dates o strings**
```MAX``` i ```MIN``` també funcionen amb dates (ex.: ```capturat```) o strings (ex.: ```nom```), però l’ordre pot ser inesperat amb strings (ordre alfabètic).

**Exemple amb dates**:
```sql
SELECT e.nom_entrenador,
       MIN(p.capturat) AS primera_captura
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:

| nom_entrenador | primera_captura     |
|----------------|---------------------|
| Red            | 2025-01-15 10:30:00 |
| Blue           | 2025-03-20 14:00:00 |
| Green          | 2025-01-10 09:00:00 |
| Yellow         | NULL                |

**Solució**: Entén l’ordre natural de la columna (numèric, cronològic, o alfabètic).

---

## 5) **Operacions en Columnes i Ordenacions**

Pots fer **operacions aritmètiques** (+, -, *, /, %) o usar funcions com ```ROUND```, ```FLOOR```, o ```ABS``` tant al ```SELECT``` (per mostrar columnes calculades) com a l’```ORDER BY``` (per ordenar segons càlculs).

### Exemple 1: Calcular el valor total dels Pokémon (nivell * experiència)

Suposem que el “valor” d’un Pokémon és el seu nivell multiplicat per la seva experiència. Mostrem el valor total per entrenador.

```sql
SELECT e.nom_entrenador,
       SUM(p.nivell * p.experiencia) AS valor_total
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:

| nom_entrenador | valor_total |
|----------------|-------------|
| Red            | 139500      | -- (12*1000 + 24*2000 + 45*4500)
| Blue           | 626800      | -- (8*800 + 28*2800 + 78*7800)
| Green          | 524100      | -- (10*900 + 26*2600 + 71*7100)
| Yellow         | NULL        | -- Sense Pokémon

**Nota**: Operacions com ```p.nivell * p.experiencia``` es fan **fila a fila** abans de l’agregació (```SUM```).

---

### Exemple 2: Ordenar per una operació (pokedollars restants després de pagar per Pokémon)

Suposem que cada Pokémon costa 1000 pokedollars per mantenir-lo. Mostrem els entrenadors amb els pokedollars restants (pokedollars - total de Pokémon * 1000), ordenats per aquesta quantitat.

```sql
SELECT e.nom_entrenador,
       e.pokedollars - (COUNT(p.id_pokemon) * 1000) AS pokedollars_restants
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
ORDER BY e.pokedollars - (COUNT(p.id_pokemon) * 1000) DESC;
```

**Resultat (simulat)**:

| nom_entrenador | pokedollars_restants |
|----------------|----------------------|
| Blue           | 5000                 | -- (8000 - 3*1000)
| Green          | 3000                 | -- (6000 - 3*1000)
| Red            | 2000                 | -- (5000 - 3*1000)
| Yellow         | 2000                 | -- (2000 - 0*1000)

**Nota**: L’operació ```e.pokedollars - (COUNT(p.id_pokemon) * 1000)``` es calcula al ```SELECT``` i es repeteix a l’```ORDER BY``` per ordenar.

---

### Exemple 3: Usar ROUND per formatar i operacions complexes

Calculem una “puntuació” per entrenador (mitjana de nivell * número de Pokémon), arrodonida a 2 decimals, i ordenem per aquesta puntuació.

```sql
SELECT e.nom_entrenador,
       ROUND(AVG(p.nivell) * COUNT(p.id_pokemon), 2) AS puntuacio
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
ORDER BY AVG(p.nivell) * COUNT(p.id_pokemon) DESC;
```

**Resultat (simulat)**:

| nom_entrenador | puntuacio |
|----------------|-----------|
| Blue           | 114.00    | -- (38.0 * 3)
| Green          | 107.10    | -- (35.7 * 3)
| Red            | 81.00     | -- (27.0 * 3)
| Yellow         | 0.00      | -- (0 * 0)

**Nota**: ```ROUND(expr, 2)``` arrodoneix a 2 decimals. L’```ORDER BY``` usa la mateixa expressió sense ```ROUND``` per mantenir la precisió en l’ordenació.

---

### Error Típic 5: Operacions amb NULLs
Les operacions amb ```NULL``` donen ```NULL```. Per exemple:
```sql
SELECT e.nom_entrenador,
       e.pokedollars - SUM(p.experiencia) AS resultat
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

Per Yellow, ```SUM(p.experiencia)``` és ```NULL```, així que ```pokedollars - NULL``` també serà ```NULL```.

**Solució**: Usa ```COALESCE```:
```sql
SELECT e.nom_entrenador,
       e.pokedollars - COALESCE(SUM(p.experiencia), 0) AS resultat
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

---

## 6) **Exemple Combinat amb HAVING, JOIN i Operacions**

**“Vull els entrenadors amb Pokémon capturats el 2025, amb el total de Pokémon, el nivell mitjà, el màxim, el mínim, i una puntuació (nivell mitjà * total Pokémon), però només si la puntuació supera 100, ordenats per puntuació.”**

```sql
SELECT e.nom_entrenador,
       COUNT(p.id_pokemon) AS total_pokemon,
       AVG(p.nivell) AS nivell_mitja,
       MAX(p.nivell) AS nivell_max,
       MIN(p.nivell) AS nivell_min,
       ROUND(AVG(p.nivell) * COUNT(p.id_pokemon), 2) AS puntuacio
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
WHERE YEAR(p.capturat) = 2025
GROUP BY e.id_entrenador, e.nom_entrenador
HAVING AVG(p.nivell) * COUNT(p.id_pokemon) > 100
ORDER BY AVG(p.nivell) * COUNT(p.id_pokemon) DESC;
```

**Resultat**:

| nom_entrenador | total_pokemon | nivell_mitja | nivell_max | nivell_min | puntuacio |
|----------------|---------------|--------------|------------|------------|-----------|
| Blue           | 3             | 38.0         | 78         | 8          | 114.00    |
| Green          | 3             | 35.7         | 71         | 10         | 107.10    |

**Explicació**:
- ```WHERE YEAR(p.capturat) = 2025``` filtra Pokémon capturats el 2025.
- ```GROUP BY e.id_entrenador, e.nom_entrenador``` agrupa per entrenador.
- ```HAVING AVG(p.nivell) * COUNT(p.id_pokemon) > 100``` filtra grups amb puntuació > 100.
- La columna ```puntuacio``` usa ```ROUND(AVG(p.nivell) * COUNT(p.id_pokemon), 2)```.
- ```ORDER BY AVG(p.nivell) * COUNT(p.id_pokemon) DESC``` ordena per puntuació.

---

## 7) **Errors Típics i Com Evitar-los**

### **Error 1: No incloure columnes no agregades al GROUP BY**
Si una columna del ```SELECT``` no està en una funció d’agregació, ha d’estar al ```GROUP BY```.

**Malament**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador; -- Error! nom_entrenador no està al GROUP BY
```

**Correcte**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Solució**: Inclou totes les columnes no agregades del ```SELECT``` al ```GROUP BY```.

---

### **Error 2: Confondre WHERE i HAVING**
```WHERE``` filtra files abans de l’agrupació; ```HAVING``` filtra grups després.

**Malament**:
```sql
SELECT e.nom_entrenador, AVG(p.nivell)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
WHERE AVG(p.nivell) > 30; -- Error! No pots usar AVG al WHERE
```

**Correcte**:
```sql
SELECT e.nom_entrenador, AVG(p.nivell)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
HAVING AVG(p.nivell) > 30;
```

**Solució**: Usa ```WHERE``` per columnes crues, ```HAVING``` per agregacions.

---

### **Error 3: No gestionar NULLs**
Les funcions d’agregació ignoren ```NULL```, però en operacions aritmètiques, ```NULL``` pot causar problemes.

**Exemple confós**:
```sql
SELECT e.nom_entrenador, e.pokedollars - SUM(p.experiencia)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

Per Yellow, ```SUM(p.experiencia)``` és ```NULL```, així que el resultat serà ```NULL```.

**Solució**: Usa ```COALESCE```:
```sql
SELECT e.nom_entrenador, e.pokedollars - COALESCE(SUM(p.experiencia), 0)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

---

### **Error 4: No usar el JOIN correcte**
Un ```INNER JOIN``` exclou entrenadors sense Pokémon (com Yellow).

**Malament**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
INNER JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Correcte**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Solució**: Usa ```LEFT JOIN``` quan vulguis incloure tots els registres de la taula principal.

---

### **Error 5: Operacions complexes sense àlies**
Si uses expressions complexes al ```SELECT``` i ```ORDER BY```, repetir-les pot ser confús i propens a errors.

**Malament**:
```sql
SELECT e.nom_entrenador,
       AVG(p.nivell) * COUNT(p.id_pokemon) AS puntuacio
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
ORDER BY AVG(p.nivell) * COUNT(p.id_pokemon) DESC;
```

**Millor** (en algunes versions de SQL, com PostgreSQL):
```sql
SELECT e.nom_entrenador,
       AVG(p.nivell) * COUNT(p.id_pokemon) AS puntuacio
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
ORDER BY puntuacio DESC; -- Usa l’àlies
```

**Nota**: En MariaDB/MySQL, pots usar l’àlies a l’```ORDER BY```, però en altres sistemes com PostgreSQL potser has de repetir l’expressió o usar una subconsulta.

**Solució**: Sigues consistent amb les expressions i considera subconsultes si l’àlies no funciona.

---

## 8) **Checklist d’Examen (Funcions d’Agregació i Operacions)**

- [ ] Usa ```COUNT(columna)``` per comptar valors no nuls; ```COUNT(*)``` per comptar files.
- [ ] Verifica que les columnes per a ```SUM``` i ```AVG``` siguin numèriques.
- [ ] Inclou totes les columnes no agregades del ```SELECT``` al ```GROUP BY```.
- [ ] Gestiona ```NULL``` amb ```COALESCE``` en operacions i agregacions.
- [ ] Combina amb ```WHERE``` (per files) i ```HAVING``` (per grups) correctament.
- [ ] Usa ```LEFT JOIN``` si vols incloure registres sense correspondència.
- [ ] Aplica operacions (+, -, *, /) al ```SELECT``` i ```ORDER BY``` amb cura.
- [ ] Usa ```ROUND``` o altres funcions per formatar resultats numèrics.
- [ ] Prova amb dades petites per assegurar-te que els resultats tenen sentit.
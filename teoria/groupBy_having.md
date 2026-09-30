# **GROUP BY i HAVING en SQL**

Les clàusules **GROUP BY** i **HAVING** són per **agrupar dades** i **filtrar grups** en SQL. Són essencials per fer resums, com comptar quants Pokémon té cada entrenador o trobar els millors gimnasos.

> **Esquema**  
> - ```pokemon(id_pokemon, nom, nivell, regio, id_entrenador, capturat)```  
> - ```entrenador(id_entrenador, nom_entrenador, data_naixement)```  
> 
> **Dades (resum)**:  
> - **Pokémon**: Bulbasaur (nivell 12, id_entrenador 101, capturat: 2025-01-15), Ivysaur (nivell 24, id_entrenador 101), Venusaur (nivell 45, id_entrenador 101), Charmander (nivell 8, id_entrenador 102), Charmeleon (nivell 28, id_entrenador 102), Charizard (nivell 78, id_entrenador 102), Squirtle (nivell 10, id_entrenador 103), Wartortle (nivell 26, id_entrenador 103), Blastoise (nivell 71, id_entrenador 103), Pikachu (nivell 35, id_entrenador NULL).  
> - **Entrenadors**: Red (101, 1995-05-10), Blue (102, 1996-08-22), Green (103, 1998-12-01), Yellow (104, sense Pokémon).  
> 
> **Nota**: Avui és **19 d’octubre de 2025**.

---

## **Què fan GROUP BY i HAVING?**

- **GROUP BY**: Agrupa files amb el **mateix valor** en una o més columnes. És com posar els Pokémon en caixes segons el seu entrenador. Després, pots aplicar funcions com ```COUNT```, ```SUM```, ```AVG```, ```MAX``` o ```MIN``` a cada grup.
- **HAVING**: Filtra **grups** (no files individuals, com fa ```WHERE```). És com dir: “Només vull veure entrenadors amb més de 2 Pokémon.”

**Ordre en una consulta SQL**:
```sql
SELECT ... 
FROM ... 
[WHERE ...] 
GROUP BY ... 
[HAVING ...] 
[ORDER BY ...];
```

---

## 1) **GROUP BY**

**Exemple**: Compta quants Pokémon té cada entrenador.

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

**Per què fem servir LEFT JOIN?**  
Com que Yellow no té Pokémon, un ```INNER JOIN``` l’exclouria. Amb ```LEFT JOIN```, apareix amb un compte de 0.

---

## 2) **HAVING**

**HAVING** s’aplica **després** del ```GROUP BY``` per filtrar els grups segons una condició.

**Exemple**: Mostra només els entrenadors amb més de 2 Pokémon.

```sql
SELECT e.nom_entrenador,
       COUNT(p.id_pokemon) AS total_pokemon
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
HAVING COUNT(p.id_pokemon) > 2;
```

**Resultat**:

| nom_entrenador | total_pokemon |
|----------------|---------------|
| Red            | 3             |
| Blue           | 3             |
| Green          | 3             |

Yellow no apareix perquè té 0 Pokémon (menys de 2).

---

## 3) **Exemple Combinat amb Funcions d’Agregació**

**“Vull saber el nivell mitjà i màxim dels Pokémon de cada entrenador, però només dels entrenadors amb algun Pokémon de nivell superior a 40.”**

```sql
SELECT e.nom_entrenador,
       AVG(p.nivell) AS nivell_mitja,
       MAX(p.nivell) AS nivell_max
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
HAVING MAX(p.nivell) > 40;
```

**Resultat (simulat)**:

| nom_entrenador | nivell_mitja | nivell_max |
|----------------|--------------|------------|
| Red            | 27.0         | 45         |
| Blue           | 38.0         | 78         |
| Green          | 35.7         | 71         |

**Explicació**:
- ```AVG(p.nivell)``` calcula la mitjana dels nivells per entrenador.
- ```MAX(p.nivell)``` troba el nivell més alt.
- ```HAVING MAX(p.nivell) > 40``` filtra els entrenadors amb Pokémon de nivell > 40.
- Yellow no apareix (no té Pokémon), i Pikachu (sense entrenador) tampoc afecta el resultat.

---

## 4) **Errors Típics i Com Evitar-los**

### **Error 1: No incloure columnes no agregades al GROUP BY**  
En SQL (especialment MariaDB/MySQL en mode estricte), **totes** les columnes del ```SELECT``` que no estan en funcions d’agregació (com ```COUNT```, ```AVG```) han d’estar al ```GROUP BY```.

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

**Solució**: Assegura’t que totes les columnes del ```SELECT``` que no siguin agregades (ex.: ```nom_entrenador```) estiguin al ```GROUP BY```.

---

### **Error 2: Confondre WHERE i HAVING**  
- ```WHERE``` filtra **files individuals** abans de l’agrupació.  
- ```HAVING``` filtra **grups** després del ```GROUP BY```.

**Malament**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
WHERE COUNT(p.id_pokemon) > 2; -- Error! No pots usar COUNT al WHERE
```

**Correcte**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador
HAVING COUNT(p.id_pokemon) > 2;
```

**Solució**: Usa ```WHERE``` per condicions sobre columnes crues (ex.: ```p.nivell > 40```) i ```HAVING``` per condicions sobre agregacions (ex.: ```COUNT(p.id_pokemon) > 2```).

---

### **Error 3: Oblidar-se de LEFT JOIN quan vols dades sense parella**  
Si uses ```INNER JOIN```, perds entrenadors sense Pokémon (com Yellow).

**Malament**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
INNER JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:
| nom_entrenador | total_pokemon |
|----------------|---------------|
| Red            | 3             |
| Blue           | 3             |
| Green          | 3             |

Yellow no apareix!

**Correcte**:
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon)
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Solució**: Usa ```LEFT JOIN``` quan vulguis incloure tots els registres de la taula “principal” (aquí, ```entrenador```), fins i tot si no tenen correspondència.

---

### **Error 4: No entendre COUNT correctament**  
- ```COUNT(p.id_pokemon)``` compta files amb ```id_pokemon``` no nul.  
- ```COUNT(*)``` compta totes les files, incloses les amb valors nuls.

**Exemple confós**:
```sql
SELECT e.nom_entrenador, COUNT(*) AS total
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Resultat**:
| nom_entrenador | total |
|----------------|-------|
| Red            | 3     |
| Blue           | 3     |
| Green          | 3     |
| Yellow         | 1     | -- Compte! Yellow té 0 Pokémon, però 1 fila

**Correcte** (per comptar només Pokémon):
```sql
SELECT e.nom_entrenador, COUNT(p.id_pokemon) AS total
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
GROUP BY e.id_entrenador, e.nom_entrenador;
```

**Solució**: Usa ```COUNT(columna)``` per comptar només valors no nuls. Usa ```COUNT(*)``` només si vols comptar files, independentment de nuls.

---

## 5) **Exemple Final**  
**“Vull els entrenadors amb Pokémon capturats el 2025, amb el nivell mitjà dels seus Pokémon i el total de captures, però només si tenen un nivell mitjà superior a 30, ordenats per total de Pokémon.”**

```sql
SELECT e.nom_entrenador,
       COUNT(p.id_pokemon) AS total_pokemon,
       AVG(p.nivell) AS nivell_mitja
FROM entrenador e
LEFT JOIN pokemon p ON e.id_entrenador = p.id_entrenador
WHERE YEAR(p.capturat) = 2025
GROUP BY e.id_entrenador, e.nom_entrenador
HAVING AVG(p.nivell) > 30
ORDER BY total_pokemon DESC;
```

**Resultat**:
| nom_entrenador | total_pokemon | nivell_mitja |
|----------------|---------------|--------------|
| Blue           | 3             | 38.0         |
| Green          | 3             | 35.7         |

**Explicació**:
- ```WHERE YEAR(p.capturat) = 2025``` filtra Pokémon capturats el 2025.
- ```GROUP BY e.id_entrenador, e.nom_entrenador``` agrupa per entrenador.
- ```HAVING AVG(p.nivell) > 30``` filtra grups amb nivell mitjà > 30.
- ```ORDER BY total_pokemon DESC``` ordena per total de Pokémon.

---

## **Checklist d’Examen (GROUP BY i HAVING)**

- [ ] Incloc **totes** les columnes no agregades del ```SELECT``` al ```GROUP BY```.
- [ ] Uso ```WHERE``` per filtrar files individuals i ```HAVING``` per filtrar grups.
- [ ] Trio el ```JOIN``` correcte (ex.: ```LEFT JOIN``` per incloure registres sense parella).
- [ ] Entenc la diferència entre ```COUNT(columna)``` (no compta nuls) i ```COUNT(*)``` (compta totes les files).
- [ ] Combino amb funcions com ```AVG```, ```SUM```, ```MAX```, ```MIN``` per treure suc als grups.
- [ ] Provo la consulta amb dades petites per detectar errors abans de llançar-la a taules grans.

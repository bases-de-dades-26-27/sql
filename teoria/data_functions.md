# Funcions de Temps en SQL

Les funcions de temps en SQL permeten manipular dates i hores. 
> **Esquema**  
> - ```pokemon(id_pokemon, nom, nivell, regio, id_entrenador, capturat)```  
> - ```entrenador(id_entrenador, nom_entrenador, data_naixement)```  
> 
> **Dades (resum)**:  
> - **Pokémon**: Bulbasaur (capturat: 2025-01-15 10:30:00), Charmander (2025-03-20 14:00:00), Pikachu (2025-06-10 09:00:00, sense entrenador).  
> - **Entrenadors**: Red (1995-05-10), Blue (1996-08-22), Yellow (1998-12-01, sense Pokémon).  
> 
> **Nota**: Totes les consultes assumeixen que estem al **19 d’octubre de 2025** (data actual).

---

## Per què necessites aquestes funcions?

Les dates i hores són claus per analitzar quan van passar coses: quan vas capturar un Pokémon, quants dies fa que un entrenador és al gimnàs, o fins i tot l’hora en què Pikachu va llançar el seu darrer atac. Les funcions de temps et permeten **extreure**, **formatar** i **calcular** amb dates i hores.

---

## 1) Extreure parts d’una data o hora

Aquestes funcions són com desmuntar un rellotge per veure’n les peces.

### ```YEAR```, ```MONTH```, ```DAY```, ```HOUR```
Retornen el **any**, **mes**, **dia** o **hora** d’una data/hora.

```sql
SELECT nom,
       YEAR(capturat) AS any_captura,
       MONTH(capturat) AS mes_captura,
       DAY(capturat) AS dia_captura,
       HOUR(capturat) AS hora_captura
FROM pokemon;
```

**Resultat**:

| nom        | any_captura | mes_captura | dia_captura | hora_captura |
|------------|-------------|-------------|-------------|--------------|
| Bulbasaur  | 2025        | 1           | 15          | 10           |
| Charmander | 2025        | 3           | 20          | 14           |
| Pikachu    | 2025        | 6           | 10          | 9            |

(3 rows)

### ```MONTHNAME```, ```DAYNAME```
Et donen el **nom** del mes o del dia de la setmana (en anglès).

```sql
SELECT nom,
       MONTHNAME(capturat) AS nom_mes,
       DAYNAME(capturat) AS nom_dia
FROM pokemon;
```

**Resultat (simulat)**:

| nom        | nom_mes  | nom_dia   |
|------------|----------|-----------|
| Bulbasaur  | January  | Wednesday |
| Charmander | March    | Thursday  |
| Pikachu    | June     | Tuesday   |

(3 rows)

### ```EXTRACT(unitat FROM data)```
És com les anteriors, però més flexible. Pots extreure qualsevol part (any, mes, dia, hora, minut, segon, etc.).

```sql
SELECT nom,
       EXTRACT(YEAR FROM capturat) AS any,
       EXTRACT(MONTH FROM capturat) AS mes,
       EXTRACT(DAY FROM capturat) AS dia
FROM pokemon
WHERE EXTRACT(MONTH FROM capturat) = 6; -- Només Pokémon capturats al juny
```

**Resultat (simulat)**:

| nom     | any  | mes | dia |
|---------|------|-----|-----|
| Pikachu | 2025 | 6   | 10  |

(1 rows)

---

## 2) Obtenir dates completes o actuals

### ```DATE```
Extreu només la **part de la data** (sense hora) d’un camp de tipus *datetime*.

```sql
SELECT nom,
       DATE(capturat) AS dia_captura
FROM pokemon;
```

**Resultat (simulat)**:

| nom        | dia_captura |
|------------|-------------|
| Bulbasaur  | 2025-01-15  |
| Charmander | 2025-03-20  |
| Pikachu    | 2025-06-10  |

(3 rows)

### ```NOW()```, ```CURDATE()```, ```CURTIME()```
- ```NOW()```: Data i hora actuals (ex.: 2025-10-19 11:55:00).  
- ```CURDATE()```: Només la data actual (ex.: 2025-10-19).  
- ```CURTIME()```: Només l’hora actual (ex.: 11:55:00).

```sql
SELECT NOW() AS avui,
       CURDATE() AS només_data,
       CURTIME() AS només_hora;
```

**Resultat (simulat)**:

| avui                | només_data  | només_hora |
|---------------------|-------------|------------|
| 2025-10-19 11:55:00 | 2025-10-19 | 11:55:00   |

(1 rows)

---

## 3) Sumar o restar temps

### ```DATE_ADD(data, INTERVAL valor unitat)```, ```DATE_SUB(data, INTERVAL valor unitat)```
Afegeixen o resten dies, mesos, anys, hores, etc. a una data.

**Exemple**: Quan farà un any que vam capturar els Pokémon?

```sql
SELECT nom,
       capturat,
       DATE_ADD(capturat, INTERVAL 1 YEAR) AS un_any_despres
FROM pokemon;
```

**Resultat (simulat)**:

| nom        | capturat            | un_any_despres     |
|------------|---------------------|--------------------|
| Bulbasaur  | 2025-01-15 10:30:00 | 2026-01-15 10:30:00 |
| Charmander | 2025-03-20 14:00:00 | 2026-03-20 14:00:00 |
| Pikachu    | 2025-06-10 09:00:00 | 2026-06-10 09:00:00 |

(3 rows)

**Exemple 2**: Pokémon capturats fa menys de 6 mesos.

```sql
SELECT nom,
       capturat
FROM pokemon
WHERE capturat >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH);
```

**Resultat** (suposant avui és 2025-10-19):

| nom     | capturat            |
|---------|---------------------|
| Pikachu | 2025-06-10 09:00:00 |

(1 rows)

---

## 4) Calcular diferències de temps

### ```DATEDIFF(data1, data2)```
Retorna el **nombre de dies** entre dues dates (data1 - data2).

**Exemple**: Quants dies fa que vam capturar cada Pokémon?

```sql
SELECT nom,
       capturat,
       DATEDIFF(CURDATE(), capturat) AS dies_des_de_captura
FROM pokemon;
```

**Resultat**:

| nom        | capturat            | dies_des_de_captura |
|------------|---------------------|---------------------|
| Bulbasaur  | 2025-01-15 10:30:00 | 277                 |
| Charmander | 2025-03-20 14:00:00 | 213                 |
| Pikachu    | 2025-06-10 09:00:00 | 131                 |

(3 rows)

---

## 5) Formatejar dates

### ```DATE_FORMAT(data, format)```
Converteix una data en un format personalitzat. Usa codis com `%Y` (any), `%m` (mes), `%d` (dia), `%H` (hora), etc.

**Exemple**: Mostra la data de captura en format “15-Jan-2025”.

```sql
SELECT nom,
       DATE_FORMAT(capturat, '%d-%b-%Y') AS captura_formatejada
FROM pokemon;
```

**Resultat (simulat)**:

| nom        | captura_formatejada |
|------------|---------------------|
| Bulbasaur  | 15-Jan-2025         |
| Charmander | 20-Mar-2025         |
| Pikachu    | 10-Jun-2025         |

(3 rows)

**Codis útils**:
- `%Y`: Any amb 4 dígits (2025)
- `%b`: Nom curt del mes (Jan, Feb, etc.)
- `%d`: Dia amb 2 dígits (01-31)
- `%H`: Hora en format 24h (00-23)
- `%i`: Minuts (00-59)

---

## 6) Exemple pràctic amb JOIN i funcions de temps

**“Vull saber l’edat dels entrenadors i quants dies fa que van capturar els seus Pokémon, només dels capturats el 2025, ordenats per edat.”**

```sql
SELECT e.nom_entrenador,
       DATEDIFF(CURDATE(), e.data_naixement) / 365 AS edat_aprox,
       p.nom AS nom_pokemon,
       DATEDIFF(CURDATE(), p.capturat) AS dies_des_de_captura
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
WHERE YEAR(p.capturat) = 2025
ORDER BY edat_aprox DESC;
```

**Resultat (simulat)**:

| nom_entrenador | edat_aprox | nom_pokemon | dies_des_de_captura |
|----------------|------------|-------------|---------------------|
| Red            | 30.5       | Bulbasaur   | 277                 |
| Blue           | 29.2       | Charmander  | 213                 |
| Yellow         | 26.9       | NULL        | NULL                |

(3 rows)

---

## Checklist d’examen (Funcions de Temps)

- [ ] **Extreure**: ```YEAR```, ```MONTH```, ```DAY```, ```HOUR```, ```MONTHNAME```, ```DAYNAME```, o ```EXTRACT``` per agafar parts concretes.
- [ ] **Data actual**: ```NOW()``` (data + hora), ```CURDATE()``` (només data), ```CURTIME()``` (només hora).
- [ ] **Sumar/restar**: ```DATE_ADD``` o ```DATE_SUB``` amb ```INTERVAL``` (ex.: ```INTERVAL 1 YEAR```).
- [ ] **Diferències**: ```DATEDIFF``` per calcular dies entre dates.
- [ ] **Format**: ```DATE_FORMAT``` amb codis com ```%Y```, ```%b```, ```%d``` per personalitzar.
- [ ] **Filtrar**: Combina amb ```WHERE``` i ```JOIN``` per fer consultes potents.
- [ ] **Compte!** Les funcions com ```MONTHNAME``` i ```DAYNAME``` retornen noms en anglès.

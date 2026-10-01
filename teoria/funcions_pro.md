# **Funcions Avançades en SQL**

Les **funcions avançades** en SQL permeten manipular, formatar i transformar dades. Aquesta guia cobreix funcions com ```ROUND```, ```IFNULL```, ```COALESCE```, ```CONCAT```, ```CEIL```, ```FLOOR```, ```LEFT```, ```UCASE```, ```LCASE```, ```REPLACE```, ```LENGTH```, ```SUBSTR```, ```INSTR```, ```IF```, ```CASE```, i algunes de criptogràfiques com ```SHA```, amb exemples clars usant l’esquema de Pokémon. També incloem errors típics i com evitar-los.

> **Esquema (adaptat)**  
> - ```pokemon(id_pokemon, nom, nivell, regio, id_entrenador, capturat, experiencia, descripcio)```  
> - ```entrenador(id_entrenador, nom_entrenador, data_naixement, pokedollars, lema)```  
> 
> **Dades (resum)**:  
> - **Pokémon**:  
>   - Bulbasaur (id: 1, nivell: 12, regio: 'Kanto', id_entrenador: 101, capturat: 2025-01-15, experiencia: 1000, descripcio: 'Un Pokémon planta molt lleial').  
>   - Charizard (id: 2, nivell: 78, regio: 'Kanto', id_entrenador: 102, capturat: 2025-03-20, experiencia: 7800, descripcio: 'Un drac de foc poderós').  
>   - Pikachu (id: 3, nivell: 35, regio: 'Kanto', id_entrenador: NULL, capturat: 2025-06-10, experiencia: 3500, descripcio: 'Elèctric i adorable').  
> - **Entrenadors**:  
>   - Red (101, data_naixement: 1995-05-10, pokedollars: 5000, lema: 'Ser el millor!').  
>   - Blue (102, data_naixement: 1996-08-22, pokedollars: 8000, lema: 'Desafiar tothom').  
>   - Yellow (104, data_naixement: 1998-12-01, pokedollars: 2000, lema: NULL).  
> 
> **Nota**: Avui és **19 d’octubre de 2025, 14:46 CEST**.

---

## **Funcions Avançades: Tipus i Exemples**

### 1) **ROUND: Arrodonir**
Arrodoneix un valor numèric a un nombre determinat de decimals.

**Exemple**: Mostra el nivell mitjà de cada Pokémon, arrodonit a 1 decimal.
```sql
SELECT p.nom,
       ROUND(p.nivell / 2.0, 1) AS nivell_mitja
FROM pokemon p;
```

**Resultat (simulat)**:
| nom        | nivell_mitja |
|------------|--------------|
| Bulbasaur  | 6.0          |
| Charizard  | 39.0         |
| Pikachu    | 17.5         |

**Error Típic**: No especificar decimals (per defecte, arrodoneix a enters).
**Solució**: Sempre indica el nombre de decimals (ex.: ```ROUND(expr, 2)```).

---

### 2) **IFNULL i COALESCE: Gestionar NULLs**
- ```IFNULL(expr, valor)```: Retorna ```valor``` si ```expr``` és NULL.
- ```COALESCE(expr1, expr2, ...)```: Retorna el primer valor no NULL.

**Exemple**: Mostra el lema de l’entrenador o ‘Sense lema’ si és NULL.
```sql
SELECT nom_entrenador,
       IFNULL(lema, 'Sense lema') AS lema,
       COALESCE(lema, nom_entrenador, 'Desconegut') AS lema_coalesce
FROM entrenador;
```
**Taula inicial**:
| nom_entrenador | lema             |
|----------------|------------------|
| Red            | Ser el millor!   |
| Blue           | Desafiar tothom  |
| Yellow         | NULL             |
| NULL           | NULL             |

**Resultat (simulat)**:
| nom_entrenador | lema             | lema_coalesce    |
|----------------|------------------|------------------|
| Red            | Ser el millor!   | Ser el millor!   |
| Blue           | Desafiar tothom  | Desafiar tothom  |
| Yellow         | Sense lema       | Yellow           |
| NULL           | Sense lema       | Desconegut       |

**Error Típic**: Oblidar que ```IFNULL``` només accepta dos arguments, mentre que ```COALESCE``` en pot tenir més.
**Solució**: Usa ```COALESCE``` quan necessites comprovar múltiples columnes o valors.

---

### 3) **CONCAT: Unir Text**
Uneix cadenes de text o columnes.

**Exemple**: Crea una descripció completa amb nom i regio del Pokémon.
```sql
SELECT nom,
       CONCAT(nom, ' de ', regio) AS descripcio_completa
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | descripcio_completa    |
|------------|------------------------|
| Bulbasaur  | Bulbasaur de Kanto     |
| Charizard  | Charizard de Kanto     |
| Pikachu    | Pikachu de Kanto       |

**Error Típic**: Concatenar amb NULL (dóna NULL).
**Solució**: Usa ```COALESCE``` o ```IFNULL```:
```sql
SELECT nom,
       CONCAT(COALESCE(nom, 'Desconegut'), ' de ', COALESCE(regio, 'Sense regio')) AS descripcio_completa
FROM pokemon;
```

---

### 4) **CEIL i FLOOR: Arrodonir Amunt o Avall**
- ```CEIL```: Arrodoneix cap amunt al següent enter.
- ```FLOOR```: Arrodoneix cap avall a l’enter anterior.

**Exemple**: Calcula el nivell arrodonit cap amunt i avall després de dividir per 10.
```sql
SELECT nom,
       nivell,
       CEIL(nivell / 10.0) AS nivell_ceil,
       FLOOR(nivell / 10.0) AS nivell_floor
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | nivell | nivell_ceil | nivell_floor |
|------------|--------|-------------|--------------|
| Bulbasaur  | 12     | 2           | 1            |
| Charizard  | 78     | 8           | 7            |
| Pikachu    | 35     | 4           | 3            |

**Error Típic**: Confondre ```CEIL``` i ```FLOOR```.
**Solució**: Recorda: ```CEIL``` puja, ```FLOOR``` baixa.

---

### 5) **LEFT: Agafar trossos de text**
Extreu els primers n caràcters d’una cadena.

**Exemple**: Mostra els 3 primers caràcters del nom del Pokémon.
```sql
SELECT nom,
       LEFT(nom, 3) AS nom_curt
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | nom_curt |
|------------|----------|
| Bulbasaur  | Bul      |
| Charizard  | Cha      |
| Pikachu    | Pik      |

**Error Típic**: Usar un valor negatiu o zero a ```LEFT```.
**Solució**: Assegura’t que el segon paràmetre sigui un enter positiu.

---

### 6) **UCASE i LCASE: Canviar Majúscules/Minúscules**
- ```UCASE```: Converteix a majúscules.
- ```LCASE```: Converteix a minúscules.

**Exemple**: Mostra el nom del Pokémon en majúscules i minúscules.
```sql
SELECT nom,
       UCASE(nom) AS nom_majuscules,
       LCASE(nom) AS nom_minuscules
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | nom_majuscules | nom_minuscules |
|------------|----------------|----------------|
| Bulbasaur  | BULBASAUR      | bulbasaur      |
| Charizard  | CHARIZARD      | charizard      |
| Pikachu    | PIKACHU        | pikachu        |

**Error Típic**: Aplicar ```UCASE```/```LCASE``` a columnes no textuals.
**Solució**: Verifica que la columna sigui de tipus VARCHAR o TEXT.

---

### 7) **REPLACE: Canvia Text**
Substitueix totes les aparicions d’una subcadena per una altra.

**Exemple**: Canvia ‘Pokémon’ per ‘Criatura’ en la descripció.
```sql
SELECT nom,
       REPLACE(descripcio, 'Pokémon', 'Criatura') AS nova_descripcio
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | nova_descripcio               |
|------------|-------------------------------|
| Bulbasaur  | Un Criatura planta molt lleial |
| Charizard  | Un drac de foc poderós        |
| Pikachu    | Elèctric i adorable           |

**Error Típic**: Usar ```REPLACE``` amb cadenes sensibles a majúscules/minúscules.
**Solució**: Converteix amb ```UCASE``` o ```LCASE``` abans si cal.

---

### 8) **LENGTH: Mesura Text**
Compta el nombre de caràcters d’una cadena.

**Exemple**: Compta els caràcters del lema de l’entrenador.
```sql
SELECT nom_entrenador,
       LENGTH(COALESCE(lema, '')) AS longitud_lema
FROM entrenador;
```

**Resultat (simulat)**:
| nom_entrenador | longitud_lema |
|----------------|---------------|
| Red            | 12            | -- 'Ser el millor!'
| Blue           | 14            | -- 'Desafiar tothom'
| Yellow         | 0             | -- NULL convertit a ''

**Error Típic**: No gestionar NULLs (```LENGTH(NULL)``` dóna NULL).
**Solució**: Usa ```COALESCE``` per substituir NULLs per una cadena buida.

---

### 9) **SUBSTR: Talla Text**
Extreu una part d’una cadena (```SUBSTR(cadena, inici, longitud)```).

**Exemple**: Extreu els 5 primers caràcters de la descripció.
```sql
SELECT nom,
       SUBSTR(descripcio, 1, 5) AS desc_curt
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | desc_curt |
|------------|-----------|
| Bulbasaur  | Un Po     |
| Charizard  | Un dr     |
| Pikachu    | Elèct     |

**Error Típic**: Índex d’inici incorrecte (en SQL, l’índex comença a 1, no a 0).
**Solució**: Recorda que ```SUBSTR(cadena, 1, n)``` comença al primer caràcter.

---

### 10) **INSTR: Troba Text**
Retorna la posició de la primera aparició d’una subcadena.

**Exemple**: Troba la posició de ‘Pokémon’ a la descripció.
```sql
SELECT nom,
       INSTR(descripcio, 'Pokémon') AS posicio_pokemon
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | posicio_pokemon |
|------------|-----------------|
| Bulbasaur  | 4               | -- ‘Un Pokémon...’
| Charizard  | 0               | -- No trobat
| Pikachu    | 0               | -- No trobat

**Error Típic**: Confondre que 0 significa “no trobat”.
**Solució**: Comprova si ```INSTR``` retorna 0 abans d’usar el resultat.

---

### 11) **IF: Condicions**
```IF(condició, valor_si_verdader, valor_si_fals)```: Retorna un valor segons una condició.

**Exemple**: Classifica els Pokémon com “Fort” o “Normal” segons el nivell.
```sql
SELECT nom,
       nivell,
       IF(nivell > 50, 'Fort', 'Normal') AS classificacio
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | nivell | classificacio |
|------------|--------|---------------|
| Bulbasaur  | 12     | Normal        |
| Charizard  | 78     | Fort          |
| Pikachu    | 35     | Normal        |

**Error Típic**: No gestionar casos límit (ex.: nivell = 50).
**Solució**: Defineix clarament els límits a la condició.

---

### 12) **CASE: Condicions Múltiples**
Permet múltiples condicions amb ```CASE ... WHEN ... THEN ... ELSE ... END```.

**Exemple**: Classifica els Pokémon per rangs de nivell.
```sql
SELECT nom,
       nivell,
       CASE
           WHEN nivell > 50 THEN 'Llegenda'
           WHEN nivell > 20 THEN 'Intermedi'
           ELSE 'Principiant'
       END AS rang
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | nivell | rang         |
|------------|--------|--------------|
| Bulbasaur  | 12     | Principiant  |
| Charizard  | 78     | Llegenda     |
| Pikachu    | 35     | Intermedi    |

**Error Típic**: Oblidar ```ELSE``` (retorna NULL si cap condició es compleix).
**Solució**: Sempre inclou un ```ELSE``` per defecte.

---

### 13) **SHA: Encriptació**
```SHA(cadena)``` genera un hash SHA-1 de 40 caràcters.

**Exemple**: Encripta el nom de l’entrenador.
```sql
SELECT nom_entrenador,
       SHA(nom_entrenador) AS hash_nom
FROM entrenador;
```

**Resultat (simulat)**:
| nom_entrenador | hash_nom                                  |
|----------------|-------------------------------------------|
| Red            | a4d3... (hash de 40 caràcters)            |
| Blue           | b7e2... (hash de 40 caràcters)            |
| Yellow         | c9f1... (hash de 40 caràcters)            |

**Error Típic**: Assumir que ```SHA``` és reversible (no ho és).
**Solució**: Usa ```SHA``` només per encriptació unidireccional (ex.: contrasenyes).

---

### 14) **BINARY: Comparacions Sensibles a Majúscules**
Força una comparació sensible a majúscules/minúscules.

**Exemple**: Compara noms en majúscules/minúscules.
```sql
SELECT nom,
       IF(BINARY nom = 'PIKACHU', 'És PIKACHU', 'No és PIKACHU') AS comparacio
FROM pokemon;
```

**Resultat (simulat)**:
| nom        | comparacio         |
|------------|--------------------|
| Bulbasaur  | No és PIKACHU      |
| Charizard  | No és PIKACHU      |
| Pikachu    | No és PIKACHU      | -- Perquè és ‘Pikachu’, no ‘PIKACHU’

**Error Típic**: No entendre que ```BINARY``` afecta la comparació.
**Solució**: Usa ```BINARY``` només quan necessites diferenciar majúscules/minúscules.

---

## 15) **Exemple Combinat amb JOIN i Funcions Avançades**

**“Vull els entrenadors amb els seus Pokémon, mostrant una descripció personalitzada (nom del Pokémon + lema de l’entrenador), el nivell arrodonit cap avall, i una classificació basada en l’experiència, ordenats per una puntuació (nivell * experiència).”**

```sql
SELECT e.nom_entrenador,
       p.nom AS nom_pokemon,
       CONCAT(UCASE(LEFT(p.nom, 1)), LCASE(SUBSTR(p.nom, 2)), ' - ', COALESCE(e.lema, 'Sense lema')) AS descripcio,
       FLOOR(p.nivell) AS nivell_arrodonit,
       CASE
           WHEN p.experiencia > 5000 THEN 'Épic'
           WHEN p.experiencia > 2000 THEN 'Avançat'
           ELSE 'Bàsic'
       END AS nivell_experiencia,
       p.nivell * p.experiencia AS puntuacio
FROM entrenador e
LEFT JOIN pokemon p
  ON e.id_entrenador = p.id_entrenador
ORDER BY p.nivell * p.experiencia DESC;
```

**Resultat (simulat)**:
| nom_entrenador | nom_pokemon | descripcio                     | nivell_arrodonit | nivell_experiencia | puntuacio |
|----------------|-------------|--------------------------------|------------------|--------------------|-----------|
| Blue           | Charizard   | Charizard - Desafiar tothom    | 78               | Épic               | 608400    |
| Red            | Bulbasaur   | Bulbasaur - Ser el millor!     | 12               | Bàsic              | 12000     |
| Yellow         | NULL        | NULL                           | NULL             | NULL               | NULL      |
| NULL           | Pikachu     | Pikachu - Sense lema           | 35               | Avançat            | 122500    |

**Explicació**:
- ```CONCAT``` i ```UCASE```/```LCASE``` creen una descripció estilitzada.
- ```FLOOR``` arrodoneix el nivell cap avall.
- ```CASE``` classifica l’experiència.
- ```ORDER BY p.nivell * p.experiencia``` ordena per puntuació.

---

## 16) **Errors Típics i Com Evitar-los**

### **Error 1: No gestionar NULLs**
Moltes funcions (```CONCAT```, ```LENGTH```, operacions aritmètiques) retornen NULL si algun argument és NULL.

**Malament**:
```sql
SELECT CONCAT(nom_entrenador, ' - ', lema) AS descripcio
FROM entrenador;
```

Yellow retorna NULL perquè ```lema``` és NULL.

**Correcte**:
```sql
SELECT CONCAT(nom_entrenador, ' - ', COALESCE(lema, 'Sense lema')) AS descripcio
FROM entrenador;
```

**Solució**: Usa ```COALESCE``` o ```IFNULL``` per substituir NULLs.

---

### **Error 2: Confondre índexs a SUBSTR**
En SQL, els índexs de ```SUBSTR``` comencen a 1, no a 0.

**Malament**:
```sql
SELECT SUBSTR(nom, 0, 3) AS nom_curt FROM pokemon; -- Error o resultat inesperat
```

**Correcte**:
```sql
SELECT SUBSTR(nom, 1, 3) AS nom_curt FROM pokemon;
```

**Solució**: Recorda que l’índex comença a 1.

---

### **Error 3: Usar funcions de text en columnes no textuals**
Funcions com ```UCASE```, ```LCASE```, ```REPLACE``` només funcionen amb VARCHAR o TEXT.

**Malament**:
```sql
SELECT UCASE(nivell) FROM pokemon; -- Error! nivell és INT
```

**Solució**: Converteix a text amb ```CAST``` si cal:
```sql
SELECT UCASE(CAST(nivell AS CHAR)) FROM pokemon;
```

---

### **Error 4: No especificar ELSE en CASE**
Si cap condició de ```CASE``` es compleix, retorna NULL.

**Malament**:
```sql
SELECT nom,
       CASE WHEN nivell > 50 THEN 'Fort' END AS classificacio
FROM pokemon;
```

**Correcte**:
```sql
SELECT nom,
       CASE WHEN nivell > 50 THEN 'Fort' ELSE 'Normal' END AS classificacio
FROM pokemon;
```

**Solució**: Inclou sempre un ```ELSE``` per defecte.

---

## 17) **Checklist d’Examen (Funcions Avançades)**

- [ ] Usa ```COALESCE``` o ```IFNULL``` per gestionar NULLs en ```CONCAT```, ```LENGTH```, etc.
- [ ] Verifica que les columnes siguin del tipus correcte (ex.: text per ```UCASE```, numèric per ```ROUND```).
- [ ] Recorda que ```SUBSTR``` i ```INSTR``` usen índexs a partir de 1.
- [ ] Inclou ```ELSE``` en ```CASE``` per evitar NULLs inesperats.
- [ ] Usa ```ROUND```, ```CEIL```, o ```FLOOR``` segons el tipus d’arrodoniment.
- [ ] Prova amb dades petites per assegurar-te que els resultats són correctes.
- [ ] Combina amb ```JOIN```, ```GROUP BY```, i ```ORDER BY``` per anàlisis potents.

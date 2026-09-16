# Bases de dades: conceptes bàsics

## 1. Què és una base de dades?

Una **base de dades (BD)** és un conjunt de dades organitzades i relacionades entre si, guardades de manera que es puguin consultar i modificar fàcilment.

El programa que gestiona la base de dades s'anomena **SGBD** (*Sistema Gestor de Bases de Dades*). És qui guarda les dades, controla qui hi accedeix i respon les nostres preguntes. Alguns exemples són **MySQL**, MariaDB, PostgreSQL, Oracle o SQL Server.

> **Per què no fer servir un full de càlcul?** Un Excel funciona bé amb poques dades i un sol usuari. Una BD pot gestionar milions de registres, permet que moltes persones hi treballin alhora sense trepitjar-se i garanteix que les dades siguin coherents.

---

## 2. La taula

En una base de dades **relacional**, les dades s'organitzen en **taules**. Cada taula guarda informació sobre un únic tipus d'element: videojocs, alumnes, factures, etc.

Taula `videojocs` (fragment):

| id | titol          | genere    | plataforma | preu  | pegi |
|----|----------------|-----------|------------|-------|------|
| 3  | Hollow Knight  | Metroidvania | PC      | 14.99 | 7    |
| 6  | Stardew Valley | Simulació | Switch     | 13.99 | 7    |
| 10 | Hades          | Roguelike | PC         | 24.99 | 12   |
| 17 | Tetris         | Puzle     | PC         | 9.99  | NULL |

| Concepte | També s'anomena | Què és | A l'exemple |
|----------|-----------------|--------|-------------|
| **Taula** | Entitat, relació | El conjunt de dades sobre un mateix tema | `videojocs` |
| **Fila** | Registre, tupla | Un element concret | La fila de *Hades* |
| **Columna** | Camp, atribut | Una característica que tenen tots els elements | `preu` |
| **Valor** | Dada | El contingut d'una cel·la | `24.99` |

Totes les files d'una taula tenen **les mateixes columnes**, i cada columna guarda sempre **el mateix tipus de dada**.

---

## 3. Tipus de dades

Quan es crea una taula, cada columna té un tipus que indica quina mena de valors pot guardar. Els més habituals a MySQL són:

### Números

| Tipus | Per a què | Exemple |
|-------|-----------|---------|
| `INT` | Nombres enters | `2023`, `-5`, `18` |
| `DECIMAL(p,d)` | Decimals exactes (diners). `p` = xifres totals, `d` = decimals | `DECIMAL(5,2)` → fins a `999.99` |
| `FLOAT` / `DOUBLE` | Decimals aproximats (càlculs científics) | `3.14159` |

### Text

| Tipus | Per a què | Exemple |
|-------|-----------|---------|
| `VARCHAR(n)` | Text de longitud variable, fins a `n` caràcters | `VARCHAR(100)` per a un títol |
| `CHAR(n)` | Text de longitud fixa | `CHAR(9)` per a un DNI |
| `TEXT` | Textos llargs | Una descripció o un comentari |

### Dates i hores

| Tipus | Format | Exemple |
|-------|--------|---------|
| `DATE` | `AAAA-MM-DD` | `'2023-05-12'` |
| `TIME` | `HH:MM:SS` | `'09:30:00'` |
| `DATETIME` | `AAAA-MM-DD HH:MM:SS` | `'2023-05-12 09:30:00'` |

### Altres

| Tipus | Per a què | Exemple |
|-------|-----------|---------|
| `BOOLEAN` | Cert o fals. MySQL el guarda com `1` o `0` | `multijugador` |

> **Compte amb les cometes:** els textos i les dates van entre **cometes simples** (`'Switch'`, `'2023-05-12'`). Els números, **sense cometes** (`24.99`).

---

## 4. El valor NULL

`NULL` vol dir **"no hi ha valor"** o **"no ho sabem"**. No és el mateix que `0` ni que un text buit `''`.

| Valor | Significat |
|-------|------------|
| `preu = 0` | El joc és gratuït |
| `preu = NULL` | No sabem quant val |

Com que NULL no és un valor, **no es pot comparar amb `=`**. Per buscar-lo es fa servir `IS NULL` o `IS NOT NULL`:

```sql
SELECT titol FROM videojocs WHERE pegi IS NULL;   -- correcte
SELECT titol FROM videojocs WHERE pegi = NULL;    -- no retorna res
```

---

## 5. Restriccions

Les **restriccions** són regles que la BD fa complir per evitar dades incorrectes.

| Restricció | Què obliga | Exemple |
|------------|-----------|---------|
| `NOT NULL` | La columna no pot quedar buida | Tot joc ha de tenir `titol` |
| `UNIQUE` | No hi pot haver dos valors iguals | Dos usuaris no poden tenir el mateix email |
| `DEFAULT` | Valor per defecte si no se n'indica cap | `multijugador DEFAULT FALSE` |
| `PRIMARY KEY` | Identifica cada fila (vegeu l'apartat següent) | `id` |
| `FOREIGN KEY` | Enllaça amb una altra taula (vegeu l'apartat 7) | `id_desenvolupadora` |

---

## 6. Clau primària

La **clau primària** (*primary key*, PK) és la columna (o conjunt de columnes) que **identifica de manera única cada fila**.

Ha de complir dues condicions: **no es pot repetir** i **no pot ser NULL**.

A la taula `videojocs`, la clau primària és `id`. Per què no el `titol`? Perquè hi podria haver dos jocs amb el mateix nom (un remake, per exemple), i perquè un títol es pot corregir, mentre que un identificador no hauria de canviar mai.

> Habitualment es fa servir un número que la BD genera sola: `id INT AUTO_INCREMENT PRIMARY KEY`.

---

## 7. Relacions entre taules

Fixa't en aquesta columna de la taula `videojocs`:

| titol | desenvolupadora |
|-------|-----------------|
| Zelda: Tears of the Kingdom | Nintendo |
| Mario Kart 8 Deluxe | Nintendo |
| Animal Crossing: New Horizons | nintendo |

"Nintendo" es repeteix, i si algú l'escriu malament, la BD ja no sap que és la mateixa empresa. A més, si volguéssim guardar el país o la web de cada desenvolupadora, ho hauríem de repetir a cada joc.

La solució és **separar la informació en dues taules** i **relacionar-les**:

**Taula `desenvolupadores`**

| id | nom       | pais   |
|----|-----------|--------|
| 1  | Nintendo  | Japó   |
| 2  | Team Cherry | Austràlia |
| 3  | Nomada Studio | Espanya |

**Taula `videojocs`**

| id | titol          | id_desenvolupadora |
|----|----------------|--------------------|
| 1  | Zelda: TotK    | 1                  |
| 3  | Hollow Knight  | 2                  |
| 11 | Mario Kart 8 Deluxe | 1             |
| 14 | Gris           | 3                  |

La columna `id_desenvolupadora` és una **clau forana** (*foreign key*, FK): guarda la clau primària d'una fila d'una altra taula. La BD comprova que el valor existeixi; no deixarà assignar un joc a la desenvolupadora `99` si no existeix.

### Tipus de relació

| Tipus | Significat | Exemple |
|-------|-----------|---------|
| **1 : 1** | Un element d'A es relaciona amb un sol element de B, i viceversa | Un alumne ↔ el seu expedient |
| **1 : N** | Un element d'A es relaciona amb molts de B, però cada element de B només amb un d'A | Una desenvolupadora → molts jocs |
| **N : M** | Molts d'A es relacionen amb molts de B | Jocs ↔ plataformes (un joc surt a diverses consoles i cada consola té molts jocs) |

Les relacions **N : M** necessiten una **taula intermèdia**:

| id_videojoc | id_plataforma |
|-------------|---------------|
| 2 (Minecraft) | 1 (PC)      |
| 2 (Minecraft) | 2 (Switch)  |
| 2 (Minecraft) | 3 (PS5)     |

> A la nostra taula de pràctiques cada joc té una sola plataforma. És una simplificació per començar amb consultes sobre una sola taula; més endavant veurem com es dissenya correctament.

---

## 8. SQL

**SQL** (*Structured Query Language*) és el llenguatge per comunicar-nos amb una base de dades relacional. Les instruccions es classifiquen en grups:

| Grup | Nom | Per a què | Instruccions |
|------|-----|-----------|--------------|
| **DDL** | *Data Definition Language* | Crear i modificar l'estructura | `CREATE`, `ALTER`, `DROP` |
| **DML** | *Data Manipulation Language* | Afegir, modificar i esborrar dades | `INSERT`, `UPDATE`, `DELETE` |
| **DQL** | *Data Query Language* | Consultar dades | `SELECT` |
| **DCL** | *Data Control Language* | Gestionar permisos | `GRANT`, `REVOKE` |

Començarem per **DQL**: aprendre a fer preguntes a una base de dades que ja existeix.

```sql
SELECT titol, preu          -- què vull veure
FROM videojocs              -- d'on
WHERE plataforma = 'Switch' -- quines files
ORDER BY preu;              -- en quin ordre
```

> Una consulta `SELECT` **mai modifica les dades**. El resultat és una taula temporal que només es mostra per pantalla.

---

## Glossari ràpid

| Terme | Definició breu |
|-------|---------------|
| Base de dades | Conjunt de dades organitzades i relacionades |
| SGBD | Programa que gestiona la BD (MySQL, PostgreSQL…) |
| Taula | Conjunt de dades sobre un mateix tema |
| Fila / registre | Un element concret de la taula |
| Columna / camp | Una característica comuna a tots els elements |
| Tipus de dada | Mena de valor que admet una columna |
| NULL | Absència de valor |
| Restricció | Regla que la BD obliga a complir |
| Clau primària (PK) | Columna que identifica cada fila de manera única |
| Clau forana (FK) | Columna que apunta a la clau primària d'una altra taula |
| Relació | Vincle entre taules (1:1, 1:N, N:M) |
| SQL | Llenguatge per treballar amb bases de dades relacionals |
| Consulta | Pregunta que fem a la BD amb `SELECT` |

---
id: query-service
title: QueryService — Referencia de métodos
sidebar_label: QueryService
sidebar_position: 2
---

# `QueryService` — Referencia completa

**Ubicación:** `NeoGenesis/Modules/Query/QueryService.cs`

`QueryService` es la clase responsable de toda la comunicación con la base de datos dentro del módulo de consultas. Recibe el `MySqlDbContext` por inyección en el constructor y expone métodos públicos para cada tipo de búsqueda.

---

## Constructor

```csharp
public QueryService(MySqlDbContext db)
```

| Parámetro | Tipo | Descripción |
|---|---|---|
| `db` | `MySqlDbContext` | Contexto de Entity Framework Core |

---

## Método privado base

```csharp
private List<Dinosaur> All() => _db.Dinosaurs.ToList();
```

Recupera **todos los dinosaurios** de la base de datos y los carga en memoria. Es el punto de partida de todas las consultas públicas.

---

## Métodos públicos

### `GetAllDinosaurs()`

```csharp
public List<Dinosaur> GetAllDinosaurs()
```

**Descripción:** Retorna la lista completa de todos los dinosaurios registrados en el sistema.

**Retorna:** `List<Dinosaur>` — todos los registros sin filtro ni orden específico.

**Uso en menú:** Opciones 1 (listado) y 9 (conteo total).

**Ejemplo de salida:**
```
[ Dinosaur{Id=1, DinoName="Rex", ...}, Dinosaur{Id=2, DinoName="Velo", ...}, ... ]
```

---

### `GetDinosaurById(int id)`

```csharp
public List<Dinosaur> GetDinosaurById(int id)
```

**Descripción:** Busca y retorna el dinosaurio cuyo `Id` coincida exactamente con el valor proporcionado.

| Parámetro | Tipo | Descripción |
|---|---|---|
| `id` | `int` | Identificador numérico único del dinosaurio |

**Retorna:** `List<Dinosaur>` — lista con 0 o 1 elemento.

:::tip
Retorna una lista (no un objeto único) para mantener consistencia con el resto de métodos del servicio y simplificar el renderizado en `Helpers.ShowDinosaurs()`.
:::

**Uso en menú:** Opción 2.

---

### `GetDinosaurByCode(string code)`

```csharp
public List<Dinosaur> GetDinosaurByCode(string code)
```

**Descripción:** Busca dinosaurios cuyo campo `RegisterCode` coincida exactamente con el código dado. Como `RegisterCode` tiene restricción de unicidad en BD, en la práctica retorna 0 o 1 resultado.

| Parámetro | Tipo | Descripción |
|---|---|---|
| `code` | `string` | Código de registro único del dinosaurio |

**Retorna:** `List<Dinosaur>` — lista con 0 o 1 elemento.

**Uso en menú:** Opción 3.

---

### `GetCodes()`

```csharp
public List<string> GetCodes()
```

**Descripción:** Extrae todos los `RegisterCode` existentes en el sistema. Se usa para construir el menú de selección interactivo antes de filtrar por código.

**Retorna:** `List<string>` — lista de códigos de registro (puede haber duplicados si el índice único fue eludido en migraciones antiguas).

**Uso en menú:** Construcción del menú de opciones para la opción 3.

---

### `GetDinosaursByZone(string zone)`

```csharp
public List<Dinosaur> GetDinosaursByZone(string zone)
```

**Descripción:** Filtra todos los dinosaurios que habiten en una zona específica del parque. La comparación es exacta (case-sensitive).

| Parámetro | Tipo | Descripción |
|---|---|---|
| `zone` | `string` | Nombre de la zona del parque |

**Retorna:** `List<Dinosaur>` — todos los dinosaurios de esa zona.

**Uso en menú:** Opciones 4 (listado) y 10 (conteo).

---

### `GetZones()`

```csharp
public List<string> GetZones()
```

**Descripción:** Obtiene las zonas únicas registradas en el sistema, excluyendo valores nulos.

**Retorna:** `List<string>` — zonas distintas y no nulas.

**Uso en menú:** Construcción del menú de selección para opciones 4 y 10.

---

### `GetDinosaursBySector(string sector)`

```csharp
public List<Dinosaur> GetDinosaursBySector(string sector)
```

**Descripción:** Filtra dinosaurios por el sector específico dentro del parque. La comparación es exacta.

| Parámetro | Tipo | Descripción |
|---|---|---|
| `sector` | `string` | Nombre del sector del parque |

**Retorna:** `List<Dinosaur>` — todos los dinosaurios de ese sector.

**Uso en menú:** Opciones 5 (listado) y 11 (conteo).

---

### `GetSectors()`

```csharp
public List<string> GetSectors()
```

**Descripción:** Obtiene los sectores únicos registrados, excluyendo valores nulos.

**Retorna:** `List<string>` — sectores distintos y no nulos.

**Uso en menú:** Construcción del menú de selección para opciones 5 y 11.

---

### `GetDinosaursByAge(int? age)`

```csharp
public List<Dinosaur> GetDinosaursByAge(int? age)
```

**Descripción:** Retorna todos los dinosaurios cuya edad sea **mayor** al valor ingresado. Funciona como filtro `>` (mayor estricto), no `>=`.

| Parámetro | Tipo | Descripción |
|---|---|---|
| `age` | `int?` | Edad mínima (exclusiva). Nullable para compatibilidad con `Helpers.IntValidation` |

**Retorna:** `List<Dinosaur>` — dinosaurios mayores a la edad indicada.

**Ejemplo:** `GetDinosaursByAge(5)` → retorna dinosaurios de 6, 10, 20 años, etc.

**Uso en menú:** Opción 6.

:::caution
Los dinosaurios con `Age = null` **no aparecen** en el resultado, ya que la comparación `null > age` evalúa a `false` en C# con nullable types.
:::

---

### `GetDinosaursByType(string type)`

```csharp
public List<Dinosaur> GetDinosaursByType(string type)
```

**Descripción:** Filtra dinosaurios según su tipo alimenticio. Los valores válidos son `"Herbivore"` y `"Carnivore"` (definidos en `QueryMenu`).

| Parámetro | Tipo | Descripción |
|---|---|---|
| `type` | `string` | `"Herbivore"` o `"Carnivore"` |

**Retorna:** `List<Dinosaur>` — dinosaurios del tipo especificado.

**Uso en menú:** Opción 7.

---

### `GetDinosaursForReports()`

```csharp
public List<string> GetDinosaursForReports()
```

**Descripción:** Genera una vista resumida para reportes con formato tabulado. Combina `DinoName` y `RegisterCode` en un string formateado con `PadRight` para alineación en consola.

**Retorna:** `List<string>` — cada elemento tiene el formato:
```
"NombreDino         CODE123        "
// 20 chars padding + 15 chars padding
```

**Uso en menú:** Opción 8. Se renderiza con `Helpers.ShowDinosaursforReports()`.

---

### `OrderByCreationDate()`

```csharp
public List<Dinosaur> OrderByCreationDate()
```

**Descripción:** Retorna todos los dinosaurios ordenados de forma **ascendente** por su fecha de creación (`CreatedAt`). El primer elemento es el dinosaurio registrado más antiguo.

**Retorna:** `List<Dinosaur>` — lista ordenada por `CreatedAt` ASC.

**Uso en menú:** Opción 14 ("Last registered dinosaurs" — aunque el orden es ascendente, muestra el historial cronológico).

---

### `GetDinosaursWoTracking()`

```csharp
public List<Dinosaur> GetDinosaursWoTracking()
```

**Descripción:** Filtra los dinosaurios que **no tienen** número de rastreo asignado (`TrackNumber == null`).

**Retorna:** `List<Dinosaur>` — dinosaurios sin `TrackNumber`.

**Uso en menú:** Opción 12.

---

### `GetDinosaursWoAddress()`

```csharp
public List<Dinosaur> GetDinosaursWoAddress()
```

**Descripción:** Filtra los dinosaurios que **no tienen** dirección registrada (`Address == null`).

**Retorna:** `List<Dinosaur>` — dinosaurios sin `Address`.

**Uso en menú:** Opción 13.

---

### `DinosaursBySpecies()`

```csharp
public List<Dinosaur> DinosaursBySpecies()
```

**Descripción:** Retorna todos los dinosaurios ordenados **alfabéticamente** por su especie (`DinoSpecies`) en orden ascendente (A → Z).

**Retorna:** `List<Dinosaur>` — lista ordenada por `DinoSpecies` ASC.

**Uso en menú:** Opción 15.

---

## Código fuente completo

```csharp
public class QueryService
{
    private readonly MySqlDbContext _db;

    public QueryService(MySqlDbContext db) => _db = db;

    private List<Dinosaur> All() => _db.Dinosaurs.ToList();

    public List<Dinosaur> GetAllDinosaurs() => All();
    public List<Dinosaur> GetDinosaurById(int id) => All().Where(d => d.Id == id).ToList();
    public List<Dinosaur> GetDinosaurByCode(string code) => All().Where(d => d.RegisterCode == code).ToList();
    public List<string>   GetCodes() => All().Select(d => d.RegisterCode).ToList();
    public List<Dinosaur> GetDinosaursByZone(string zone) => All().Where(d => d.Zone == zone).ToList();
    public List<string>   GetZones() => All().Select(d => d.Zone).Distinct().Where(z => z != null).ToList()!;
    public List<Dinosaur> GetDinosaursBySector(string sector) => All().Where(d => d.Sector == sector).ToList();
    public List<string>   GetSectors() => All().Select(d => d.Sector).Distinct().Where(s => s != null).ToList()!;
    public List<Dinosaur> GetDinosaursByAge(int? age) => All().Where(d => d.Age > age).ToList();
    public List<Dinosaur> GetDinosaursByType(string type) => All().Where(d => d.Type == type).ToList();
    public List<string>   GetDinosaursForReports() => All().Select(d => $"{d.DinoName.PadRight(20)}{d.RegisterCode.PadRight(15)}").ToList();
    public List<Dinosaur> OrderByCreationDate() => All().OrderBy(d => d.CreatedAt).ToList();
    public List<Dinosaur> GetDinosaursWoTracking() => All().Where(d => d.TrackNumber == null).ToList();
    public List<Dinosaur> GetDinosaursWoAddress() => All().Where(d => d.Address == null).ToList();
    public List<Dinosaur> DinosaursBySpecies() => All().OrderBy(d => d.DinoSpecies).ToList();
}
```

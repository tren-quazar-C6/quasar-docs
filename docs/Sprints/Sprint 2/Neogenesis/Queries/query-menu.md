---
id: query-menu
title: QueryMenu — Menú interactivo
sidebar_label: QueryMenu
sidebar_position: 3
---

# `QueryMenu` — Menú interactivo de consultas

**Ubicación:** `NeoGenesis/Modules/Query/QueryMenu.cs`

`QueryMenu` es la capa de presentación e interacción del módulo de consultas. Recibe la opción seleccionada por el usuario desde `Program.cs`, obtiene datos de `QueryService` y delega el renderizado a `Helpers`.

---

## Constructor

```csharp
public QueryMenu(MySqlDbContext db, Helpers helpers)
```

| Parámetro | Tipo | Descripción |
|---|---|---|
| `db` | `MySqlDbContext` | Contexto de base de datos |
| `helpers` | `Helpers` | Instancia para renderizar tablas y mensajes en consola |

---

## Método principal

```csharp
public void Handle(string queryOpt)
```

Punto de entrada único del menú. Recibe el string con la opción elegida por el usuario y ejecuta el caso correspondiente.

| Parámetro | Tipo | Descripción |
|---|---|---|
| `queryOpt` | `string` | Opción del menú (`"1"` a `"15"`, o `"0"` para volver) |

**Retorna:** `void`. Todo el output va a consola.

---

## Detalle de cada opción

### Opción `"1"` — Listar todos los dinosaurios

```csharp
case "1":
    _helpers.ShowDinosaurs(dinosaursAll);
```

Muestra la tabla completa con todos los dinosaurios. No requiere input adicional del usuario.

**Columnas mostradas:** Id · Username · Species · Age · Type · Zone · Sector · TrackNumber · Address · CreatedAt

---

### Opción `"2"` — Buscar por Id

```csharp
case "2":
    // Lista IDs y usernames disponibles
    // Pide al usuario un ID
    _helpers.ShowDinosaurs(_queryService.GetDinosaurById(dinoId));
```

Primero muestra una tabla auxiliar con `Id`, `Username` y `Species` de todos los dinosaurios. Luego pide que el usuario ingrese un ID y muestra el resultado.

**Input requerido:** Número entero (Id del dinosaurio).

---

### Opción `"3"` — Buscar por Código de Registro

```csharp
case "3":
    // Muestra códigos disponibles numerados
    // Pide al usuario elegir un número
    _helpers.ShowDinosaurs(_queryService.GetDinosaurByCode(codes[codeOpt - 1]));
```

Lista todos los códigos de registro disponibles numerados. El usuario elige por número (no escribe el código directamente).

**Input requerido:** Número entre 1 y la cantidad de códigos disponibles.

---

### Opción `"4"` — Filtrar por Zona

```csharp
case "4":
    // Muestra zonas disponibles numeradas
    _helpers.ShowDinosaurs(_queryService.GetDinosaursByZone(zones[zoneOpt - 1]));
```

Lista las zonas únicas del parque y filtra por la elegida.

**Input requerido:** Número de zona.

---

### Opción `"5"` — Filtrar por Sector

```csharp
case "5":
    // Muestra sectores disponibles numerados
    _helpers.ShowDinosaurs(_queryService.GetDinosaursBySector(sectors[sectorOpt - 1]));
```

Igual que zona, pero para sectores.

**Input requerido:** Número de sector.

---

### Opción `"6"` — Filtrar por Edad (mayor que)

```csharp
case "6":
    int? age = _helpers.IntValidation(Console.ReadLine()!);
    _helpers.InputErrorHandler(age);
    _helpers.ShowDinosaurs(_queryService.GetDinosaursByAge(age));
```

Pide una edad y muestra todos los dinosaurios **mayores** a ese número. Usa `IntValidation` para validar que sea un entero. Si el input no es numérico, `age` es `null` y `InputErrorHandler` imprime un aviso.

**Input requerido:** Número entero positivo.

:::caution Comportamiento
El filtro es `Age > age` (estrictamente mayor). Un dinosaurio de exactamente la edad ingresada **no aparece** en el resultado.
:::

---

### Opción `"7"` — Filtrar por Tipo

```csharp
case "7":
    // Mapea "1" → "Herbivore", "2" → "Carnivore"
    _helpers.ShowDinosaurs(_queryService.GetDinosaursByType(dinoType));
```

Presenta dos opciones fijas: `1. Herbivore` / `2. Carnivore`. Cualquier otra entrada cancela la operación.

**Input requerido:** `"1"` o `"2"`.

---

### Opción `"8"` — Reporte Nombre + Código

```csharp
case "8":
    _helpers.ShowDinosaursforReports(_queryService.GetDinosaursForReports());
```

Muestra una tabla simplificada solo con **Nombre** y **Código de Registro**, útil para reportes rápidos de inventario.

**Formato de salida:**
```
Name                Code
----------------------------------------
Rex                 REX-001
Velociraptor Blue   VELO-002
----------------------------------------
Total: 2
```

---

### Opción `"9"` — Contar total de dinosaurios

```csharp
case "9":
    _helpers.CountDinosaurs(dinosaursAll);
```

Imprime el conteo total de dinosaurios registrados en el sistema.

**Salida:** `Total: N`

---

### Opción `"10"` — Contar por Zona

```csharp
case "10":
    _helpers.CountDinosaurs(_queryService.GetDinosaursByZone(zones[zoneOpt - 1]));
```

Igual que la opción 4 pero en vez de mostrar la tabla, imprime solo el conteo.

---

### Opción `"11"` — Contar por Sector

```csharp
case "11":
    _helpers.CountDinosaurs(_queryService.GetDinosaursBySector(sectors[sectorOpt - 1]));
```

Igual que la opción 5 pero solo muestra el conteo.

---

### Opción `"12"` — Sin número de rastreo

```csharp
case "12":
    _helpers.ShowDinosaurs(_queryService.GetDinosaursWoTracking());
```

Lista los dinosaurios que tienen `TrackNumber = null`. Útil para identificar animales sin seguimiento GPS o dispositivo de monitoreo.

---

### Opción `"13"` — Sin dirección

```csharp
case "13":
    _helpers.ShowDinosaurs(_queryService.GetDinosaursWoAddress());
```

Lista los dinosaurios que tienen `Address = null`. Útil para identificar registros incompletos.

---

### Opción `"14"` — Ordenar por fecha de creación

```csharp
case "14":
    _helpers.ShowDinosaurs(_queryService.OrderByCreationDate());
```

Muestra todos los dinosaurios ordenados por `CreatedAt` ascendente (el más antiguo primero).

---

### Opción `"15"` — Ordenar por Especie

```csharp
case "15":
    _helpers.ShowDinosaurs(_queryService.DinosaursBySpecies());
```

Muestra todos los dinosaurios ordenados alfabéticamente por `DinoSpecies` (A → Z).

---

### Opción `"0"` — Volver

Retorna al menú principal sin realizar ninguna acción.

---

## Diagrama de flujo general

```
Handle(queryOpt)
    │
    ├─ Carga previa (siempre al entrar):
    │     codes ← GetCodes()
    │     zones ← GetZones()
    │     sectors ← GetSectors()
    │     dinosaursAll ← GetAllDinosaurs()
    │
    └─ switch(queryOpt)
          ├─ "1"  → ShowDinosaurs(all)
          ├─ "2"  → pide ID → ShowDinosaurs(byId)
          ├─ "3"  → elige código → ShowDinosaurs(byCode)
          ├─ "4"  → elige zona → ShowDinosaurs(byZone)
          ├─ "5"  → elige sector → ShowDinosaurs(bySector)
          ├─ "6"  → pide edad → ShowDinosaurs(byAge)
          ├─ "7"  → elige tipo → ShowDinosaurs(byType)
          ├─ "8"  → ShowDinosaursForReports()
          ├─ "9"  → CountDinosaurs(all)
          ├─ "10" → elige zona → CountDinosaurs(byZone)
          ├─ "11" → elige sector → CountDinosaurs(bySector)
          ├─ "12" → ShowDinosaurs(woTracking)
          ├─ "13" → ShowDinosaurs(woAddress)
          ├─ "14" → ShowDinosaurs(byDate)
          ├─ "15" → ShowDinosaurs(bySpecies)
          └─ "0"  → return
```

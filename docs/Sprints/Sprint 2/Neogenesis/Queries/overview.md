---
id: queries-overview
title: Sistema de Consultas — Overview
sidebar_label: Overview
sidebar_position: 1
---

# Sistema de Consultas

El módulo **Query** permite recuperar y filtrar dinosaurios desde la base de datos MySQL. Está compuesto por tres archivos con responsabilidades separadas.

---

## Archivos del módulo

| Archivo | Responsabilidad |
|---|---|
| `QueryService.cs` | Acceso a datos — ejecuta las consultas contra la BD |
| `QueryMenu.cs` | Orquestación — maneja la entrada del usuario y llama a `QueryService` |
| `Queries.cs` | Queries auxiliares sobre listas en memoria (capa opcional) |

---

## Flujo de ejecución

```
Program.cs (menú principal)
    │
    └─► QueryMenu.Handle(opción)
            │
            └─► QueryService.[método]()
                    │
                    └─► MySqlDbContext.Dinosaurs
                            │
                            └─► MySQL (tabla Dinosaurs)
```

---

## Resumen de consultas disponibles

| # | Opción del menú | Método en QueryService | Descripción breve |
|---|---|---|---|
| 1 | List all dinosaurs | `GetAllDinosaurs()` | Todos los registros |
| 2 | Get by Id | `GetDinosaurById(id)` | Uno por ID numérico |
| 3 | Get by Register Code | `GetDinosaurByCode(code)` | Uno por código único |
| 4 | Filter by Zone | `GetDinosaursByZone(zone)` | Filtro por zona del parque |
| 5 | Filter by Sector | `GetDinosaursBySector(sector)` | Filtro por sector |
| 6 | Filter by Age | `GetDinosaursByAge(age)` | Mayores a una edad |
| 7 | Filter by Type | `GetDinosaursByType(type)` | Herbívoros o Carnívoros |
| 8 | Name + Register Code | `GetDinosaursForReports()` | Vista resumida para reportes |
| 9 | Count total | `GetAllDinosaurs()` + count | Total de dinosaurios |
| 10 | Count by Zone | `GetDinosaursByZone(zone)` + count | Cantidad en una zona |
| 11 | Count by Sector | `GetDinosaursBySector(sector)` + count | Cantidad en un sector |
| 12 | Without TrackNumber | `GetDinosaursWoTracking()` | Sin número de rastreo |
| 13 | Without Address | `GetDinosaursWoAddress()` | Sin dirección registrada |
| 14 | By creation date | `OrderByCreationDate()` | Ordenados por fecha de registro |
| 15 | Order by Species | `DinosaursBySpecies()` | Ordenados alfabéticamente por especie |

---

## Estrategia de carga

Todas las consultas usan el método privado `All()` como punto de entrada:

```csharp
private List<Dinosaur> All() => _db.Dinosaurs.ToList();
```

:::info Nota de rendimiento
El método `All()` trae **todos los registros** a memoria antes de filtrar. Para colecciones pequeñas esto es aceptable, pero para datasets grandes se recomienda migrar los filtros a consultas LINQ-to-SQL directas con `.Where()` antes de `.ToList()`.
:::

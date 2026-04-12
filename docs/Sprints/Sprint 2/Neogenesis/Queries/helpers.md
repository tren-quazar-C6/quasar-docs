---
id: helpers
title: Helpers — Utilidades de consola
sidebar_label: Helpers
sidebar_position: 4
---

# `Helpers` — Utilidades de consola

**Ubicación:** `NeoGenesis/Shared/Helpers.cs`

Clase de utilidades compartida por todos los módulos. Concentra el renderizado de tablas, mensajes de color y validaciones de input.

---

## Métodos de visualización

### `ShowDinosaurs(List<Dinosaur> dinosaurs)`

Renderiza una tabla tabulada en consola con los campos principales de cada dinosaurio.

**Columnas:**

| Columna | Ancho | Campo fuente |
|---|---|---|
| Id | 5 chars | `d.Id` |
| Username | 15 chars | `d.Username` |
| Species | 20 chars | `d.DinoSpecies` |
| Age | 6 chars | `d.Age` o `"N/A"` |
| Type | 12 chars | `d.Type` o `"N/A"` |
| Zone | 10 chars | `d.Zone` o `"N/A"` |
| Sector | 10 chars | `d.Sector` o `"N/A"` |
| Tracknumber | 15 chars | `d.TrackNumber` o `"N/A"` |
| Address | 12 chars | `d.Address` o `"N/A"` |
| CreatedAt | 10 chars | `d.CreatedAt` |

---

### `ShowDinosaursforReports(List<string> dinosaurs)`

Renderiza la tabla simplificada para reportes (opción 8 del menú).

```
Name                Code
----------------------------------------
Rex                 REX-001
----------------------------------------
Total: 1
```

---

### `CountDinosaurs(List<Dinosaur> dinosaurs)`

Imprime el total de elementos en la lista.

```
Total: 42
```

---

## Métodos de mensajería

### `PrintSuccess(string message)` — estático

Imprime el mensaje en **verde** con ícono ✔.

```
✔ Dino registered successfully. ID asignado: 5
```

### `PrintError(string message)` — estático

Imprime el mensaje en **rojo** con ícono ✖.

```
✖ El identificador 'rex01' ya está registrado en el sistema.
```

### `PrintInfo(string message)` — estático

Imprime el mensaje en **cyan**, con dos espacios de sangría.

```
  Edad se mantiene sin cambios.
```

---

## Métodos de validación e interacción

### `IntValidation(string value)`

```csharp
public int? IntValidation(string value)
```

Intenta parsear `value` como entero. Retorna el número si es válido, o `null` si no.

| Entrada | Retorno |
|---|---|
| `"25"` | `25` |
| `"abc"` | `null` |
| `""` | `null` |

### `InputErrorHandler(dynamic value)`

Si `value` es `null`, imprime `"Invalid input"` en consola.

### `Hold()`

Pausa la ejecución hasta que el usuario presione una tecla. Imprime `"Press any key to continue..."` y luego limpia la pantalla.

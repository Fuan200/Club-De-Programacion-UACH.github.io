---
title: Introducción a la Programación con Go parte 3
date: 2026-10-18
---

## Estructuras de datos y métodos

En la [parte 2](/blog/introduccion-a-la-programacion-con-go-2) se registraron gastos como una lista de números.
Pero un gasto real puede tener más información como una descripción, una cantidad y una categoría.

Se podrían usar varios slices en paralelo (uno para descripciones, otro para cantidades, otro para categorías),
pero mantenerlos sincronizados se vuelve difícil rápidamente. Lo que se puede hacer en su lugar
es agrupar los datos relacionados en un solo valor.

## Structs

Un `struct` define un tipo propio que agrupa varios _campos_, cada uno con su nombre y su tipo:

```go
type Gasto struct {
	Descripcion string
	Cantidad    float64
	Categoria   string
}
```

`type Gasto struct` crea un nuevo tipo llamado `Gasto`. A partir de ese tipo se pueden crear valores
(también llamados _instancias_):

```go
gasto := Gasto{
	Descripcion: "Libros",
	Cantidad:    450.50,
	Categoria:   "Educación",
}

fmt.Println(gasto.Descripcion) // Libros
fmt.Println(gasto.Cantidad)    // 450.5
```

Los campos se consultan y modifican con un punto: `gasto.Cantidad`.

El tipo `Gasto` es la descripción de cómo se ve un gasto; `gasto` es un gasto en particular.
Del mismo modo que `int` es un tipo y `20` es un valor de ese tipo.

Los campos que no se especifican reciben su _valor cero_, igual que las variables:

```go
var g Gasto
fmt.Println(g.Cantidad) // 0
```

## Métodos

Un método es una función asociada a un tipo:

```go
func (g Gasto) EsCaro() bool {
	return g.Cantidad > 1000
}
```

Lo que aparece entre paréntesis antes del nombre, `(g Gasto)`, es el _receptor_: el valor sobre el
que se llama el método. Se usa con un punto, igual que un campo:

```go
if gasto.EsCaro() {
	fmt.Println("Este gasto supera los $1000")
}
```

Es equivalente a una función normal que recibe el gasto como argumento, `esCaro(gasto)`, pero el
método deja claro que esa operación le pertenece a `Gasto`.

## Maps

Un map asocia _claves_ con _valores_. A diferencia de un slice, donde se accede por posición (`0`, `1`, `2`...),
en un map se accede por clave:

```go
gastosPorCategoria := map[string]float64{
	"Comida":     800,
	"Transporte": 350,
	"Educación":  450,
}

gastosPorCategoria["Comida"] += 100
gastosPorCategoria["Salud"] = 200

fmt.Println(gastosPorCategoria["Comida"]) // 900
```

`map[string]float64` es un map cuyas claves son `string` y cuyos valores son `float64`.
Asignar a una clave que no existe la agrega; asignar a una que ya existe la reemplaza.

Consultar una clave que no existe devuelve el valor cero del tipo, por lo que `0` no indica si la clave estaba o no.
Para saberlo se usa una segunda variable:

```go
cantidad, ok := gastosPorCategoria["Ropa"]
if !ok {
	fmt.Println("No hay gastos en Ropa")
}
```

Y `range` recorre las claves y valores:

```go
for categoria, cantidad := range gastosPorCategoria {
	fmt.Println(categoria, cantidad)
}
```

> El orden al recorrer un map no está garantizado: puede cambiar entre ejecuciones.

Un slice conviene cuando importa el orden o se trabaja con una secuencia de elementos;
un map conviene cuando se necesita buscar un valor a partir de una clave.

## Ejemplo

### Resumen de gastos por categoría

Este programa combina structs, métodos, slices y maps: guarda varios gastos, marca los que superan un límite
y calcula el total de cada categoría.

```go
package main

import "fmt"

type Gasto struct {
	Descripcion string
	Cantidad    float64
	Categoria   string
}

func (g Gasto) SuperaLimite(limite float64) bool {
	return g.Cantidad > limite
}

func totalesPorCategoria(gastos []Gasto) map[string]float64 {
	totales := map[string]float64{}

	for _, g := range gastos {
		totales[g.Categoria] += g.Cantidad
	}

	return totales
}

func main() {
	gastos := []Gasto{
		{Descripcion: "Libros", Cantidad: 450.50, Categoria: "Educación"},
		{Descripcion: "Camión", Cantidad: 12, Categoria: "Transporte"},
		{Descripcion: "Súper", Cantidad: 1250, Categoria: "Comida"},
		{Descripcion: "Tacos", Cantidad: 90, Categoria: "Comida"},
		{Descripcion: "Gasolina", Cantidad: 600, Categoria: "Transporte"},
	}

	const limite = 500.0

	fmt.Println("Gastos:")
	for _, g := range gastos {
		aviso := ""
		if g.SuperaLimite(limite) {
			aviso = "  <- supera $500"
		}
		fmt.Printf("  %-10s %-11s $%8.2f%s\n", g.Descripcion, g.Categoria, g.Cantidad, aviso)
	}

	fmt.Println("\nTotal por categoría:")
	total := 0.0
	for categoria, suma := range totalesPorCategoria(gastos) {
		fmt.Printf("  %-11s $%8.2f\n", categoria, suma)
		total += suma
	}

	fmt.Printf("\nTotal: $%.2f\n", total)
}
```

`totales[g.Categoria] += g.Cantidad` funciona aunque la categoría todavía no exista en el map:
la consulta devuelve `0` y se le suma la cantidad.

Al ejecutarlo varias veces, el orden de "Total por categoría" puede cambiar, porque Go no garantiza
el orden al recorrer un map.

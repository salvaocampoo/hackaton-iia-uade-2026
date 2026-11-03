# M03 · ¿Este pedido es fraudulento?

**Integrantes:** Salvador Ocampo
**IA declarada:** Claude
**Pista:** Reglas
**Publicación en LinkedIn:** <link>

---

## 1. Formulación y PEAS

**La pregunta.** Para cada pedido, ¿es fraudulento? SÍ significa: el pedido es fraudulento (compra con tarjeta robada).

**Objetivo (distinto de la métrica).** Evitar las pérdidas por fraude sin frenar a clientes legítimos. La métrica es solo su traducción numérica: sensibilidad (prioritaria), precisión y especificidad.

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | Sensibilidad (fraudes detectados sobre fraudes reales) como métrica principal, acompañada de precisión y especificidad; se compara contra una línea base trivial |
| Entorno | Pedidos de una tienda online, evaluados uno por uno antes de despachar. Parcialmente observable, estocástico, episódico |
| Actuadores | Frenar el pedido (SÍ) o despacharlo (NO) |
| Sensores | Monto del pedido, antigüedad de la cuenta, distancia envío–facturación |

## 2. El sistema

Datos que usa: distancia envío–facturación (el monto y la antigüedad se observaron pero no entran en la regla final). NO usa "Contracargo registrado": ocurre después de despachar, no se conoce al decidir (fuga de información). De hecho coincide con el resultado real en los 20 casos.

Hecho observable (umbral fijado mirando solo los casos 1 a 14):
- distancia_alta: distancia envío–facturación ≥ 450 km.

Regla:
- R1: SI distancia_alta ENTONCES responder SÍ (fraudulento); si no, NO.

Cómo correrlo en vivo: tomar la distancia del pedido, compararla con 450 km y responder. Ejemplo: 1.121 km → SÍ; 36 km → NO.

Por qué 450: los 6 fraudes del entrenamiento tienen 485 km o más; los legítimos están casi todos en 425 km o menos (salvo 2 casos).

## 3. Medición

Filas = clase real, columnas = clase predicha. Clase positiva = fraude.

**Entrenamiento (casos 1 a 14)**

| | Predicho SÍ | Predicho NO | Total |
|---|---|---|---|
| Real SÍ | VP = 6 | FN = 0 | 6 |
| Real NO | FP = 2 | VN = 6 | 8 |

6 + 0 + 2 + 6 = 14. Exactitud 85,7 % · Precisión 75 % · Sensibilidad 100 % · Especificidad 75 %.
Falsos positivos: casos 9 y 10 (legítimos con 701 y 619 km).

**Prueba (casos 15 a 20, medida una sola vez)**

| | Predicho SÍ | Predicho NO | Total |
|---|---|---|---|
| Real SÍ | VP = 2 | FN = 2 | 4 |
| Real NO | FP = 1 | VN = 1 | 2 |

2 + 2 + 1 + 1 = 6. Exactitud 50 % · Precisión 66,7 % · Sensibilidad 50 % · Especificidad 50 %.
Errores: FN en casos 18 y 20 (fraudes con 317 y 307 km), FP en el caso 19 (legítimo con 522 km).

**Entrenamiento contra prueba.** El sistema rinde bastante peor con casos nuevos (sensibilidad 100 % → 50 %). Con solo 14 casos de entrenamiento y 6 de prueba, el umbral se ajustó a los pocos casos vistos y la estimación es muy ruidosa: es sobreajuste y tamaño de muestra chico. No se tocó el umbral después de ver la prueba.

## 4. El error más caro

El falso negativo: despachar un pedido fraudulento (se pierde la mercadería y el banco cobra el contracargo). El falso positivo (frenar un pedido legítimo) cuesta un cliente enojado y una venta perdida, pero es menos grave.

Qué cambiamos por él: se descartó el diseño A (distancia alta Y cuenta reciente), que daba 100 % en entrenamiento pero dejaba pasar más fraudes al exigir dos condiciones. Se eligió una sola condición (distancia), aceptando más falsos positivos a cambio de mayor sensibilidad.

## 5. ¿Comparado contra qué?

**Línea base trivial:** responder siempre NO (clase mayoritaria del entrenamiento). Exactitud 57,1 % en entrenamiento y 33,3 % en prueba, con sensibilidad 0 %. Nuestro sistema la supera, pero la exactitud no es la métrica principal: lo es la sensibilidad.

**Checklist de la Clase 6:**
- ¿Qué datos? 20 casos inventados de la ficha M03; muy pocos y no representan una tienda real.
- ¿Qué métrica? Sensibilidad, con precisión y especificidad.
- ¿Comparado contra qué? Contra decir siempre NO.
- ¿Quién lo midió? El propio grupo, sobre una prueba de 6 casos: la evidencia es débil.

## 6. Riesgo

Datos → modelo → decisión → consecuencia.
- Datos: 14 casos de entrenamiento, uno de ellos con una columna (contracargo) que filtraría la respuesta.
- Modelo: una regla de umbral en la distancia, sobreajustada a esos casos.
- Decisión: frenar o despachar según la distancia, sin mirar el monto ni la antigüedad.
- Consecuencia: fraudes con distancia corta pasan sin freno, y clientes legítimos que viven lejos de su dirección de facturación son frenados sin motivo.

## 7. Bitácora

**Qué se le pidió a la IA (Claude):** leer la ficha M03 y el CSV, proponer umbrales usando solo los casos de entrenamiento, calcular las matrices y redactar el README.
**Qué se aceptó:** la regla de distancia ≥ 450 km, la lectura del error más caro y las matrices.
**Qué se rechazó y por qué:** usar la columna "Contracargo registrado" (fuga de información) y el diseño A (menor sensibilidad).

**Corridas contra el evaluador:**
- Corrida 1 · Nota 9,5/10 · Agujero: ¿qué cambio concreto se hizo para reducir falsos negativos y dónde se muestra con datos? · Qué cambiamos: se explicó que se eligió el diseño B por el costo del error (el falso negativo es el más caro) y porque A exige dos condiciones.

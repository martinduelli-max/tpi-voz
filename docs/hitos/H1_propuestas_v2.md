# Hito 1 — Propuestas v2 (después de la consulta con el tutor)

> Devolución del tutor (19/10/2026): las propuestas anteriores no estaban acotadas al marco del proyecto. El trabajo tiene que
> centrarse en **codificación y decodificación de voz**, siguiendo el capítulo de Dutoit, Moreau y Kroon, *"How is speech
> processed in a cell phone conversation?"*. La versión anterior queda como historial en [H1_propuestas.md](H1_propuestas.md).

Las tres propuestas siguen la estructura del capítulo: modelo LP (1.1.1), estimación por Yule-Walker / Levinson-Durbin
(1.1.2), procesamiento por tramas (1.1.3), códecs LPC10 y CELP (1.1.4) y la prueba de concepto paso a paso (1.2). En todas
se traduce el código MATLAB del capítulo a Python y se le agrega una parte que el capítulo deja abierta.

---

## Propuesta 1 — Vocoder LPC10 completo con cuantización de parámetros (2,4 kbps)

**Problema:** transmitir voz por canales de muy bajo ancho de banda (satélite, radio, telemedicina en zonas rurales) exige
mandar parámetros del modelo de producción de la voz en lugar de las muestras. El capítulo implementa el análisis-síntesis
LPC10 pero aclara que **no cuantiza los coeficientes**, que es justamente lo que define la tasa de bits real.

**Solución:** implementar en Python el codificador y decodificador LPC10 completo: análisis por tramas, Levinson-Durbin,
estimación de pitch y decisión sonoro/sordo con la autocorrelación del residuo, y **cuantización** de los parámetros con la
asignación del estándar (54 bits cada 22,5 ms: 7 de pitch y V/UV, 5 de ganancia, 42 de coeficientes). Comparar cuantizar los
aᵢ directamente contra coeficientes de reflexión y *log area ratios*, mostrando que los aᵢ cuantizados pueden dar filtros
inestables (polos fuera del círculo unitario).

**Fortaleza:** es el núcleo del capítulo, con el código guía disponible, y se defiende con AR/IIR, FIR inverso, transformada
Z, polos y estabilidad.
**Debilidad:** LPC10 no conserva la forma de onda, así que la SNR no sirve para medir calidad; hay que usar distancia espectral,
PESQ o escucha.

---

## Propuesta 2 — De LPC10 a CELP: compromiso entre tasa de bits y calidad

**Problema:** los celulares actuales no usan LPC10 (voz "robótica") sino códecs CELP, que mejoran mucho la calidad a costa
de más bits y más cómputo. Elegir un códec es elegir un punto de ese compromiso.

**Solución:** implementar el CELP del capítulo (libro de códigos estocástico, búsqueda por análisis-síntesis, predictor de
largo plazo y filtro perceptual A(z)/A(z/γ)) y barrer sus parámetros (componentes por trama, tamaño del libro, largo de
trama, γ, con y sin predictor de largo plazo). Calcular la tasa de bits de cada configuración y su calidad, y construir la
**curva tasa–calidad** comparada con LPC10 (2,4 kbps) y PCM (64 kbps).

**Fortaleza:** sigue el capítulo de punta a punta; el resultado (una curva) es claro y defendible; usa proyecciones y producto
interno, filtros con memoria entre tramas y el efecto de mover los polos con γ.
**Debilidad:** CELP es la parte más trabajosa y lenta de cómputo; conviene llegar al MVP con LPC10 y sumar CELP después.

---

## Propuesta 3 — Estimación de pitch y decisión sonoro/sordo para el vocoder LPC10, validada con laringógrafo

**Problema:** el capítulo advierte que LPC10 es muy sensible a errores de F0 y de decisión sonoro/sordo, y que en voces
femeninas el segundo armónico cae sobre el primer formante y se confunde con F0. Esos errores son la principal causa de la
mala calidad del vocoder.

**Solución:** comparar el estimador del capítulo (máximo de la autocorrelación del residuo LPC) contra alternativas
(cepstrum, AMDF/YIN) usando la base PTDB-TUG, que trae la señal de un **laringógrafo** (sensor de contacto de las cuerdas
vocales) como referencia. Medir el error de pitch y de V/UV por sexo, y cuánto mejora la voz decodificada por LPC10 al
cambiar el estimador.

**Fortaleza:** tiene referencia de verdad (laringógrafo) y métricas claras; ataca una debilidad que el propio capítulo
señala; el laringógrafo le da el componente biomédico.
**Debilidad:** se concentra en una parte del códec y no en el códec completo; hay que remuestrear la base de 48 kHz a 8 kHz.

---

## Datos y métricas comunes

| Recurso | Para qué |
|---|---|
| Frases limpias de [NOIZEUS](https://ecs.utdallas.edu/loizou/speech/noizeus/) (8 kHz, banda telefónica) | Señales de prueba a la misma fs que el capítulo |
| [PTDB-TUG](https://www.spsc.tugraz.at/databases-and-tools/ptdb-tug-pitch-tracking-database-from-graz-university-of-technology.html) (voz + laringógrafo) | Referencia de pitch y V/UV (P3), voces de hombres y mujeres |
| [CMU ARCTIC](http://festvox.org/cmu_arctic/) | Más voz limpia; remuestreo a 8 kHz |
| Grabaciones propias | La guía valora la adquisición propia |

Métricas: tasa de bits; SNR y SNR segmental (solo tiene sentido en CELP, que intenta copiar la forma de onda); distancia
espectral logarítmica o de Itakura-Saito entre envolventes LPC; PESQ (paquete `pesq`, banda angosta a 8 kHz); prueba de
escucha con compañeros.

Variante con componente biomédico (opcional, si el tutor la acepta como caso de prueba): pasar voces sanas y patológicas
(VOICED, 8 kHz) por los códecs y ver si se conservan los parámetros clínicos.

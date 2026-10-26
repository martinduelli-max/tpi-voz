# Hito 1 — Lluvia de ideas y 3 propuestas (borrador para la Reunión 01)

> Borrador de trabajo para discutir en grupo. **No es la decisión final**: la decide el grupo en la reunión y la valida el tutor.
> Entregable del hito: 1 carilla con las 3 ideas → el tutor elige 1.

**Paper base:** Dutoit, T., Moreau, N. & Kroon, P. "How is speech processed in a cell phone conversation?", cap. 1 de
*Applied Signal Processing: A MATLAB-Based Proof of Concept* (Springer, 2009). Resumen en
[docs/teoria/resumen-voz.md](../teoria/resumen-voz.md#7-paper-base-how-is-speech-processed-in-a-cell-phone-conversation).

En una línea: el celular no transmite la voz, transmite **los parámetros del modelo fuente-filtro** (coeficientes LPC del
tracto vocal + excitación) cada 10–20 ms. El paper dice explícitamente que **jitter y shimmer** hacen difícil estimar el pitch
y que el códec LPC10 es sensible a errores de sonoro/sordo y de F0 → ese es el puente natural entre el paper y un problema
**clínico** de voz.

Criterios: **ni muy fácil ni muy difícil**, base de datos **pública**, núcleo técnico = el del paper (LPC por Levinson-Durbin,
filtrado inverso, síntesis), y defendible con los temas de la materia (DFT, ventanas, LTI, FIR/IIR/AR, deconvolución, Z, polos).

---

## 1. Lluvia de ideas (pre-cargada, sumar las del grupo)

| # | Idea | Base posible | Comentario rápido |
|---|---|---|---|
| A | **Parámetros del códec LPC como indicadores de disfonía** (idea 1 de Pau) | SVD / VOICED | Usa el paper tal cual y le agrega la pregunta clínica |
| B | **Teleconsulta por celular: ¿el códec (LPC10/CELP) y el ruido borran la información clínica de la voz?** | VOICED/SVD + ruidos NOIZEUS | Implementa los dos códecs del paper; cae en lo que la cátedra sugirió ("rasgos robustos al ruido") |
| C | **Monitoreo de voz en Parkinson guardando parámetros, no audio** (idea 2 de Pau, versión app) | Italian PVS / MDVR-KCL | Muy clínica; más riesgo (habla continua, bases chicas) |
| D | Comparar estimadores de F0 (autocorrelación del residuo LPC como en el paper, cepstrum, YIN) contra laringógrafo | PTDB-TUG | El paper avisa que en voces femeninas el 2º armónico se confunde con F0: se puede medir |
| E | Espacio vocálico con formantes desde los polos LPC (Parkinson vs control) | Italian PVS (vocales a-e-i-o-u) | Directo del GTP6 (polos ↔ picos de \|H\|) |
| F | Cepstral Peak Prominence (CPP) como marcador de disfonía | SVD | Cepstro = deconvolución homomórfica |
| G | Sana vs patológica con jitter/shimmer/HNR | VOICED | Tema muy trabajado; necesitaría una pregunta propia |
| H | Disartria: velocidad de habla e inteligibilidad | TORGO | Datos más complejos |
| I | Detección de eventos de tos | COUGHVID / Coswara | Se aleja del paper |
| J | Emoción/estrés en voz | RAVDESS | Poco clínico |

---

## 2. Tres propuestas sugeridas (orden de preferencia sugerido)

Las tres comparten el **mismo núcleo técnico del paper** (análisis LPC por tramas + filtrado inverso + síntesis), así que el
trabajo de código del inicio sirve para cualquiera de ellas.

### Propuesta 1 — "El celular manda una foto del tracto vocal": parámetros LPC como indicadores de disfonía

**Problema clínico.** Pacientes con trastornos vocales (nódulos, pólipos, disfonía funcional, parálisis de cuerda vocal) tienen
alterada la periodicidad y el timbre de la voz. En telemedicina, mandar audio crudo consume ancho de banda y no entrega
indicadores objetivos para seguir la evolución del tratamiento.

**Idea.** Implementar el analizador LPC del paper y usar sus propios parámetros como indicadores clínicos compactos.
**Hipótesis:** el modelo LPC supone una excitación "binaria" (tren de pulsos o ruido). Una voz sana se ajusta bien; una voz
patológica (soplada, áspera) tiene excitación mixta y ciclos irregulares (jitter, shimmer) → **más energía de residuo,
menor ganancia de predicción, pico de autocorrelación del residuo más bajo (menos sonoridad), más errores sonoro/sordo y
peor reconstrucción**.

**Datos.** Saarbrücken Voice Database (vocal /a/ en tono normal; sanos vs disfonía funcional, nódulos, parálisis), o VOICED
(PhysioNet, 8 kHz como en el paper).

**Diagrama de bloques preliminar.**
señal → pasa-altos (quitar DC) → tramas de 30 ms, salto 10 ms, ventana Hamming → autocorrelación → **Levinson-Durbin (p = 10 a
8 kHz)** → A(z) → **filtrado inverso** (residuo) → F0 y decisión sonoro/sordo desde la autocorrelación del residuo →
síntesis LPC10 → métricas.

**Métricas.** Ganancia de predicción y energía del residuo; proporción de tramas sonoras; distancia espectral entre original y
reconstruida; comparación sanos vs patológicos (boxplots + Mann-Whitney; opcional ROC/AUC con un umbral). Tasa de bits del
"reporte paramétrico" vs audio crudo.

**Conceptos de la materia.** Modelo AR (IIR todo-polos) y su inverso FIR; ecuaciones de Yule-Walker (matriz Toeplitz);
transformada Z, polos ↔ formantes; respuesta en frecuencia (`freqz`: envolvente LPC sobre el periodograma); ventanas;
autocorrelación como producto interno; deconvolución.

**Dificultad:** media. **Riesgo:** medio-bajo (el paper trae el código paso a paso en MATLAB; lo traducimos a Python y
validamos con `librosa.lpc`). **MVP:** análisis-síntesis LPC10 + residuo y ganancia en sanos vs patológicos.

---

### Propuesta 2 — Teleconsulta por celular: ¿el códec y el ruido borran la información clínica de la voz?

**Problema clínico.** Cada vez más evaluaciones de voz se hacen a distancia (llamada o app). Pero la voz que llega al médico ya
pasó por el **códec del celular** (que, según el paper, la reemplaza por un modelo con excitación simplificada) y por el
**ruido del ambiente**. Si los parámetros clínicos (F0, jitter, shimmer, HNR) cambian por el canal, una voz sana puede parecer
patológica o al revés. Es el ejemplo de Ishikawa de la guía: *¿el problema es fisiológico, de instrumentación o del entorno?*

**Idea.** Implementar en Python los dos códecs del paper — **LPC10** (pulsos/ruido, 2,4 kbps) y **CELP** (libro de códigos +
predicción de largo plazo + filtro perceptual) — pasar voces sanas y patológicas por ellos (y opcionalmente agregar ruido real
de NOIZEUS a SNR controlada) y medir cuánto se distorsiona cada parámetro clínico. **Hipótesis:** LPC10 destruye jitter,
shimmer y HNR (impone pulsos perfectamente periódicos) mientras que CELP los preserva mejor; F0 sobrevive en ambos.

**Datos.** VOICED o SVD (voces con diagnóstico) + ruidos NOIZEUS.

**Métricas.** Error de cada parámetro vs la señal original; SNR segmental; ¿se mantiene la separación sano/patológico después
del códec? Tasa de bits de cada códec.

**Conceptos de la materia.** Todo lo de P1 + cuantización y tasa de bits, filtro perceptual A(z)/A(z/γ) (escalar polos hacia
adentro del círculo unitario), estado interno de los filtros entre tramas, SNR, proyecciones (búsqueda en libro de códigos =
producto interno).

**Dificultad:** media (CELP es la parte más trabajosa, pero el paper la detalla). **Riesgo:** bajo-medio (control experimental
total: siempre tenemos la señal original).

---

### Propuesta 3 — Monitoreo de voz en Parkinson guardando parámetros, no audio (prototipo de app)

**Problema clínico.** La enfermedad de Parkinson produce **hipofonía** (voz de bajo volumen) y **monotonía** (poca variación
de tono). Monitorear al paciente en el día genera mucho audio y expone su privacidad si se guarda o transmite crudo.

**Idea.** Prototipo en Python del "motor" de una app que hace lo mismo que el celular del paper — guardar **parámetros en
lugar de audio** — pero orientado a la clínica: detecta segmentos con voz (energía de corto término + decisión sonoro/sordo
del paper), guarda por trama F0, ganancia (nivel) y coeficientes LPC, **descarta el audio** y genera un reporte de
variabilidad de F0 (en semitonos), nivel y formantes. **Hipótesis:** los pacientes con Parkinson muestran menor variabilidad
de F0 e intensidad que los controles.

**Datos.** Italian Parkinson's Voice and Speech (IEEE DataPort, CC-BY) y/o MDVR-KCL (Zenodo, grabado con celular).

**Métricas.** Detección de voz contra segmentación manual de algunos archivos (precisión / sensibilidad); diferencias
Parkinson vs control; bytes del reporte vs bytes del audio.

**Conceptos de la materia.** Energía, potencia y RMS; LPC; autocorrelación; ventanas; filtros.

**Dificultad:** media-alta. **Riesgos:** F0 en habla continua es más difícil que en vocal sostenida; el volumen absoluto no se
puede medir en grabaciones sin calibrar (solo dB relativos); bases chicas (MDVR-KCL: 37 personas). La app real queda **fuera
de alcance**: se presenta como prototipo Python que simula el flujo.

---

### Ideas de respaldo (si el tutor rebota alguna)

- **D. Estimadores de F0 vs laringógrafo (PTDB-TUG).** El paper estima F0 con la autocorrelación del residuo LPC y advierte que
  falla en voces femeninas. Se puede comparar con cepstrum y YIN contra la F0 de referencia (Gross Pitch Error).
- **E. Espacio vocálico en Parkinson.** Formantes F1–F2 desde los polos de A(z) para a-e-i-o-u; área del espacio vocálico
  Parkinson vs control.

---

## 3. Comparación rápida

| | P1 LPC → disfonías | P2 Códec + ruido en teleconsulta | P3 Parkinson app |
|---|---|---|---|
| Fidelidad al paper | ⭐⭐⭐ (LPC10) | ⭐⭐⭐ (LPC10 + CELP) | ⭐⭐ (LPC como representación) |
| Datos públicos | ✅ SVD / VOICED | ✅ VOICED/SVD + NOIZEUS | ✅ IPVS / MDVR-KCL (chicas) |
| Vínculo con la materia | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Relevancia clínica | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Dificultad | media | media | media-alta |
| Riesgo de no llegar | medio-bajo | bajo-medio | medio-alto |

P1 y P2 se pueden **encadenar**: P1 es el MVP natural de P2 (primero LPC10, después CELP y la comparación).

---

## 4. Ishikawa preliminar (para la propuesta elegida)

Efecto: **"el parámetro acústico no refleja fielmente el estado de la voz del paciente"**

- **Fisiológico / sujeto:** patología, sexo y edad (F0 distinta), fatiga, hidratación, esfuerzo vocal.
- **Protocolo:** vocal sostenida vs habla continua, intensidad y tono pedidos, duración.
- **Instrumentación / canal:** micrófono, distancia, fs y resolución, control automático de ganancia, **códec del celular**.
- **Entorno:** ruido de fondo, reverberación.
- **Método / algoritmo:** estimador de F0, largo de ventana, orden LPC, umbral sonoro/sordo, software (Praat vs otros dan
  valores distintos — Elisei 2012).
- **Datos:** desbalance de clases, etiquetado clínico, mismo hablante en entrenamiento y prueba (fuga de datos).

---

## 5. Preguntas para el tutor (19/10)

1. ¿Hay que **replicar** el paper (LPC10 y CELP) y después aplicarlo a un problema clínico, o alcanza con inspirarse en él?
2. ¿Se espera un clasificador (aprendizaje automático) o alcanza con análisis + estadística + métricas?
3. ¿Bases que piden registro (IEEE DataPort) o muy grandes (SVD ~38 GB completa, se puede bajar parcial) son aceptables?
4. ¿Está bien usar `librosa` / `parselmouth` (Praat) para **validar** lo que implementamos a mano con NumPy/SciPy?
5. Para la propuesta 3: ¿alcanza con un prototipo en Python que simule la app?

# Procesamiento de voz — resumen teórico y herramientas

Resumen orientado al TPI, conectado con los temas de la materia (GTP0–GTP6). Sirve de base para la justificación teórica del
Hito 2; las afirmaciones hay que respaldarlas con bibliografía (ver final).

---

## 1. Cómo se produce la voz: el modelo fuente-filtro

La voz se modela como la salida de un **sistema LTI** (el tracto vocal) excitado por una **fuente** (la glotis):

```
fuente (glotis) ──► filtro del tracto vocal ──► radiación en los labios ──► s[n]
   e[n]                    V(z)                        R(z)
```

s[n] = e[n] ∗ g[n] ∗ v[n] ∗ r[n]  →  en Z: S(z) = E(z) · G(z) · V(z) · R(z)

- **Fuente.** En sonidos **sonoros** (vocales, /m/, /l/) las cuerdas vocales vibran y generan un tren de pulsos casi periódico
  de frecuencia **F0** (tono). Típico: hombres ≈ 85–180 Hz, mujeres ≈ 165–255 Hz. En sonidos **sordos** (/s/, /f/) la fuente es
  ruido turbulento.
- **Filtro.** El tracto vocal (faringe + boca + nariz) es un tubo resonante. Sus resonancias son los **formantes** F1, F2, F3…
  y definen qué vocal escuchamos. Se modela muy bien como un filtro **todo-polos (AR)**: cada formante es un par de **polos
  complejos conjugados cerca del círculo unitario** — exactamente la "regla geométrica" del GTP6 (polo cerca del círculo →
  pico agudo en |H|).
- **Radiación.** Se aproxima por un derivador (≈ +6 dB/octava).

Consecuencia clave: en el espectro de una vocal se ven dos cosas superpuestas — los **armónicos** (rayas cada F0, vienen de la
fuente) y la **envolvente** (montañas en los formantes, viene del filtro). Separar fuente y filtro es una **deconvolución**:
casi todas las herramientas de voz son formas distintas de hacerlo.

## 2. Características de la señal de voz

- **No estacionaria**, pero **cuasi-estacionaria en tramas de 20–30 ms** → todo se analiza "de corto término": se corta en
  tramas solapadas (salto ≈ 10 ms) con ventana Hamming/Hann. Es el compromiso tiempo–frecuencia del GTP3.
- **Ancho de banda:** telefonía 300–3400 Hz (fs = 8 kHz); banda ancha hasta 7–8 kHz (fs = 16 kHz). Por Nyquist, con fs = 8 kHz
  (VOICED) solo hay información hasta 4 kHz.
- **Vocal sostenida** (/a/ de 3–5 s): casi periódica, es el protocolo clínico estándar para medir perturbaciones de la fuente.
  **Habla continua**: más natural y realista, pero más difícil de analizar.

## 3. Herramientas

### 3.1 Dominio temporal

| Herramienta | Qué es | Para qué |
|---|---|---|
| Energía / RMS de corto término | Σ x²[n] por trama; en dB = 20·log10(RMS) | Intensidad, detección de voz (VAD), hipofonía |
| Tasa de cruces por cero (ZCR) | Cuántas veces cambia de signo por trama | Distinguir sonoro (ZCR baja) de sordo/ruido (ZCR alta) |
| Autocorrelación | r[k] = Σ x[n]·x[n+k]: **producto interno** de la señal con ella misma desplazada | Estimar el período T0 = 1/F0 (primer pico fuera del origen) |
| AMDF / YIN | Diferencia en vez de producto | Estimadores de F0 más robustos (`librosa.pyin` usa YIN probabilístico) |

### 3.2 Dominio frecuencial

- **STFT y espectrograma.** DFT de cada trama ventaneada. **Ventana corta** (≈ 5 ms) → espectrograma de *banda ancha*: se ven
  los formantes. **Ventana larga** (≈ 30–40 ms) → *banda angosta*: se ven los armónicos. Es el compromiso lóbulo principal vs
  resolución del GTP3.
- **Pre-énfasis.** Filtro FIR H(z) = 1 − 0,97·z⁻¹ (pasa-altos suave) que compensa la caída espectral natural de la voz antes de
  LPC.
- **Filtrado.** Pasa-altos ~60–80 Hz para quitar DC y ruido de baja frecuencia. Si interesa la forma de onda ciclo a ciclo
  (jitter, shimmer) conviene **fase cero** (`scipy.signal.filtfilt`) o FIR de fase lineal, para no deformar los ciclos.

### 3.3 Predicción lineal (LPC) — modelo AR del tracto vocal

Idea: cada muestra se predice como combinación lineal de las p anteriores:

ŝ[n] = Σₖ₌₁ᵖ aₖ · s[n−k]     error e[n] = s[n] − ŝ[n]

- Es exactamente un **modelo AR**: H(z) = G / A(z), con A(z) = 1 − Σ aₖ z⁻ᵏ (IIR todo-polos).
- Los aₖ salen de las **ecuaciones de Yule-Walker** (matriz de autocorrelación Toeplitz), resueltas con el algoritmo de
  **Levinson-Durbin** (`scipy.linalg.solve_toeplitz`, o `librosa.lpc` para validar).
- **Orden típico:** p ≈ fs/1000 + 2 (≈ 10 a 8 kHz, ≈ 18 a 16 kHz).
- **Filtrado inverso:** pasar la señal por A(z) (un **FIR**) da el **residuo** ≈ la excitación glótica → es una
  **deconvolución**.
- **Formantes:** raíces de A(z) → ángulo del polo = frecuencia, distancia al círculo = ancho de banda.
- **|1/A(e^{jω})|** es la envolvente espectral suave.
- **Codificación de voz (vocoder LPC).** En vez de mandar muestras (PCM 8 kHz × 8 bits = 64 kbps) se mandan, cada ~20 ms:
  coeficientes, ganancia, F0 y sonoro/sordo. El LPC-10 clásico funciona a **2,4 kbps**. El receptor sintetiza excitando
  1/A(z) con tren de pulsos (sonoro) o ruido (sordo).

### 3.4 Cepstrum — deconvolución homomórfica

c[n] = IDFT{ log |DFT{x[n]}| }

- El logaritmo convierte el **producto** de espectros (fuente × filtro) en **suma** → la convolución se vuelve suma y fuente y
  filtro se separan en el eje de *quefrencia*.
- Quefrencias bajas → envolvente (tracto vocal). Pico en quefrencia = T0 → periodicidad (fuente) → estimador de F0.
- **CPP (Cepstral Peak Prominence):** altura de ese pico sobre la tendencia; marcador clínico de disfonía robusto, sirve
  también en habla continua.
- **MFCC:** banco de filtros en escala mel + log + DCT. Son los descriptores estándar para clasificación y reconocimiento.

### 3.5 Parámetros clínicos de calidad vocal

| Parámetro | Mide | Notas |
|---|---|---|
| F0 media y desvío | Tono y su estabilidad | Desvío en semitonos para monotonía (Parkinson) |
| **Jitter** | Variación del período **ciclo a ciclo** | Jitter local = media\|Tᵢ − Tᵢ₊₁\| / media(T). Variantes RAP, PPQ5 (Farrús 2007) |
| **Shimmer** | Variación de la amplitud ciclo a ciclo | En % o dB. Variantes APQ3, APQ5, APQ11 |
| **HNR** | Energía armónica / energía de ruido (dB) | Voz sana típicamente > 20 dB |
| CPP | Prominencia del pico cepstral | Más robusto que jitter en voces muy alteradas |

Valores de referencia "normales" muy citados (Praat/MDVP): jitter local < ~1 %, shimmer local < ~3,8 %. **Ojo:** distintos
programas dan valores distintos para la misma voz y la confiabilidad cae en voces muy patológicas (Elisei 2012, Praat vs
Anagraf). Jitter y shimmer necesitan **detectar cada ciclo**; medir variación entre tramas de 64 ms no es jitter.

### 3.6 Prosodia (habla continua)

Contorno de F0, contorno de intensidad, velocidad de habla, pausas. En Parkinson: menor variación de F0 (monotonía), menor
intensidad (hipofonía), pausas alteradas.

### 3.7 Realce / reducción de ruido

- **Sustracción espectral:** estimar el espectro del ruido en tramas sin voz y restarlo en magnitud, conservar la fase y
  reconstruir con IFFT + overlap-add.
- **Filtro de Wiener:** ganancia por frecuencia SNR/(1+SNR).
- **Métricas:** SNR, SNR segmental; calidad perceptual PESQ / inteligibilidad STOI (paquetes `pesq`, `pystoi`).

### 3.8 Decisión y validación

- Comparación entre grupos: boxplots, test de Mann-Whitney / t de Student.
- Clasificadores simples (`scikit-learn`): umbral, kNN, regresión logística, SVM.
- Métricas: matriz de confusión, sensibilidad, especificidad, ROC/AUC, F1.
- **Validación cruzada separando por hablante:** si el mismo paciente aparece en entrenamiento y prueba, el resultado se infla
  (fuga de datos).

## 4. Valores típicos de trabajo

| Parámetro | Valor típico |
|---|---|
| Trama / salto | 20–30 ms / 10 ms |
| Ventana | Hamming o Hann |
| Pre-énfasis | 0,95–0,97 |
| Orden LPC | 10–12 (fs = 8 kHz), 16–18 (fs = 16 kHz) |
| Rango de búsqueda de F0 (adultos) | 60–400 Hz |
| Pasa-altos de preprocesamiento | 60–80 Hz |

## 5. Software y librerías

| Herramienta | Uso |
|---|---|
| **NumPy / SciPy** (`signal.butter`, `filtfilt`, `lfilter`, `stft`, `freqz`, `tf2zpk`, `get_window`; `linalg.solve_toeplitz`) | Núcleo: lo que la rúbrica pide implementar y defender |
| **Matplotlib** | Señal, espectros, espectrogramas, plano Z |
| **librosa** | Carga de audio, `pyin` (F0), `lpc`, `mfcc`, `rms`, `zero_crossing_rate`, espectrogramas |
| **soundfile** | Leer/escribir `.wav` |
| **praat-parselmouth** | Praat desde Python: jitter, shimmer, HNR, formantes "de referencia" para **validar** lo nuestro |
| **Praat** (programa) | Estándar en clínica e investigación de voz; útil para mirar y etiquetar |
| **scikit-learn / pandas** | Clasificación, métricas, tablas de resultados |
| `wfdb` | Leer bases de PhysioNet en formato WFDB |

Estrategia recomendada: **implementar a mano** lo central (autocorrelación, LPC con Levinson, STFT, filtros) y usar librosa /
parselmouth solo para **comparar y validar**. La rúbrica evalúa la "correcta implementación de algoritmos vistos en la
materia" y en la defensa preguntan por qué cada decisión.

## 6. Bibliografía de partida

- Rabiner, L. & Schafer, R. *Theory and Applications of Digital Speech Processing*. Pearson, 2011. (Referencia general: modelo
  fuente-filtro, LPC, cepstrum.)
- Farrús, M., Hernando, J. & Ejarque, P. "Jitter and shimmer measurements for speaker recognition". Interspeech 2007.
- Elisei, N. "Análisis acústico de la voz normal y patológica utilizando dos sistemas diferentes: ANAGRAF y PRAAT".
  *Interdisciplinaria* 29(2), 2012.
- Goddard et al. (2009). Visualización de voz normal y patológica con reducción dimensional. *Biomedical Signal Processing and
  Control*. (Caso 5 de la clase de aplicaciones.)
- Martínez et al. (2015). Denoising bioinspirado en dominio espectro-temporal. *Digital Signal Processing*. (Caso 7.)
- Sharma et al. (2017). EMD para análisis AM-FM de señales de habla. *Speech Communication*. (Caso 6.)
- **Paper base:** Dutoit, T., Moreau, N. & Kroon, P. (2009). "How is speech processed in a cell phone conversation?". En
  Dutoit, T. & Marqués, F. (eds.), *Applied Signal Processing: A MATLAB-Based Proof of Concept*, cap. 1. Springer.

## 7. Paper base: How is speech processed in a cell phone conversation?

Subtítulo del capítulo: *"todo celular resuelve 10 ecuaciones lineales con 10 incógnitas cada 20 ms"*.

**Idea central.** El celular no transmite muestras de voz: cada 20 ms calcula los parámetros del modelo fuente-filtro y manda
solo eso; el receptor resintetiza la voz. Conclusión del paper: *el celular transmite una "foto" del tracto vocal más que la
voz misma*.

**Modelo (sección 1.1).**
- S(z) = E(z) / A_p(z): la voz es una excitación e(n) filtrada por un **todo-polos 1/A(z)** (filtro de síntesis). A(z) es el
  **filtro inverso** (FIR).
- Excitación: tren de pulsos de período T0 (sonoro) o ruido blanco de varianza σ² (sordo) → decisión **V/UV**.
- s(n) = e(n) − Σ aᵢ s(n−i): cada muestra es combinación lineal de las p anteriores + excitación.
- Los aᵢ minimizan la energía del residuo → **ecuaciones de Yule-Walker** con la autocorrelación (matriz **Toeplitz**) →
  **Levinson-Durbin**. Ganancia σ² = Σ aᵢ φ(i).
- Como solo usa la autocorrelación, el modelo copia la **envolvente espectral**, no la forma de onda ni la fase (el oído es más
  sensible a la magnitud que a la fase).
- **El filtrado inverso decorrelaciona la voz**: el residuo tiene espectro casi plano pero conserva F0 y armónicos.
- Estimar T0 es difícil porque la voz nunca es periódica de verdad: **jitter** (período variable) y **shimmer** (amplitud
  variable), más ruido.

**En la práctica (1.1.3).** Tramas de 30 ms con salto de 10 ms y ventana Hamming; 240 muestras a 8 kHz. Orden p = 2·B + 2 (B =
ancho de banda en kHz; ~1 formante por kHz, 2 polos por formante, +2 para la forma del pulso glótico) → **p = 10 a 8 kHz**.

**Códecs (1.1.4).**

| Códec | Excitación | Tasa | Notas |
|---|---|---|---|
| LPC10 (NATO, hasta 1996) | Pulsos o ruido (binaria) | 2,4 kbps | Tramas de 22,5 ms, 54 bits: 7 pitch+V/UV, 5 ganancia, 42 coeficientes. Suena "zumbón"; sensible a errores de F0 y V/UV; en voces femeninas el 2º armónico cae sobre F1 y se confunde con F0 |
| MPE (Atal 1982) | Pocos pulsos con posición y amplitud libres | — | Primer **análisis por síntesis** (lazo cerrado); ya no necesita pitch ni V/UV |
| CELP (Schroeder y Atal 1985) | Vector elegido de un **libro de códigos** (cuantización vectorial del residuo) | — | + predictor de largo plazo (retardo ≈ período) + filtro perceptual A(z)/A(z/γ), γ ≈ 0,8 |
| GSM FR (1992–96) | RPE-LPC (pulsos equiespaciados) | 13 kbps | Tramas de 20 ms, 260 bits |
| GSM EFR / G.729 / AMR | ACELP (CELP algebraico) | 12,2 / 8 / 4,75–12,2 kbps | Los celulares actuales usan variantes de CELP (el capítulo dice 11,2 kbps para EFR; el estándar es 12,2) |

En vez de los aᵢ se cuantizan **coeficientes de reflexión** o *log area ratios*: cuantizar directamente los aᵢ puede dar un
filtro **inestable** (polos fuera del círculo unitario).

**Prueba de concepto (1.2), paso a paso.**
1. Espectrograma de banda ancha (ventana de 5 ms): pitch como líneas verticales, formantes como bandas horizontales.
2. Trama sonora de 30 ms: LPC orden 10 → `freqz` de A(z) y 1/A(z) → la envolvente 1/A(z) se apoya sobre el periodograma →
   `zplane`: los **polos quedan cerca del círculo unitario en los ángulos de los formantes**.
3. Residuo = filtrar la trama por A(z): espectro plano con los armónicos. Pasarlo por 1/A(z) devuelve la trama original.
4. Síntesis sonora con tren de pulsos (ganancia ajustada a la varianza del residuo) → misma envolvente, forma de onda distinta.
5. Trama sorda: síntesis con ruido blanco; PSD con `pwelch`.
6. Archivo completo con F0 fija: hay que **conservar el estado interno del filtro** entre tramas (si no, cada respuesta al
   impulso se corta) y manejar el desfase de pulsos entre tramas.
7. Todo sordo → suena a susurro. F0 original (estilo LPC10): T0 = máximo de la autocorrelación del residuo; V/UV = relación entre
   ese máximo y la varianza del residuo.
8. CELP: libro de códigos de 512 vectores aleatorios, tramas de síntesis de 5 ms, búsqueda voraz de las N mejores componentes
   (proyecciones), "ringing" del filtro, predicción de largo plazo y filtro perceptual. Con 2 componentes + LTP ≈ 70 bits cada
   5 ms = 14 kbps.

**Traducción MATLAB → Python** (para el Lead Coder):

| MATLAB (paper) | Python |
|---|---|
| `wavread`, `sound` | `soundfile.read`; `IPython.display.Audio` |
| `lpc(x, p)` | propia (autocorrelación + `scipy.linalg.solve_toeplitz`) y `librosa.lpc` para validar |
| `filter(b, a, x)` / `filter(b, a, x, z)` | `scipy.signal.lfilter(b, a, x)` / `lfilter(b, a, x, zi=z)` (devuelve también el estado final) |
| `freqz(b, a, 512)` | `scipy.signal.freqz(b, a, worN=512, fs=fs)` |
| `zplane(1, a)` | `scipy.signal.tf2zpk` + gráfico propio con el círculo unitario |
| `periodogram`, `pwelch` | `scipy.signal.periodogram`, `scipy.signal.welch` |
| `specgram(x, 512, fs, hamming(40))` | `scipy.signal.spectrogram` o `plt.specgram(x, NFFT=40, Fs=fs, window=np.hamming(40), pad_to=512)` |
| `hamming(N)`, `randn` | `np.hamming(N)` o `scipy.signal.get_window`; `np.random.randn` |

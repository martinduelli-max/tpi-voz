# Datos

Las señales **no se suben al repositorio** (pesan mucho y tienen licencia propia). Cada integrante las descarga en esta carpeta
siguiendo las instrucciones de abajo. Una vez elegida la base, documentar acá: fs, resolución, formato, cantidad de sujetos por
clase, tareas de habla, licencia y cómo citarla.

## Bases candidatas (voz)

| Base | Contenido | fs / formato | Acceso | Sirve para |
|---|---|---|---|---|
| [VOICED (PhysioNet)](https://physionet.org/content/voiced/1.0.0/) | 208 vocales /a/ sostenidas de 5 s: 150 patológicas, 58 sanas, con diagnóstico | 8 kHz, `.txt`/WFDB | Libre | Sana vs patológica |
| [Saarbrücken Voice Database (SVD)](https://stimmdb.coli.uni-saarland.de/) | > 2000 hablantes; vocales /a/ /i/ /u/ en tono normal, alto, bajo y glissando + frase; ~70 patologías; incluye EGG | 50 kHz | Libre. Web oficial permite bajar subconjuntos; espejo completo en [Zenodo](https://zenodo.org/records/16874898) (~38 GB) y paquete `sbvoicedb` en PyPI | Disfonías, nódulos, parálisis, comparación por patología |
| [Italian Parkinson's Voice and Speech](https://ieee-dataport.org/open-access/italian-parkinsons-voice-and-speech) | Parkinson vs sanos jóvenes y mayores; vocales, sílabas (pa/ta), palabras, lectura | `.wav` + `.xlsx` | Open access (CC-BY), requiere cuenta IEEE gratuita | Parkinson, prosodia, espacio vocálico |
| [MDVR-KCL (Zenodo)](https://zenodo.org/records/2867216) | 37 personas (16 Parkinson, 21 control), grabadas con celular: lectura + diálogo espontáneo | `.wav` | Libre | Parkinson en condiciones "de app" |
| [PTDB-TUG](https://www.spsc.tugraz.at/databases-and-tools/ptdb-tug-pitch-tracking-database-from-graz-university-of-technology.html) | 20 hablantes, 4720 grabaciones con micrófono **+ laringógrafo** y F0 de referencia | 48 kHz | Libre (ODbL) | Validar estimadores de F0 contra verdad de referencia |
| [NOIZEUS](https://ecs.utdallas.edu/loizou/speech/noizeus/) | 30 frases con 8 ruidos reales (bar, auto, calle, tren…) a 0/5/10/15 dB + versión limpia | 8 kHz | Libre | Realce de voz / robustez al ruido |
| [CMU ARCTIC](http://festvox.org/cmu_arctic/) | ~1150 frases por hablante, algunas con EGG | 16 kHz | Libre | Voz limpia de referencia, LPC/codificación |
| [TORGO](http://www.cs.toronto.edu/~complingweb/data/TORGO/torgo.html) | Habla disártrica (parálisis cerebral, ELA) + controles | 16 kHz | Libre | Disartria, inteligibilidad, formantes |

## Base elegida: PTDB-TUG

- **Página:** https://www.spsc.tugraz.at/databases-and-tools/ptdb-tug-pitch-tracking-database-from-graz-university-of-technology.html
- **Reporte técnico:** https://www2.spsc.tugraz.at/databases/PTDB-TUG/DOCUMENTATION/PTDB-TUG_REPORT.pdf
- **Cita:** G. Pirker, M. Wohlmayr, S. Petrik, F. Pernkopf, "A Pitch Tracking Corpus with Evaluation on Multipitch Tracking
  Scenario", Interspeech 2011, pp. 1509–1512.
- **Licencia:** Open Database License (ODbL 1.0) + Database Contents License.
- **Hablantes:** 20 nativos de inglés, 10 mujeres (F01–F10) y 10 hombres (M01–M10), 22 a 48 años.
- **Material:** 236 frases de TIMIT por hablante, 4720 en total. Las frases `sa1` y `sa2` las leen los 20 hablantes; el
  resto (`sx`, `si`) lo lee una mujer y un hombre.
- **Señales:** micrófono (`MIC`) y laringógrafo (`LAR`), WAV mono, 48 kHz, 16 bits.
- **Referencia de F0 (`REF`, archivos `.f0`):** RAPT sobre el laringógrafo filtrado pasa-altos (Kaiser, 25 Hz mujeres /
  15 Hz hombres), ventana de 32 ms y salto de 10 ms. Texto con 4 columnas por trama: **F0 [Hz]**, **probabilidad de
  sonoridad**, RMS local, pico de correlación normalizada. Las tramas sordas tienen F0 = 0.
- **Nombres de archivo:** `[mic|lar|ref]_[hablante]_[frase]`, por ejemplo `mic_F04_sa2.wav`, `ref_F04_sa2.f0`.

### Descarga

Completa: `SPEECH_DATA_ZIPPED.zip` (3,9 GB) en http://www2.spsc.tugraz.at/databases/PTDB-TUG/

Archivos sueltos (alcanza para las primeras pruebas), con la estructura
`SPEECH DATA/{FEMALE|MALE}/{MIC|LAR|REF}/{hablante}/`:

```
https://www2.spsc.tugraz.at/databases/PTDB-TUG/SPEECH%20DATA/FEMALE/MIC/F01/mic_F01_sa1.wav
https://www2.spsc.tugraz.at/databases/PTDB-TUG/SPEECH%20DATA/FEMALE/REF/F01/ref_F01_sa1.f0
```

Guardar en `data/ptdb-tug/` respetando la misma estructura.

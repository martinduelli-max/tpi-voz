# R03 — Devolución del tutor por correo (Hito 1)

- **Fecha:** 25/10/2026
- **Tutor:** Iván Peralta
- **Medio:** correo a María Victoria Viganoni

## Devolución

- Se avanza con la propuesta **"Estimación de pitch y decisión sonoro/sordo para LPC10, validada con laringógrafo"**.
  Motivos: ataca una debilidad que señala el propio capítulo (sensibilidad de LPC10 a errores de F0 y sonoro/sordo), usa datos
  reales con referencia objetiva, las métricas son claras y el laringógrafo aporta el componente biomédico.
- Sugerencias de organización:
  - Base: implementar el análisis-síntesis LPC10 del capítulo (sección 1.2.6).
  - Comparar el estimador del capítulo con cepstrum y YIN usando error grueso de pitch, error fino y error de decisión
    sonoro/sordo, separados por sexo.
  - Meter cada estimador en el decodificador y medir cuánto cambia la voz resintetizada. Agregar como caso de referencia la F0
    del laringógrafo (mejor calidad posible con pitch perfecto).
  - PTDB-TUG viene a 48 kHz: remuestrear a 8 kHz (factor 6) filtrando pasabajos antes de diezmar para evitar aliasing
    (`scipy.signal.resample_poly` o `decimate`).
- La propuesta de voz sonora a partir de susurro queda como posible extensión si sobra tiempo.

## Para la próxima reunión

- **Traer una primera prueba con algunas señales de la base.**

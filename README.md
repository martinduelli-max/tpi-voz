# TPI Señales y Sistemas 2026 — Procesamiento de voz

Trabajo Práctico Integrador (TPI) de Señales y Sistemas, Bioingeniería (FIUNER), 2026.
Tema general: **procesamiento digital de señales de voz** con datos reales.

Paper base: Dutoit, Moreau & Kroon (2009), "How is speech processed in a cell phone conversation?", cap. 1 de
*Applied Signal Processing: A MATLAB-Based Proof of Concept* (Springer). El PDF no se sube al repo (derechos de autor).

> Estado: **Hito 2 — Propuesta de implementación** (semana 12).
> Propuesta elegida (Hito 1, [entrega](docs/hitos/Hito1_Grupo3_CodificacionDeLaVoz.pdf)): **Evaluación de métodos de
> estimación de pitch y clasificación sonoro/sordo en la codificación de voz LPC10** (base PTDB-TUG).

Grupo 3 — Tutor: Iván Peralta.

## Equipo y roles

| Integrante | Rol principal | Rol secundario |
|---|---|---|
| Martín Duelli | Responsable Técnico (Lead Coder) | |
| María Victoria Viganoni | Moderador/Coordinador (project manager) | |
| Paula Lonardi | Analista/Documentador (Scribe) | |
| Zoe Barrionuevo | | |
| Narella Corona | | |
| Chiara Kremer | | |

Roles definidos según la Guía Metodológica del TPI. Ver detalle y roles sugeridos extra en
[docs/reuniones/2026-10-18_R01_brainstorm.md](docs/reuniones/2026-10-18_R01_brainstorm.md).

## Hitos

| Hito | Semana | Entregable | Estado |
|---|---|---|---|
| H1 — Definición del problema | 11 | 1 carilla con 3 ideas → 1 elegida | en curso |
| H2 — Propuesta de implementación | 12 | 2 carillas: diagrama de bloques, justificación teórica, link a la base | — |
| H3 — Prototipo funcional (MVP) | 14 | Notebook / script Python + gráficos preliminares | — |
| H4 — Defensa final | 15 | Presentación 15–20 min + defensa individual | — |

El incumplimiento de un hito impide avanzar al siguiente.

## Estructura del repositorio

```
tpi-voz/
├── README.md               ← este archivo
├── docs/
│   ├── reuniones/          ← minutas: AAAA-MM-DD_RNN_tema.md
│   ├── hitos/              ← entregables de cada hito
│   ├── teoria/             ← resúmenes teóricos del grupo
│   └── ia/                 ← registro OBLIGATORIO de uso de IA
├── data/                   ← señales (NO se suben al repo, ver data/README.md)
├── notebooks/              ← exploración y MVP
├── src/                    ← funciones reutilizables (filtros, LPC, F0, métricas…)
├── results/figures/        ← gráficos exportados para informe y defensa
├── requirements.txt
└── .gitignore
```

## Cómo correr

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt
jupyter lab
```

Las bases de datos se descargan aparte: instrucciones en [data/README.md](data/README.md).

## Notebooks

| Notebook | Qué hace |
|---|---|
| [01_primera_prueba_LPC10](notebooks/01_primera_prueba_LPC10.ipynb) | Análisis-síntesis LPC10 del capítulo sobre 8 señales de PTDB-TUG: remuestreo, envolvente, polos, residuo, pitch del capítulo y comparación con el laringógrafo |
| [02_comparacion_estimadores](notebooks/02_comparacion_estimadores.ipynb) | Compara el estimador del capítulo con cepstrum y YIN (VDE, GPE, FPE, mujeres vs hombres) sobre 200 grabaciones, y mete cada uno en el decodificador LPC10 |
| [03_evaluacion_base_completa](notebooks/03_evaluacion_base_completa.ipynb) | Describe la base PTDB-TUG y repite la comparación sobre toda la base (4678 grabaciones, sin las de calibración): métricas por sexo y por hablante, test de Wilcoxon, error vs F0 del hablante, errores de octava y conclusiones. Genera una biblioteca de 400 audios para comparar métodos (`results/audio/03_base_completa/`, con `indice.xlsx`). Guarda caché en `results/cache/` |

Se corren desde la carpeta `notebooks/` (las rutas son relativas). En Colab, cambiar `DATA_DIR` en la primera celda; si
faltan archivos, se descargan sueltos del servidor de PTDB-TUG.

## Convenciones

- Minutas de cada reunión en `docs/reuniones/` (las decisiones de WhatsApp se pasan a la minuta).
- Commits chicos y descriptivos, en castellano: `feat: filtro pasa-altos de preprocesamiento`.
- Todo uso de IA se registra en [docs/ia/registro-uso-ia.md](docs/ia/registro-uso-ia.md)
  (la cátedra lo exige en el informe final; mal uso = desaprobación automática).

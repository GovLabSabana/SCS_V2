
Contiene:
- ID del proyecto (ej. `1-1`)
- Nombre exacto del proyecto
- Objetivo
- Requerimientos

Regla:
- El **primer número del ID define el área temática**.

---

### 4.2 SisPT por municipio (entrada)

Carpeta:

SisPT/
├── 25486.xlsx
├── 25815.xlsx
└── ...


Cada archivo corresponde a **un municipio** y debe incluir la hoja:

Columnas críticas:
- `Código de indicador de producto (MGA)`
- `Personalización de Indicador de Producto`
- `Código DANE`
- `Entidad Territorial`

🚫 No usar códigos IP, sectores ni programas como proxy.  
✅ Usar **exclusivamente** códigos de producto MGA.

---

## 5. Proceso de Matching (Pipeline Lógico)

El sistema utiliza un proceso híbrido de dos fases para garantizar precisión y escalabilidad:

### Fase 1: Scoring Técnico (Filtrado)
Para cada municipio y cada proyecto estratégico, el sistema selecciona los **100 productos más prometedores** del SisPT:
1. **Fuzzy Matching**: Se evalúa la similitud textual ignorando el orden de las palabras (`token_set_ratio`).
2. **Coincidencia de Tokens**: Se cuentan palabras clave compartidas de más de 3 letras.
3. **Selección**: Solo los 100 con mayor puntaje pasan a la siguiente fase.

### Fase 2: Análisis Semántico por IA (LLM)
La IA recibe el proyecto y los 100 candidatos. Realiza los siguientes pasos:
1. **Pensamiento Interno**: Un análisis profundo donde contrasta el objetivo del proyecto contra el producto y evalúa si los requerimientos técnicos se ven reflejados. Clasifica la relación como directa, funcional o nula.
2. **Selección Final**: Elige un máximo de **5 productos** por proyecto.
3. **Calificación**: Asigna un valor de **0 a 3** basado estrictamente en la escala definida abajo.
4. **Justificación**: Redacta una justificación técnica de 1-2 frases.

---

## 6. Escala de calificación multicriterio (OBLIGATORIA)

Cada proyecto es evaluado en **tres criterios independientes**, cada uno en escala de **0 a 5**.
La calificación final del proyecto es el **promedio** de los tres.

### Criterio 1: Especificidad
Qué tan directamente el programa del plan de desarrollo apunta a lo que busca el proyecto.

| Valor | Significado |
|------|--------------|
| 0 | Sin relación alguna con el objetivo o requerimientos del proyecto |
| 1 | Relación muy tangencial; el producto solo crea condiciones generales muy lejanas |
| 2 | El producto es genérico pero guarda cierta relación con el sector del proyecto |
| 3 | El producto cumple parcialmente el objetivo O algunos requerimientos del proyecto |
| 4 | El producto cumple casi totalmente el objetivo y la mayoría de requerimientos |
| 5 | El producto es prácticamente idéntico en objetivo y cumple todos los requerimientos del proyecto |

### Criterio 2: Visión regional
Qué tanto el programa del plan de desarrollo contempla cooperación o impacto en otros municipios de Sabana Centro.

| Valor | Significado |
|------|--------------|
| 0 | El producto es estrictamente local, sin mención de articulación regional |
| 1 | El producto es local pero podría generar externalidades menores en otros municipios |
| 2 | El producto menciona o implica coordinación con algún actor regional (gobernación, etc.) |
| 3 | El producto involucra activamente a otros municipios o a la región Sabana Centro |
| 4 | El producto está diseñado para tener alcance regional claro y explícito |
| 5 | El producto es un mecanismo de gobernanza o acción colectiva intermunicipal por diseño |

### Criterio 3: Impacto
Cuánto puede contribuir el producto a la sostenibilidad ambiental, social o económica de Sabana Centro.

| Valor | Significado |
|------|--------------|
| 0 | El producto no tiene efecto relevante en la sostenibilidad de la región |
| 1 | El producto tiene un impacto muy marginal o solo simbólico |
| 2 | El producto contribuye de forma modesta a una dimensión de la sostenibilidad |
| 3 | El producto tiene un impacto moderado y medible en la sostenibilidad regional |
| 4 | El producto tiene un impacto significativo y aborda varias dimensiones de la sostenibilidad |
| 5 | El producto es transformador para la sostenibilidad regional a largo plazo |

⚠️ Regla dura:  
Sin producto MGA → **los tres criterios deben ser 0, y el promedio resultante será 0**.

---

## 7. Estructura del dataset de salida

El sistema genera un archivo Excel en `salidas/resultados_matching.xlsx` con las siguientes columnas:

| Columna | Descripción |
|-------|------------|
| Municipio | Nombre del municipio (extraído de columna "Entidad territorial") |
| Codigo_DANE | Código DANE (extraído de columna "Código DANE") |
| Documento | Siempre: `SisPT – Plan indicativo - Productos` |
| ID_Proyecto | ID del proyecto estratégico (ej. 1-1, 3-3) |
| Nombre_Proyecto | Nombre exacto del proyecto |
| Codigos_MGA | Lista de códigos MGA seleccionados |
| Indicador de Producto(MGA) | Texto literal del indicador asociado al código MGA |
| Productos | Texto literal de “Personalización de Indicador de Producto” |
| Especificidad | Puntaje 0-5: qué tan directa es la relación con el proyecto |
| Vision_Regional | Puntaje 0-5: alcance intermunicipal o regional del programa |
| Impacto | Puntaje 0-5: contribución a la sostenibilidad de la región |
| Calificacion_Promedio | Promedio decimal de los tres criterios anteriores |
| Justificacion | 1–2 frases, técnicas y concisas, explicando los tres puntajes |

🚫 No resumir ni reinterpretar nombres de productos.  
✅ Copiar literal desde SisPT.

---

## 8. Qué NO debe hacer ningún agente

- ❌ Inventar productos, códigos o nombres.
- ❌ Usar PDFs del Plan de Desarrollo.
- ❌ Usar sectores, programas o códigos IP como sustituto.
- ❌ Inflar calificaciones por lenguaje genérico.
- ❌ Escribir justificaciones largas o narrativas.

## 9. Estructura del proyecto

SCS_V2/
├── main.py
├── Proyectos.xlsx
├── SisPT/
│   ├── 25486.xlsx
│   ├── 25815.xlsx
│   └── ...
├── .env
└── salidas/
    └── resultados_matching.xlsx
└── README.md


---

## 10. Estado actual del proyecto

- Metodología validada manualmente (Áreas 1 a 6).
- Piloto exitoso con municipio Nemocón.
- Pipeline diseñado para escalar a todos los municipios.
- Dataset final reproducible y comparable.

---

## 11. Próximos pasos esperados

- Automatización completa municipio × 46 proyectos.
- Caché de resultados del modelo.
- Análisis comparativo intermunicipal.
- Visualización y reporting.

---

**Este README funciona como contrato metodológico.  
Cualquier agente que trabaje en este proyecto debe seguirlo estrictamente.**

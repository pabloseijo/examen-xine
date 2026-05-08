---
title: BIM y BM25
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# BIM y BM25

## BIM — Binary Independence Model

Modelo probabilístico de RI. Los documentos se representan como vectores binarios (1 si el término está presente, 0 si no). Asume que los términos son **independientes** entre sí.

**Objetivo**: estimar la probabilidad de que un documento sea relevante dado que contiene ciertos términos.

### Variables clave

- **pᵢ**: probabilidad de que el término i esté en un documento **relevante**
- **rᵢ**: probabilidad de que el término i esté en un documento **irrelevante**
- **S**: número total de documentos relevantes conocidos
- **s**: documentos relevantes que contienen el término t
- **N**: total de documentos en la colección
- **dfₜ**: documentos que contienen el término t

### Tabla de contingencia

|  | Relevante (R=1) | No relevante (R=0) | Total |
|--|-----------------|-------------------|-------|
| término presente (xᵢ=1) | s | dfₜ - s | dfₜ |
| término ausente (xᵢ=0) | S - s | (N - dfₜ) - (S - s) | N - dfₜ |
| Total | S | N - S | N |

### Estimación de pᵢ y rᵢ

```
pᵢ ≈ s / S
rᵢ ≈ (dfₜ - s) / (N - S)
```

**Ejemplo** (del examen):

| Término | n_Rᵢ (docs relevantes con término) | R (total relevantes) | n_Iᵢ (docs irrelevantes con término) | I (total irrelevantes) |
|---------|-------------------------------------|----------------------|---------------------------------------|------------------------|
| Term1 | 10 | 20 | 5 | 80 |
| Term2 | 15 | 20 | 10 | 80 |
| Term3 | 5 | 20 | 40 | 80 |

- p₁ = 10/20 = 0.5, r₁ = 5/80 = 0.0625
- p₂ = 15/20 = 0.75, r₂ = 10/80 = 0.125
- p₃ = 5/20 = 0.25, r₃ = 40/80 = 0.5

### Limitaciones de BIM

- No considera cuántas veces aparece el término (solo presencia/ausencia)
- No considera la longitud del documento
- La independencia entre términos es una asunción incorrecta pero funciona bien en práctica

---

## BM25 — Best Matching 25

Extensión probabilística de BIM que corrige sus limitaciones. Es el modelo de referencia en recuperación de información moderna.

### Mejoras sobre BIM

| Limitación BIM | Solución BM25 |
|---------------|--------------|
| Ignora frecuencia del término | TF normalizada (con saturación) |
| Ignora longitud del documento | Normalización por longitud con parámetro b |
| IDF básico | IDF mejorado |

### Parámetros

- **k1**: controla la saturación de la frecuencia de términos (típicamente 1.2–2.0)
- **b**: controla la normalización por longitud (entre 0 y 1, típicamente 0.75)

### Conceptualmente

BM25 asigna una puntuación a cada documento para una query considerando:
1. **TF normalizada**: cuántas veces aparece el término, pero con saturación (más ocurrencias ayudan, pero con rendimientos decrecientes)
2. **IDF**: qué tan raro es el término en la colección
3. **Longitud del documento**: documentos más largos se penalizan

**Basada en distribución de Poisson**: distingue entre "términos élite" (directamente relacionados con el tema del documento) y el resto.

## En el examen

**BIM** — lo que piden:
- Dado una tabla de ocurrencias de términos en docs relevantes e irrelevantes: calcular pᵢ y rᵢ para cada término
- NO hace falta memorizar fórmulas de scoring completo

**BM25** — lo que piden:
- Entender qué mejoras supone respecto a BIM (descripción conceptual)
- NO hace falta calcular con la fórmula completa

## Relación con otros conceptos

- Alternativa probabilística al [[tfidf]] (vectorial)
- [[metricas-ri]] evalúan rankings de BM25
- Responde a las mismas queries que [[indice-invertido]] pero con mejor ranking

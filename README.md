# Predicción de mora en tarjetas de crédito

**Proyecto Final Integrador · Unidad N°9**
Diplomado en Ciencia de Datos y Análisis Avanzado, Centro de e-Learning UTN BA · Grupo 19

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Frojas0/UTN-proyecto-final-integrador/blob/main/baseline_v9.ipynb)

## De qué se trata

Un emisor de tarjetas puede contactar cada mes sólo al 20% de su cartera para hacer cobranza preventiva. Este proyecto entrena un modelo que estima la probabilidad de que cada cliente deje de pagar el mes siguiente y ordena a quién llamar primero según la **pérdida esperada** (probabilidad × saldo). Después lo compara con las reglas que se usan hoy, con la misma cantidad de llamadas.

**Resultado principal** (6.002 clientes que el modelo no vio al entrenar): con 1.200 gestiones, el modelo alcanza NT$ 51,5 millones de deuda de clientes que efectivamente dejan de pagar. Es un 17,7% más que la mejor regla actual (con un margen de error que va de 12,4% a 23,6%). El detalle completo está en el [informe final](informe/Informe_Final.pdf).

## Qué hay en el repositorio

```
.
├── README.md
├── requirements.txt           # librerías y versiones exactas
├── baseline_v9.ipynb          # notebook completo, ya ejecutado
├── data/
│   └── default of credit card clients.xls   # datos originales de UCI
├── informe/
│   └── Informe_Final.pdf      # informe final del proyecto
├── figuras/                   # gráficos del informe (los genera el notebook)
└── resultados.json            # todos los resultados del modelo (lo genera el notebook)
```

## Datos

- **Fuente:** [Default of Credit Card Clients](https://archive.ics.uci.edu/dataset/350/default+of+credit+card+clients), UCI Machine Learning Repository, dataset 350.
- **Licencia:** CC BY 4.0. Permite compartir los datos citando la fuente, por eso se incluyen en `data/`.
- **Tamaño:** 30.000 clientes y 25 columnas, de abril a septiembre de 2005. No tiene valores faltantes.
- **Cita:** Yeh, I-C. y Lien, C-H. (2009). The comparisons of data mining techniques for the predictive accuracy of probability of default of credit card clients. *Expert Systems with Applications*, 36(2), 2473-2480.

## Cómo ejecutarlo

### En Google Colab

1. Abrir el notebook con el botón **Abrir en Colab** de arriba.
2. En la primera celda del Bloque 1 hay una línea comentada que empieza con `# %pip install`. Sacarle el `#` del comienzo, ejecutar esa celda y después ir a **Entorno de ejecución → Reiniciar sesión**.
3. Volver a comentar esa línea (o dejarla, no molesta) y elegir **Entorno de ejecución → Ejecutar todas**.

No hace falta subir los datos: si no los encuentra, el notebook los descarga de UCI.

### En una computadora local

```bash
git clone https://github.com/Frojas0/UTN-proyecto-final-integrador.git
cd UTN-proyecto-final-integrador
python -m venv .venv
source .venv/bin/activate          # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook baseline_v9.ipynb
```

Después, **Kernel → Restart & Run All**. También funciona en VS Code con la extensión de Jupyter.

**Tiempo:** unos minutos en una computadora común. La parte más lenta es la comparación con Random Forest (sección 6.3), que tarda menos de un minuto.

## A tener en cuenta

- **Usar las versiones de `requirements.txt`.** Con otra versión de scikit-learn el modelo da números un poco distintos. Por ejemplo, con scikit-learn 1.9.1 la PR-AUC del conjunto de prueba da 0,550 en lugar de 0,547, y la ventaja sobre la mejor regla va de 13,5% a 24,5% en lugar de 12,4% a 23,6%. Las conclusiones no cambian, pero las cifras dejan de coincidir con el informe. Colab trae versiones más nuevas, por eso hay que instalar las de este archivo antes de ejecutar.
- **SHAP es opcional.** La sección 7.6bis usa la librería `shap`. Si no está instalada, el notebook intenta instalarla; si no puede, saltea esa sección y sus dos figuras, y el resto funciona igual.
- **No se toca el conjunto de prueba para decidir.** El notebook está ordenado para que todas las decisiones (limpieza, algoritmo, forma de ordenar a los clientes) se tomen sólo con los datos de entrenamiento. El conjunto de prueba se usa una sola vez, al final, para medir.
- **Los supuestos económicos son supuestos.** La severidad de la pérdida (60%), la efectividad de la gestión (20%) y el costo por contacto (NT$ 200) están definidos al comienzo del notebook (`CONFIG`) y hay que confirmarlos con el negocio. Cambian el tamaño del beneficio, pero no cuál estrategia gana.
- **Los costos de puesta en marcha y mantenimiento no salen del notebook.** Los NT$ 500.000 de inversión inicial y los NT$ 50.000 por mes que menciona el informe son estimaciones del equipo. Todos los demás números del informe salen de `resultados.json`.
- **Los datos son de Taiwán, de 2005, y cubren sólo seis meses.** Por eso no se pudo probar el modelo con meses posteriores. Antes de usarlo en un negocio real hay que reentrenarlo con datos propios y hacer una prueba piloto con grupo de control.

## Qué genera

- **`resultados.json`:** todos los resultados del modelo que cita el informe.
- **`figuras/`:** los gráficos del informe en PNG (nueve, o siete si `shap` no está instalado).

Si se ejecuta el notebook de nuevo, estos archivos se reemplazan. Con las versiones de `requirements.txt` quedan iguales (sólo cambian los tiempos de ejecución que se registran).

## Cómo está ordenado el notebook

| Bloque | Qué hace | Etapa de CRISP-DM |
|:---:|---|---|
| 0–2 | Revisa las librerías, carga los datos, verifica la fuente y controla la calidad | Comprensión de los datos |
| 3 | Fija las métricas, la forma de validar, los objetivos de éxito y las reglas de comparación, antes de modelar | Comprensión del negocio |
| 4–5 | Separa entrenamiento y prueba (sin repartir duplicados) y toma las decisiones de limpieza | Preparación de los datos |
| 6 | Modelo base (regresión logística), hipótesis y comparación de tres algoritmos | Modelado |
| 7 | Evaluación final: KPI de negocio, estabilidad, sensibilidad, reproducibilidad, equidad e interpretación (incluye SHAP) | Evaluación |
| 8–9 | Figuras y `resultados.json` | — |

Al final del notebook hay un anexo con los cambios entre versiones.

## Cómo comprobar que la ejecución salió bien

- **Semilla fija:** `SEED = 42` en todo el notebook.
- **Huella digital de los datos** (SHA-256 del `.xls`):
  `30c6be3abd8dcfd3e6096c828bad8c2f011238620f5369220bd60cfc82700933`
  El notebook la calcula y la guarda en `resultados.json` → `fuente.sha256_full`. Si no coincide, el archivo de datos es otro.
- **Valores esperados en `resultados.json`:**

| Clave | Qué es | Valor esperado |
|---|---|---|
| `cv.pr_auc_mean` | PR-AUC del modelo base (logística) | 0.503 |
| `cv_final.pr_auc_mean` | PR-AUC del modelo final en validación cruzada | 0.5579 |
| `holdout.pr_auc` | PR-AUC en el conjunto de prueba | 0.547 |
| `KPI.expo_M1` | Deuda alcanzada por el modelo (NT$) | 51471700.0 |
| `boot.ratio_ic` | Ventaja sobre la mejor regla, intervalo del 95% (%) | [12.42, 23.57] |

### Versiones con las que se generaron los resultados

| Paquete | Versión |
|---|---|
| Python | 3.12 o 3.13 |
| pandas | 3.0.2 |
| numpy | 2.4.4 |
| scikit-learn | 1.8.0 |
| matplotlib | 3.10.8 |

## Autores

Esteban German Rojas Serrano · Agustin Alejandro Martinez · Ana Paula Athor · Alexia Antonoglou · Facundo Martin Rojas

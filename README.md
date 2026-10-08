# INF-8239 · Demostración OCR

**Asignatura:** Ciencia de Datos II (INF-8239)  
**Unidad:** Procesamiento de Lenguaje Natural y Visión Computacional

Este repositorio contiene una demostración práctica de **Reconocimiento Óptico de Caracteres (OCR)** desarrollada en Google Colab.

La actividad tiene como propósito mostrar cómo una imagen que contiene texto puede transformarse en texto procesable mediante un flujo sencillo de adquisición, preprocesamiento, reconocimiento y validación.

La demostración no constituye un laboratorio independiente ni genera una puntuación adicional.

---

## Objetivo

Aplicar un flujo básico de OCR para:

- crear una imagen reproducible con texto;
- preprocesar la imagen;
- convertirla a escala de grises;
- aplicar umbralización;
- reconocer caracteres mediante Tesseract;
- inspeccionar la confianza de reconocimiento por palabra;
- discutir las limitaciones y riesgos de interpretación.

---

## ¿Qué es OCR?

OCR significa **Optical Character Recognition** o **Reconocimiento Óptico de Caracteres**.

Su función es transformar una imagen que contiene caracteres en texto procesable.

El proceso no se limita a “leer una foto”. Puede incluir varias etapas:

| Etapa | Acción | Riesgo |
|---|---|---|
| Adquisición | Capturar o cargar la imagen | Baja resolución, inclinación o iluminación |
| Preprocesamiento | Escala de grises, reducción de ruido y umbralización | Eliminar trazos útiles |
| Detección | Localizar regiones con texto | Omitir columnas o mezclar bloques |
| Reconocimiento | Convertir formas visuales en caracteres | Confundir símbolos semejantes |
| Posprocesamiento | Aplicar reglas lingüísticas o del dominio | Corregir indebidamente nombres o códigos |
| Validación | Revisar confianza y comparar con el documento | Aceptar automáticamente errores críticos |

---

## Entorno utilizado

La demostración fue ejecutada en **Google Colab**.

Se instalaron las siguientes herramientas:

```bash
!apt-get -qq update
!apt-get -qq install -y tesseract-ocr tesseract-ocr-spa
!pip -q install pytesseract opencv-python-headless pillow
```

También fue necesario instalar una fuente compatible con caracteres en español:

```bash
!apt-get -qq install -y fonts-dejavu-core
```

---

## Creación de una imagen reproducible

Se generó una imagen con fondo blanco y tres líneas de texto:

```text
Solicitud: INF-8239
Estado: Pendiente de revisión
Prioridad: Alta
```

La imagen fue creada con Pillow utilizando una fuente DejaVu Sans.

Código utilizado:

```python
from PIL import Image, ImageDraw, ImageFont

canvas = Image.new("RGB", (1200, 320), "white")
draw = ImageDraw.Draw(canvas)

font = ImageFont.truetype(
    "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf",
    42
)

draw.text((55, 70), "Solicitud: INF-8239", fill="black", font=font)
draw.text((55, 150), "Estado: Pendiente de revisión", fill="black", font=font)
draw.text((55, 230), "Prioridad: Alta", fill="black", font=font)

display(canvas)
canvas.save("documento_demo.png")
```

---

## Preprocesamiento de la imagen

La imagen fue cargada con OpenCV y transformada a escala de grises.

Posteriormente se aplicó umbralización automática mediante el método de Otsu.

```python
import cv2
import pytesseract
from PIL import Image

image = cv2.imread("documento_demo.png")

gray = cv2.cvtColor(
    image,
    cv2.COLOR_BGR2GRAY
)

_, binary = cv2.threshold(
    gray,
    0,
    255,
    cv2.THRESH_BINARY + cv2.THRESH_OTSU
)
```

El preprocesamiento permite simplificar la imagen y facilitar el reconocimiento de los caracteres.

---

## Reconocimiento de texto

El reconocimiento se realizó mediante Tesseract con configuración para idioma español:

```python
text = pytesseract.image_to_string(
    binary,
    lang="spa"
)

print(text)
```

El OCR logró recuperar correctamente:

```text
Solicitud: INF-8239
Estado: Pendiente de revisión
Prioridad: Alta
```

---

## Confianza de reconocimiento

Además del texto extraído, se inspeccionó el nivel de confianza reportado por Tesseract para cada palabra.

Código utilizado:

```python
data = pytesseract.image_to_data(
    binary,
    lang="spa",
    output_type=pytesseract.Output.DATAFRAME
)

words = data.dropna(subset=["text"])
words = words[
    words["text"].str.strip().ne("")
]

words[["text", "conf"]].reset_index(drop=True)
```

Los resultados obtenidos fueron aproximadamente:

| Texto | Confianza |
|---|---:|
| Solicitud: | 93.24 |
| INF-8239 | 90.22 |
| Estado: | 96.50 |
| Pendiente | 96.74 |
| de | 96.91 |
| revisión | 96.79 |
| Prioridad: | 95.53 |
| Alta | 95.67 |

Los valores de confianza fueron altos para todas las palabras reconocidas.

---

## Interpretación responsable

El sistema OCR logró reconocer correctamente el contenido de la imagen de demostración y presentó niveles de confianza elevados.

Sin embargo, una confianza alta no garantiza que el contenido reconocido sea verdadero ni que el documento haya sido interpretado correctamente en su contexto.

OCR únicamente transforma información visual en texto procesable.

En documentos reales, especialmente cuando contienen:

- nombres;
- códigos;
- montos;
- fechas;
- información sensible;
- números de identificación;

la salida debe ser validada antes de utilizarse en procesos posteriores.

La calidad del reconocimiento puede verse afectada por factores como:

- resolución;
- iluminación;
- inclinación;
- ruido;
- tipografía;
- idioma;
- estructura del documento.

---

## Limitaciones

Esta demostración utiliza una imagen limpia, generada digitalmente y con buena resolución.

Por esta razón, no representa todas las dificultades presentes en documentos reales.

Entre las principales limitaciones se encuentran:

- no se evaluaron imágenes borrosas;
- no se utilizaron fotografías tomadas con cámara;
- no se evaluaron documentos manuscritos;
- no se probaron múltiples columnas;
- no se evaluaron tablas complejas;
- no se midió desempeño sobre diferentes tipos documentales;
- no se trabajó con información personal real.

---

## Uso responsable

OCR puede ser utilizado como etapa inicial en procesos de:

- digitalización;
- minería de texto;
- clasificación documental;
- búsqueda;
- extracción de información;
- tamizaje.

Sin embargo, el texto reconocido no debe aceptarse automáticamente como correcto en campos críticos.

Los documentos reales requieren autorización, protección de datos, reglas de retención y validación humana cuando el riesgo lo requiera.

---

## Conexión con la unidad

Este microejemplo muestra cómo una imagen puede convertirse en texto y posteriormente alimentar tareas de análisis de datos o procesamiento de lenguaje natural.

El flujo general puede resumirse como:

```text
Imagen
   ↓
Preprocesamiento
   ↓
OCR
   ↓
Texto
   ↓
Minería / clasificación / análisis
```

Cada etapa puede introducir errores, por lo que la calidad debe evaluarse y documentarse.

---

## Notebook

El notebook ejecutado contiene:

- instalación de Tesseract;
- instalación del idioma español;
- creación de una imagen reproducible;
- preprocesamiento con OpenCV;
- reconocimiento OCR;
- inspección de confianza;
- interpretación responsable.

Archivo principal:

```text
demostracion_ocr.ipynb
```

---

## Estado

- Instalación de OCR: completada.
- Imagen reproducible: creada.
- Preprocesamiento: completado.
- Reconocimiento: completado.
- Confianza por palabra: inspeccionada.
- Interpretación responsable: documentada.

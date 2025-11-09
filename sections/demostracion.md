## 1. Instalación y primeros pasos

Utilizamos el comando <a data-preview-video="assets/installation.mp4"><strong><code>pipx</code></strong></a> para instalar **word2speech** en un entorno aislado de la configuración global de python

Los comandos **models** y **cheat** nos permiten explorar las funcionalidades


## 2. Generación de Audio con Control Prosódico

El comando <a data-preview-video="assets/speak.mp4"><strong><code>speak</code></strong></a> permite no solo generar audio, sino también **controlar con precisión su prosodia**.

- Soporta varias opciones de configuración como **velocidad**, **voz**, **emoción**, etc.
- Podemos utiliza distintos modelos con la opción **--models**
- Se pueden definir puntos de entonación con la opción **--contour, -c** (sólo disponible para el modelo **speechgen**)
<p class="fragment" data-fragment-index="1" style="display:flex;justify-content:center;align-items: center;">
  <strong>Hipopótamo:</strong>&nbsp;&nbsp;<audio src="assets/hipopotamo.wav" controls></audio>
</p>


## 3. Configuración

La gestión de las claves API se puede hacer con el con el comando <a data-preview-video="assets/config.mp4"><strong><code>keys</code></strong></a>
Podemos crear directorios con opciones de configuración predefinidas
<p class="fragment" data-fragment-index="1" style="display:flex;justify-content:center;align-items: center;">
  <strong>tratamiento1/</strong>&nbsp;&nbsp;<audio src="assets/tratamiento1.wav" controls></audio>
</p>


## 4. Funcionalidad Específica de Intervención

- El comando **`prosody`** para un **ajuste fino de la prosodia**.
- El comando **`spell`** está diseñado específicamente para **ejercicios de concienciación silábica**.

### Ejemplo: Deletreo Silábico
```bash
# Deletrea "estornudo" como "es - tor - nu - do"
word2speech spell estornudo
[02:39:51]: Generando deletreo de sílabas: "estornudo"
[02:39:51]: Texto deletreado: es <break time="250ms"/> tor <break time="250ms"/> nu <break time="250ms"/> do
[02:39:52]: Audio deletreado generado "out_spell.wav" (coste: 0, saldo: 64137)
```

<audio src="assets/estornudo.wav" controls></audio>


## 5. Eficiencia con Manejo por Lotes

El comando <a data-preview-video="assets/batch.mp4"><strong><code>batch</code></strong></a> automatiza la generación de **grandes volúmenes de audio** a partir de un fichero de entrada en formato JSON.

Los terapeutas del proyecto Leeduca utilizan unos <a data-preview-link="assets/palabaras-tratamiento.png">ficheros</a> para los tratamientos de concienciación fonológica con palabras y pseudopalabras que se grababan una a una.

<p class="fragment" data-fragment-index="1">
Las claves son los <strong>&lt;directorios&gt;</strong> que va a crear y por cada clave una lista de listas. La lista externa contiene todos los ficheros que se van a generar y la interna contiene dos elementos [<strong>&lt;nombre del fichero&gt;</strong>, <strong>&lt;texto a generar&gt;</strong>].
</p>


### Rendimiento Comprobado
En nuestras pruebas, fue capaz de generar **765 audios** de palabras en solo **23 minutos**.

Esta funcionalidad es **crucial para escalar** la producción de material terapéutico.

Note: La herramienta combina facilidad de uso con potentes capacidades de procesamiento masivo.


## 6. Evaluación de audio

Con el comando <a data-preview-video="assets/analyze.mp4"><strong><code>analize</code></strong></a> obtenemos una evalución objetiva e instantánea de la calidad del audio.

|                  |                    |
| ---------------- | ------------------ |
| ./speechgen.wav: | 3.8849918842315674 |
| ./mms.wav:       | 3.733323574066162  |
| ./parler.wav:    | 3.375762701034546  |
<!-- .element: class="fragment" data-fragment-index="1" -->

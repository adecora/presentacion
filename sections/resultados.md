## Comparativa de Modelos TTS

La evaluación se centró en la **calidad perceptual** de los modelos TTS implementados.

Utilizando el índice MOS a través del comando **`analyze`**.

Para el análisis se ha preparado un entorno de pruebas mediante la descarga de [30 audios](https://github.com/adecora/proyecto-dislexia/tree/main/referencia) de [Tatoeba](https://tatoeba.org/es/sentences/search?from=spa&has_audio=yes&list=&native=yes&original=&orphans=no&query=&sort=relevance&sort_reverse=&tags=&to=&trans_filter=exclude&trans_has_audio=&trans_link=&trans_native=&trans_orphan=&trans_to=&trans_unapproved=&trans_user=&unapproved=no&user=&word_count_max=&word_count_min=3)

<p class="fragment" data-fragment-index="1">
Recurso abierto y de libre acceso, facilita la reproducibilidad de los experimentos y la comparación objetiva entre diferentes
configuraciones y sistemas de síntesis. Además, ofrece audios de referencia locutados con alta calidad y una amplia diversidad lingüística, aspectos clave en la
evaluación de modelos TTS.
</p>


El fichero <a data-preview-link="assets/data-validate.json"><strong><code>data-validate.json</code></strong></a> contiene la información necesaria para genrar los audios, el fichero <a href="https://github.com/adecora/proyecto-dislexia/blob/main/bin/generate-test-data.sh"><strong><code>bin/generate-test-data.sh</strong></code></a> automatiza la generación de los audios para el entorno de pruebas.


### Escuchas iniciales

<ul>
  <li style="margin-top:15px;display:flex;align-items: center;">
    <strong>Referencia:</strong>&nbsp;&nbsp;<audio src="assets/test_referencia.wav" controls></audio>
  </li>
  <li style="margin-top:10px;display:flex;align-items: center;">
    <strong>MMS:</strong>&nbsp;&nbsp;<audio src="assets/test_mms.wav" controls></audio>&nbsp;&nbsp;<p class="fragment" data-fragment-index="3">🥉​</p>
  </li>
  <li style="margin-top:10px;display:flex;align-items: center;">
    <strong>Parler:</strong>&nbsp;&nbsp;<audio src="assets/test_parler.wav" controls></audio>&nbsp;&nbsp;<p class="fragment" data-fragment-index="2">🥈​</p>
  </li>
  <li style="margin-top:10px;display:flex;align-items: center;">
    <strong>Speechgen:</strong>&nbsp;&nbsp;<audio src="assets/test_speechgen.wav" controls></audio>&nbsp;&nbsp;<p class="fragment" data-fragment-index="1">🥇​</p>
  </li>
</ul>


#### **word2speech:** --speed 1, --pitch 0 y --emotion neutral

Parámetros por defecto


#### **word2speech:** --speed 1, --pitch 0 y --emotion neutral

<div class="img-grid">
  <!-- imagen 1 -->
  <input type="checkbox" id="img1" class="img-zoom-toggle">
  <label for="img1" class="img-zoom-label">
    <img src="img/boxplot0.png" alt="Boxplot">
  </label>

  <!-- imagen 2 -->
  <input type="checkbox" id="img2" class="img-zoom-toggle">
  <label for="img2" class="img-zoom-label">
    <img src="img/correlacion0.png" alt="Correlación">
  </label>
</div>


#### **word2speech:** --speed 0.9 y --pitch -2

Una locución ligeramente más lenta y un tono más grave


#### **word2speech:** --speed 0.9 y --pitch -2

<div class="img-grid">
  <!-- imagen 1 -->
  <input type="checkbox" id="img3" class="img-zoom-toggle">
  <label for="img3" class="img-zoom-label">
    <img src="img/boxplot1.png" alt="Boxplot">
  </label>

  <!-- imagen 2 -->
  <input type="checkbox" id="img4" class="img-zoom-toggle">
  <label for="img4" class="img-zoom-label">
    <img src="img/correlacion1.png" alt="Correlación">
  </label>
</div>


#### **word2speech:** --speed 0.8, --emotion calm y --pitch -2

Un prosodia más calmada y relajada


#### **word2speech:** --speed 0.8, --emotion calm y --pitch -2

<div class="img-grid">
  <!-- imagen 1 -->
  <input type="checkbox" id="img5" class="img-zoom-toggle">
  <label for="img5" class="img-zoom-label">
    <img src="img/boxplot2.png" alt="Boxplot">
  </label>

  <!-- imagen 2 -->
  <input type="checkbox" id="img6" class="img-zoom-toggle">
  <label for="img6" class="img-zoom-label">
    <img src="img/correlacion2.png" alt="Correlación">
  </label>
</div>


### **Resultados MOS por Modelo**

| Modelo         | Media | Mediana | Mínimo | Máximo | Desviación |
| -------------- | :---: | :-----: | :----: | :----: | :--------: |
| **Referencia** | 3.301 |  3.290  |  2.803 |  3.783 |    0.226   |
| **Speechgen**  | 3.662 |  3.691  |  3.050 |  3.997 |    0.233   |
| **Parler**     | 3.235 |  3.225  |  2.643 |  3.832 |    0.262   |
| **MMS**        | 3.567 |  3.577  |  3.003 |  4.021 |    0.269   |


### **Análisis de Rendimiento**

**Speechegen 🥇**
<!-- .element: class="fragment" data-fragment-index="1" -->
**MMS 🥈**​
<!-- .element: class="fragment" data-fragment-index="2" -->
**Referencia 🥉**​
<!-- .element: class="fragment" data-fragment-index="3" -->
**Parler**
<!-- .element: class="fragment" data-fragment-index="4" -->
La tecnología de **Síntesis de Voz** (Text-to-Speech) ha evolucionado drásticamente en los últimos años.

Fundamentada en el gran avance de la **inteligencia artificial** en los últimos años.


## Paradigma en Cascada
**Ejemplo: Tacotron 2**

Fue uno de los primeros enfoques neuronales robustos. **Separa el proceso** en dos etapas principales:

- **Modelo acústico**: convierte texto en una representación intermedia (espectrograma de Mel)
- **Vocoder**: sintetiza la forma de onda final a partir de esa representación


<div class="split-slide">
  <img src="img/Tacotron2.png" alt="Arquitectura Tacotron 2">

  <ol class="text-col">
    <li><b>Texto de entrada:</b> El modelo recibe el texto (por ejemplo, una oración).</li>
    <li><b>Codificador de texto:</b> Convierte cada carácter o fonema en una representación numérica (vector), que captura información lingüística como pronunciación o entonación.</li>
    <li><b>Atención:</b> Actúa como un “mapa” que decide qué parte del texto corresponde al sonido que se está generando en ese momento.</li>
    <li><b>Decodificador:</b> Genera fragmentos del sonido representados como espectrogramas mel. El decodificador genera el frame a frame. En cada paso, la salida anterior se usa como entrada para el siguiente paso</li>
    <li><b>Post-net:</b> Mejora la calidad del espectrograma corrigiendo detalles.</li>
    <li><b>Vocoder:</b> Transforma el espectrograma en audio real (onda sonora).</li>
  </ol>
</div>

Note: Paso 1: Entrada y Codificación del Texto
El proceso comienza con la entrada de una secuencia de caracteres (el texto que se desea sintetizar).
1. El Encoder: El texto es procesado por un codificador (encoder).
    ◦ Este componente convierte los caracteres de entrada en representaciones ocultas.
    ◦ Utiliza una combinación de embeddings, capas convolucionales (CNNs) y una red Bidirectional Long Short-Term Memory (BiLSTN). La BiLSTN procesa la secuencia de caracteres en dos direcciones para capturar el contexto completo.
Paso 2: Conversión a Espectrograma de Mel (Modelo Acústico)
Las representaciones ocultas generadas por el encoder alimentan el decoder autorregresivo, que tiene la tarea de predecir la estructura temporal del audio.
1. El Decoder Autorregresivo con Atención: Este componente utiliza un mecanismo de atención para mapear las representaciones del texto a las características acústicas.
    ◦ Produce el espectrograma de Mel (Mel spectrogram) de forma autorregresiva (generando el audio frame a frame).
2. El PostNet: La predicción inicial del decoder es refinada por el bloque PostNet.
    ◦ PostNet es una red convolucional que actúa aprendiendo un valor residual que se aplica como corrección sobre la predicción del decoder.
3. Finalización de Secuencia: Un clasificador de stop token determina cuándo debe finalizar la secuencia, indicando el final del enunciado sintetizado.
Al finalizar este paso, se obtiene una representación espectral del audio (el espectrograma de Mel), que es una representación de frecuencias más cercana a la percepción humana.
Paso 3: Síntesis de la Forma de Onda (Vocoder Neuronal)
El espectrograma de Mel generado en el Paso 2 es la entrada para el vocoder.
1. El Vocoder Neuronal: Este componente es responsable de la tarea inversa al procesamiento inicial: toma el espectrograma de Mel (la representación de frecuencia-tiempo) y lo convierte directamente en la forma de onda de audio en el dominio del tiempo.
2. Generación de Audio Final: Aunque inicialmente se usaron modelos como WaveNet (que lograban alta calidad pero eran costosos), los sistemas modernos prefieren vocoders eficientes en tiempo real como HiFi-GAN, los cuales generan la forma de onda final con baja latencia y alta fidelidad.
--------------------------------------------------------------------------------
Nota sobre el Entrenamiento (Proceso Desacoplado)
Es importante notar que el entrenamiento de Tacotron 2 se realiza en un proceso desacoplado de dos etapas:
1. Entrenamiento del Modelo Acústico: Primero se entrena el modelo seq2seq (Paso 2). Para esto, se utilizan pares de (texto, audio). El audio se convierte a su espectrograma de Mel real, y el modelo aprende a predecirlo a partir del texto de entrada, minimizando el error de reconstrucción.
2. Entrenamiento del Vocoder: En la segunda etapa, el vocoder (Paso 3) se entrena de forma independiente para aprender a generar la forma de onda de audio a partir de los espectrogramas de Mel. Se usan los espectrogramas reales (no los predichos) para asegurar la máxima calidad


## Modelos No Autorregresivos
<a data-preview-link="img/FastSpeech2.png"><strong>Ejemplo: FastSpeech 2</strong></a>

Para superar la **lentitud** de los modelos en cascada, surgieron arquitecturas no autorregresivas.

- Generan **todos los frames** del espectrograma en **paralelo**
- **Inferencia más rápida** y robusta
- Evitan errores acumulativos como **repeticiones u omisiones**


<div class="split-slide">
  <img src="img/FastSpeech2.png" alt="Arquitectura Tacotron 2">

  <ol class="text-col">
    <li><b>Texto de entrada:</b> Se ingresa el texto que queremos convertir en voz (por ejemplo: “Hola, ¿cómo estás?”).</li>
    <li><b>Codificador Transformer:</b> Convierte el texto (en fonemas o caracteres) en representaciones numéricas que contienen la información lingüística y prosódica (cómo debería sonar cada parte).</li>
    <li><b>Predictor de duración:</b> Aprende cuánto debe durar cada fonema. Esto permite que el modelo sepa cuánto tiempo debe sonar cada sonido.</li>
    <li><b>Predictor de tono (F0) y energía:</b> Determinan cómo debe variar la altura de la voz (entonación) y la intensidad (fuerza) del sonido. Así el modelo puede expresar emociones o naturalidad.</li>
    <li><b>Generador de espectrograma mel:</b> Con toda esa información (texto + duración + tono + energía), el modelo genera un espectrograma Mel, una representación visual del sonido.</li>
    <li><b>Vocoder:</b> Convierte el espectrograma Mel en onda de audio, es decir, el sonido real de la voz.</li>
  </ol>
</div>

Note: Explicación del Flujo de FastSpeech 2 Paso a Paso:
FastSpeech 2 opera en un modelo que predice directamente la duración y las características prosódicas, permitiendo la generación paralela y eliminando la latencia y los errores acumulativos de los modelos autorregresivos (como Tacotron 2).
1. Encoder (Codificador): El texto (secuencia de caracteres o fonemas) es procesado por un Encoder basado en Transformer. Este codificador convierte la secuencia de texto en representaciones ocultas de alta dimensión.
2. Adaptador de Varianza (Variance Adaptor): Este es el componente distintivo y esencial de FastSpeech 2. Su función es modelar explícitamente la prosodia y el "tamaño" temporal antes de que se genere el espectrograma.
    ◦ Predictor de Duración: Estima cuántos frames de audio le corresponden a cada fonema.
    ◦ Regulador de Longitud (Length Regulator): Utiliza la duración predicha para expandir la secuencia de texto a la longitud temporal del audio objetivo, un paso que se realiza en paralelo. Al depender de alineaciones externas o procedimientos automáticos en lugar de la atención, se esquivan la mayoría de fallos de alineación.
    ◦ Predictor de Tono (Pitch): Predice el contorno de la frecuencia fundamental (F0), lo que define la entonación.
    ◦ Predictor de Energía: Estima la intensidad o volumen por unidad temporal, lo que es útil para controlar el énfasis.
3. Decoder (Decodificador): Una vez que la secuencia de texto ha sido alineada y condicionada por la prosodia (tono y energía) y la duración, el Decoder basado en Transformer toma esta representación expandida y predice todos los frames del Espectrograma de Mel en un único paso.
4. Vocoder Neuronal: El espectrograma de Mel (salida intermedia) es pasado a un vocoder (como HiFi-GAN) para generar la forma de onda de audio final


## Arquitecturas End-to-End
<a data-preview-link="img/VITS.png"><strong>Ejemplo: VITS</strong></a>

Representan el **paradigma actual**. Estos modelos unifican todo el proceso en una **única red neuronal** entrenada de principio a fin.

- Convierten **texto directamente** en una forma de onda de audio
- **Sin espectrogramas intermedios**
- **Calidad excepcional** con **latencia muy baja**


<div class="split-slide">
  <img src="img/VITS.png" alt="Arquitectura VITS">

  <ol class="text-col">
    <li><b>Texto de entrada:</b></li> El modelo recibe el texto que se quiere convertir en voz (por ejemplo: “Hola, ¿cómo estás?”).</li>
    <li><b>Codificador de texto:</b></li> Convierte el texto en una representación numérica (vectores) que capturan cómo deberían sonar los fonemas y su ritmo.</li>
    <li><b>Predictor estocástico de duración:</b></li> Aprende cuánto debe durar cada fonema o sílaba en el audio, pero de forma probabilística, permitiendo que la voz suene más natural y variable.</li>
    <li><b>Flujo normalizador (Normalizing Flow):</b></li> Transforma las representaciones del texto en una distribución más parecida a la del audio real. Esto crea un espacio latente que conecta el texto con el sonido sin depender de alineaciones externas.</li>
    <li><b>Generador (basado en HiFi-GAN):</b></li> A partir del espacio latente (ya ajustado por duración y flujo), el generador crea directamente la onda de audio — es decir, la voz sintética.</li>
    <li><b>Codificador posterior y discriminador (solo en entrenamiento):</b></li> El codificador posterior aprende de audios reales para que el modelo entienda cómo deberían sonar. El discriminador (parte tipo GAN) diferencia entre audio real y generado, ayudando al modelo a sonar más natural.</li>
  </ol>
</div>

Note: Resumen del Proceso de VITS:
1. Codificador de Texto: Convierte el texto de entrada en representaciones locales con contexto.
2. Mapeo Texto a Audio: Esta etapa, que incluye un predictor de duraciones y funciones flows (funciones invertibles y diferenciables), gestiona el alineamiento.
3. Predicción Estocástica de Duraciones: VITS utiliza esta técnica en lugar de un decoder autorregresivo. Aprende una distribución de posibles duraciones (a partir de ruido gaussiano simple transformado por un flow) para cada token. Esto introduce variación natural en el ritmo y la entonación.
4. Generador Neuronal: La secuencia de texto expandida (ajustada a la duración predicha) alimenta un generador tipo HiFi-GAN/WaveNet. Este bloque sintetiza la forma de onda final, optimizado mediante pérdidas adversarias GAN
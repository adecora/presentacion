La arquitectura de **word2speech** fue diseñada con una filosofía de **separación de responsabilidades** y **extensibilidad**.


## Interfaz de Línea de Comandos
[**`cli.py`**](https://github.com/adecora/proyecto-dislexia/blob/main/word2speech/cli.py)

- **Punto de entrada** para el usuario
- Implementada con la librería **click**
- Gestiona todos los **comandos, argumentos y parámetros**
- Experiencia de uso **moderna e intuitiva**


**cli.py**

```py [20: 1|5|13|22-31|44-48|63-70]
@click.group(invoke_without_command=True)
@click.option("--version", is_flag=True, help="Mostrar version")
@click.option("--verbose", "-v", is_flag=True, help="Verbose logging")
@click.pass_context
def cli(ctx, version, verbose):
    """word2speech: Herramienta CLI para el manejo de modelos TTS."""
    if verbose:
        logging.basicConfig(level=logging.DEBUG, format="[%(asctime)s] %(levelname)s: %(message)s", force=True)
    else:
        logging.basicConfig(level=logging.INFO, format="[%(asctime)s]: %(message)s", datefmt="%H:%M:%S", force=True)

    # Registra todos los modelos disponibles
    discover_models()

    if version:
        click.echo("word2speech version 1.0.3")
        return

    if ctx.invoked_subcommand is None:
        click.echo(ctx.get_help())

@cli.command()
@click.argument("text")
@click.option("-m", "--model", default="speechgen.io", help="Modelo TTS a usar (default: speechgen.io)")
@click.option("-o", "--output", default="out", help="Nombre del archivo de salida")
@click.option("--voice", help="Voz: male/female o nombre específico (p.ej., 'Alvaro')")
@click.option("--speed", type=float, help="Velocidad del habla: 0.1-2.0 (default: 1.0)")
@click.option("--pitch", help="Tono: low/normal/high or -20 to 20 (default: 0)")
@click.option("--emotion", help="Emoción: calm/energetic/neutral (default: neutral)")
@click.option("--contour", "-c", multiple=True, metavar="tiempo,tono", help="Detalle de entonación. %tiempo  duración (0 a 100), %tono entonación (-100 a 100)")
def speak(text, model, output, voice, speed, pitch, emotion, contour):
    """
    Generar voz a partir de texto usando modelos TTS.

    \b
    Ejemplos:
        word2speech speak "hola mundo"
        word2speech speak "hola" --voice female --speed 1.2 --emotion calm

    \b
    Para listar los modelos:
        word2speech models
    """
    tts_model = registry.get(model)
    if not tts_model:
        click.echo(f"Moldelo '{model}' no encontrado.", err=True)
        click.echo("Usa 'word2speech models' para ver los modelos disponibles.", err=True)
        sys.exit(1)

    options = {}
    if voice:
        options["voice"] = voice
    if speed:
        options["speed"] = speed
    if pitch:
        options["pitch"] = pitch
    if emotion:
        options["emotion"] = emotion
    if contour:
        options["contour"] = contour

    try:
        log.info(f'Generando audio para: "{text}"')
        audio, file_format, cost, balance = tts_model.generate(text, **options)

        output_file = f"{output}.{file_format}"
        with open(output_file, "wb") as f:
            f.write(audio)

        log.info(f'Audio generado "{output_file}" (coste: {cost}, saldo: {balance})')

    except Exception as e:
        click.echo(f"Error generando audio: {e}", err=True)
        sys.exit(1)
```


## Configuración
[**`config.py`**](https://github.com/adecora/proyecto-dislexia/blob/main/word2speech/config.py)

La aplicación busca la configuración siguiendo una estructura jerárquica:
1.  Fichero de configuración local: `.word2speech/config.yml`
2.  Configuración global del usuario: `$HOME/.config/word2speech/config.yml` o `$HOME/.word2speech/config.yml`


**config.py**

```py [14: 5|12|17|21|6]
class Config:
    """Clase para manejar la configuración de word2speech."""

    def __init__(self):
        self.config_dir = self._get_config_dir()
        self.config_file = self.config_dir / "config.yml"
        self._config = self._load_config()

    def _get_config_dir(self):
        """Obtiene el directorio de configuración."""
        # Primero buscamos en el directorio de trabajo actual
        local_config = Path.cwd() / ".word2speech"
        if local_config.exists():
            return local_config

        # Usamos la configuración global del usuario
        config_home = os.environ.get("XDG_CONFIG_HOME")
        if config_home:
            return Path(config_home) / "word2speech"
        else:
            return Path.home() / ".word2speech"
```


## Sistema de Plugins y Registro
[**`models.py`**](https://github.com/adecora/proyecto-dislexia/blob/main/word2speech/models.py), [**`plugins/`**](https://github.com/adecora/proyecto-dislexia/tree/main/word2speech/plugins)

El **corazón de la herramienta**. Esta arquitectura permite:

- **Registrar y gestionar** de forma centralizada diferentes modelos TTS
- Soporte para **APIs externas** y **modelos locales**
- **Incorporación de nuevas tecnologías** sin alterar el código base


**models.py**

```py [5|10|17-18|27-28|35|42-48|64]
"""
Unifica la inferencia y el registro de los modelos TTS
"""

import abc
import logging

log = logging.getLogger(__name__)

class TTSModel(abc.ABC):
    """Plantilla para todos los modelos TTS."""

    def __init__(self, model_id, name):
        self.model_id = model_id
        self.name = name or model_id

    @abc.abstractmethod
    def generate(self, text, **kwargs):
        """
        Genera audio de texto.

        Returns:
            Tuple de (audio_bytes, format, cost, balans)
        """
        pass

    @abc.abstractmethod
    def supports(self, feature):
        """Comprobar si el modelo admite una función específica"""
        pass

    def __str__(self):
        return f"{self.name} ({self.model_id})"

class TTSRegistry:
    """Registro de los modelos TTS."""

    def __init__(self):
        self._models = {}
        self._aliases = {}

    def register(self, model, aliases):
        """Registra un modelo TTS."""
        self._models[model.model_id] = model
        if aliases:
            for alias in aliases:
                self._aliases[alias] = model.model_id
        log.debug(f"Modelo TTS regitrado: {model}")

    def get(self, model_id):
        """Obtiene un modelo por ID o alias."""
        actual_id = self._aliases.get(model_id, model_id)
        return self._models.get(actual_id)

    def list_models(self):
        """Lista todos los modelos registrados."""
        return list(self._models.values())

    def list_model_ids(self):
        """Lista todos los IDs de los modelos"""
        return list(self._models.keys())

# Instancia global del regsitro de modelos
registry = TTSRegistry()
```


**plugins/**

<div class="split-code">

```py [14: 4-10|33-44|12-31]
class SpeechGenModel(TTSModel):
    """Modelo TTS de speechgen.io con interfaz unificada."""

    def __init__(self):
        super().__init__("speechgen.io", "SpeechGen.io")
        self.url = "https://speechgen.io/index.php?r=api/text"

        self.speaker_map = {"female": "Estrella", "male": "Alvaro"}
        self.pitch_map = {"low": -10, "normal": 0, "high": 10}
        self.emotion_map = {"calm": "good", "energetic": "evil", "neutral": "neutral"}

    def generate(self, text, **kwargs):
        """Generar audio utilizando la API de SpeechGen.io."""
        params = self._build_params(text, **kwargs)

        response = self._make_request(params)

        # Manejo de la respuesta de speechgen.io
        if response["status"] == 1:
            if "file" in response and "format" in response:
                file_url = response["file"]
                file_format = response["format"]
                audio = requests.get(file_url).content
                return (audio, file_format, response["cost"], response["balans"])
            else:
                raise HTTPError(f"404 Not Found: {response['error']}")
        else:
            if "login" in response["error"]:
                raise HTTPError(f"401 Unauthorized: {response['error']}")
            else:
                raise HTTPError(f"400 Bad Request: {response['error']}")

    def supports(self, feature):
        """Features soportadas por el modelo"""
        supported_features = {
            "ssml": True,
            "voices": True,
            "speed": True,
            "pitch": True,
            "emotions": True,
            "contour": True,
            "offline": False,
        }
        return supported_features.get(feature, False)

    def _build_params(self, text, **kwargs):
        model_config = config.get_model_config(self.model_id)

        params = {
            "token": config.get_api_key("speechgen"),
            "email": config.get_api_key("speechgen-email"),
            "voice": model_config.get("voice", "Alvaro"),
            "format": "wav",
            "speed": 1.0,
            "pitch": 0,
            "emotion": "neutral",
            "bitrate": 44100,
        }

        # Validamos parámetros obligatorios
        if not params["token"]:
            raise ValueError("Token es obligatorio para SpeechGen API. Para configurarlo: word2speech keys set speechgen TU_TOKEN")
        if not params["email"]:
            raise ValueError("Email es obligatorio para SpeechGen API. Para configurarlo: word2speech keys set speechgen-email TU_EMAIL")

        # Sobreescribimos los parámetros modificados
        if "voice" in kwargs:
            params["voice"] = self.speaker_map.get(kwargs["voice"].lower(), kwargs["voice"])
        if "speed" in kwargs:
            params["speed"] = kwargs["speed"]
        if "pitch" in kwargs:
            try:
                params["pitch"] = int(kwargs["pitch"])
            except ValueError:
                params["pitch"] = self.pitch_map.get(kwargs["pitch"].lower(), 0)
        if "emotion" in kwargs:
            params["emotion"] = self.emotion_map.get(kwargs["emotion"].lower(), params["emotion"])

        # Manejo específico de los puntos de contorno
        if "contour" in kwargs:
            text = format(Contour(kwargs["contour"]), text)

        params["text"] = text
        return params

    def _make_request(self, params):
        """Petición a al API de speechgen.io."""
        response = requests.post(self.url, data=params)
        return response.json()
```

```py [27: 4-18|12|20-25|65-76|41-63|44|78-111]
class ParlerModel(TTSModel):
    """Modelo TTS de Parler-TTS con interfaz unificada."""

    def __init__(self):
        if not DEPS_AVAILABLE:
            raise ImportError(
                "Las dependencias de Parler-TTS no están disponibles. Instalalas con: pip install git+https://github.com/huggingface/parler-tts.git"
            )

        super().__init__("parler-tts/parler-tts-mini-multilingual-v1.1", "Parler-TTS Multilingual")
        self.device = "cuda:0" if torch.cuda.is_available() else "cpu"
        self._model = None
        self._tokenizer = None
        self._description_tokenizer = None

        # Opciones de prosodia predefinidas
        self.speaker_map = {"male": "Steven's voice", "female": "Olivia's voice"}
        self.emotion_map = {"calm": "warm and soothing", "energetic": "clear and energetic", "neutral": "deep and whispering"}

    @property
    def model(self):
        """Lazy load model."""
        if self._model is None:
            self._model = ParlerTTSForConditionalGeneration.from_pretrained(self.model_id).to(self.device)
        return self._model

    @property
    def tokenizer(self):
        """Lazy load tokenizer."""
        if self._tokenizer is None:
            self._tokenizer = AutoTokenizer.from_pretrained(self.model_id)
        return self._tokenizer

    @property
    def description_tokenizer(self):
        """Lazy load description tokenizer."""
        if self._description_tokenizer is None:
            self._description_tokenizer = AutoTokenizer.from_pretrained(self.model.config.text_encoder._name_or_path)
        return self._description_tokenizer

    def generate(self, text, **kwargs):
        """Generar audio usando Parler-TTS"""
        # Genera el prompt con la descripción de la voz
        description = self._build_voice_description(**kwargs)

        # Tokeniza las entradas
        description_tokens = self.description_tokenizer(description, return_tensors="pt", padding=True, truncation=True)  # .input_ids.to(self.device)
        input_ids = description_tokens.input_ids.to(self.device)
        attention_mask = description_tokens.attention_mask.to(self.device)

        prompt_input_ids = self.tokenizer(text, return_tensors="pt").input_ids.to(self.device)

        # Genera el audio
        generation = self.model.generate(input_ids=input_ids, attention_mask=attention_mask, prompt_input_ids=prompt_input_ids)
        audio_arr = generation.cpu().numpy().squeeze()

        # Guardamos en un buffer y leemos los bytes
        buffer = BytesIO()
        sf.write(buffer, audio_arr, self.model.config.sampling_rate, format="WAV")
        buffer.seek(0)  # Salvaguarda incio del audio·
        audio_bytes = buffer.read()

        return (audio_bytes, "wav", 0, 0)

    def supports(self, feature):
        """Features soportadas por el modelo"""
        supported_features = {
            "ssml": False,
            "voices": True,  # Acepta female/male
            "speed": True,
            "pitch": True,
            "emotions": True,
            "contour": False,
            "offline": True,
        }
        return supported_features.get(feature, False)

    def _build_voice_description(self, **kwargs):
        """Construye la descripción de la voz para Parler-TTS."""
        model_config = config.get_model_config(self.model_id)

        speaker = model_config.get("speaker", "Steven's voice")
        speed = model_config.get("speed", "slow")
        pitch = model_config.get("pitch", "normal")
        emotion = model_config.get("emotion", "deep and whispering")

        if "voice" in kwargs:
            speaker = self.speaker_map.get(kwargs["voice"].lower(), speaker)
        if "speed" in kwargs:
            if kwargs["speed"] <= 0.5:
                speed = "very slow"
            elif kwargs["speed"] >= 1.5:
                speed = "normal"
            else:
                speed = "slow"
        if "pitch" in kwargs:
            try:
                pitch = int(kwargs["pitch"])
                if pitch <= -10:
                    pitch = "low"
                elif pitch >= 10:
                    pitch = "high"
                else:
                    pitch = "normal"
            except ValueError:
                pitch = kwargs["pitch"]
        if "emotion" in kwargs:
            emotion = self.emotion_map.get(kwargs["emotion"].lower(), emotion)

        # Construimos la descripción
        return f"{speaker} is {emotion}, speaking {speed} with a {pitch} pitch, recorded with very smooth intonation for natural prosody."
```

</div>


**plugins/\_\_init\_\_.py**

```py
def discover_models():
    """Descubre y registra todos los modelos disponibles."""
    from ..models import registry

    # Registramos el modelo de SpeechGen
    speechgen = SpeechGenModel()
    registry.register(speechgen, aliases=["speechgen", "default"])

    # Registramos Parler-TTS si sus dependencias están disponibles
    try:
        parler = ParlerModel()
        registry.register(parler, aliases=["parler"])
    except ImportError:
        pass

    # Registramos MMS-TTS si sus dependencias están disponibles
    try:
        parler = MMSModel()
        registry.register(parler, aliases=["mms"])
    except ImportError:
        pass
```


## Módulos Específicos
[**`modules/`**](https://github.com/adecora/proyecto-dislexia/tree/main/word2speech/modules)

Implementan funcionalidades avanzadas orientadas a la **intervención terapéutica**:

- [**`spell`**](https://github.com/adecora/proyecto-dislexia/blob/main/word2speech/modules/deletrear.py): Deletreo silábico de palabras para ejercicios de concienciación
- [**`prosody`**](https://github.com/adecora/proyecto-dislexia/blob/main/word2speech/modules/prosodia.py): Control fino de la prosodia mediante el estándar **SSML** (Speech Synthesis Markup Language)


## Sistema de Análisis
[**`analysis/`**](https://github.com/adecora/proyecto-dislexia/blob/main/word2speech/analysis/audio_analyzer.py)

Integra un modelo de predicción de **Mean Opinion Score (MOS)**:

- Utiliza [**UTMOS 2022 Strong**](https://github.com/tarepan/SpeechMOS)
- Un sistema de predicción de la Puntuación Media de Opinión (MOS) desarrollado por UTokyo-SaruLab y presentado en el **VoiceMOS Challenge 2022**
- Permite evaluar de forma **objetiva e instantánea** la calidad del audio generado


## Distribución
**PyPI**

- Empaquetada y distribuida en el [**Python Package Index (PyPI)**](https://pypi.org/project/word2speech/)
- Instalación sencilla mediante **`pipx`**
- **Diferentes niveles** de instalación:

  <!-- .slide: data-state="small-tables" -->
  | Instalación         | Descripción                                                        | Comando de Instalación                   |
  | :------------------ | :----------------------------------------------------------------- | :--------------------------------------- |
  | **Básica**          | Soporte de generación TTS únicamente con speechgen.io.             | `pipx install word2speech`               |
  | **Modelos Locales** | Añade la generación TTS con modelos locales (Parler-TTS, MMS-TTS). | `pipx install word2speech[local-models]` |
  | **Análisis**        | Permite la evaluación terapéutica de los audios con el módulo MOS. | `pipx install word2speech[analysis]`     |
  | **Completa**        | Instalación completa.                                              | `pipx install word2speech[all]`          |

Note: La modularidad permite adaptarse a diferentes necesidades de usuario y casos de uso.
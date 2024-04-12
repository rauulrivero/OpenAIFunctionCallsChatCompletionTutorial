# OpenAIFunctionCallsChatCompletionTutorial

## Guía básica para las llamadas de función en OpenAI

### 1. Configuración inicial

```python
from openai import OpenAI
import os
from dotenv import load_dotenv
import json

# Cargar las variables de entorno desde el archivo .env
load_dotenv()

# Configurar la clave de la API de OpenAI
client = OpenAI(
    api_key=os.getenv("OPENAI_API_KEY"),
)
```

### 2. Definir descripciones de funciones
Define las descripciones de las funciones que planeas utilizar en tu diálogo. Estas descripciones incluyen el nombre de la función, una breve descripción y detalles sobre los parámetros que espera la función.

```python
function_descriptions = [
    {
        "name": "consultar_api_vuelos",
        "description": "Descripción de la función consultar_api_vuelos",
        "parameters": {
            # Detalles de los parámetros esperados
        },
    },
    # Otras descripciones de funciones...
]
```

### 3. Definir las funciones (ejemplo de consulta a una api)
```python
import requests

def consultar_api_vuelos(url, params=None):
    """
    Realiza una consulta a una API utilizando la URL proporcionada y los parámetros (si los hay).

    Args:
    - url (str): La URL de la API a la que se va a realizar la consulta.
    - params (dict, opcional): Los parámetros de la consulta (por ejemplo, filtros o datos de búsqueda).

    Returns:
    - dict: Un diccionario que contiene la respuesta de la API, convertida a JSON.
    - None: Si ocurrió algún error durante la consulta.
    """
    try:
        # Realizar la solicitud GET a la API con los parámetros proporcionados
        response = requests.get(url, params=params)

        # Verificar si la solicitud fue exitosa (código de estado 200)
        if response.status_code == 200:
            # Convertir la respuesta a JSON y devolverla
            return response.json()
        else:
            # Imprimir un mensaje de error si la solicitud no fue exitosa
            print(f"Error al consultar la API. Código de estado: {response.status_code}")
            return None
    except Exception as e:
        # Capturar cualquier excepción que ocurra durante la consulta
        print(f"Error al realizar la consulta a la API: {e}")
        return None
    
    # OTRAS FUNCIONES....

```

 ### 4. Interactuar con el modelo de lenguaje
```python

user_prompt = "When's the next flight from Amsterdam to New York?"

completion = client.chat.completions.create(
    model="gpt-3.5-turbo-0613",
    messages=[{"role": "user", "content": user_prompt}],
    functions=function_descriptions,
    function_call="auto",  # especificar la llamada de función automática
)
output = completion.choices[0].message
```

### 5. Llamadas de función manual
```python
params = json.loads(output.function_call.arguments)

chosen_function = eval(output.function_call.name)
flight = chosen_function(**params)
```

### 6. Incorporar resultados de función en la respuesta
```python

second_completion = client.chat.completions.create(
    model="gpt-3.5-turbo-0613",
    messages=[
        {"role": "user", "content": user_prompt},
        {"role": "function", "name": output.function_call.name, "content": flight},
    ],
    functions=function_descriptions,
)
response = second_completion.choices[0].message.content
```


### 7. Utilizar la función en un flujo de conversación (estas funciones sirven para realizar lo anterior)
Con la primera función obtendremos la función que se va a utilizar y los parámetros recogidos del user_prompt. Con la segunda función obtendremos la salida de la función utilizado y devolveremos el contenido al usuario en formato frase.

```python

def detect_function(prompt):
    completion = client.chat.completions.create(
        model="gpt-3.5-turbo-0613",
        messages=[{"role": "user", "content": prompt}],
        functions=function_descriptions,
        function_call="auto",  # especificar la llamada de función automática
    )
    output = completion.choices[0].message
    return output

def function_calling(prompt, function, content):
    """Give LLM a given prompt and get an answer."""

    completion = client.chat.completions.create(
        model="gpt-3.5-turbo-0613",
        messages=[{"role": "user", "content": prompt},
                  {"role":"function", "name": function, "content": content}],

        functions=function_descriptions,
        function_call="auto",  # specify the function call
    )

    output = completion.choices[0].message
    return output

```


### 8. Pasos para preparar y ejecutar el ejemplo.

Para configurar y ejecutar el ejemplo, sigue estos pasos:

1. Configuración de la API Key:
Abre o crea un archivo .env en el directorio del proyecto y añade tu API Key de OpenAI de la siguiente manera:

```makefile
OPENAI_API_KEY=tu_api_key_aquí
```
Asegúrate de reemplazar tu_api_key_aquí con tu clave de API real.

2. Instalación de dependencias:
Abre una terminal y ejecuta el siguiente comando para instalar las librerías necesarias:

```bash
pip install openai dotenv
```

Este comando instalará las librerías openai y dotenv, que son necesarias para interactuar con la API de OpenAI y gestionar las variables de entorno, respectivamente.

Siguiendo estos pasos, tendrás todo listo para ejecutar el código de ejemplo proporcionado.


### Bibliografía
Vídeo interesante: "OpenAI Function Calling - Full Beginner Tutorial" https://www.youtube.com/watch?v=aqdWSYWC_LI

Página de referencia: https://cookbook.openai.com/examples/how_to_call_functions_with_chat_models

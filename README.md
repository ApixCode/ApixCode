
from flask import Flask, request, jsonify
from flask_cors import CORS
import asyncio
from aiohttp import ClientSession
import re
import time
import requests
import random
import string
import base64
import datetime
import time
import requests
import json
import os
from urllib.parse import urlparse, parse_qs
from bs4 import BeautifulSoup



app = Flask(__name__)
CORS(app)  # Enable CORS for all routes

headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'DNT': '1',  # Do Not Track Request Header
    'Connection': 'close',
    'Referer': 'https://linkvertise.com',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x66) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36'
}

key_regex = r'let content = \("([^"]+)"\);'

async def fetch(session, url, referer):
    headers["Referer"] = referer
    async with session.get(url, headers=headers) as response:
        content = await response.text()
        if response.status != 200:
            return None, response.status, content
        return content, response.status, None

async def process_link(hwid):
    endpoints = [
        {
            "url": f"https://flux.li/android/external/start.php?HWID={hwid}",
            "referer": "https://linkvertise.com"
            },
        {
            "url": "https://flux.li/android/external/check1.php?hash={hash}",
            "referer": "https://linkvertise.com"
            },
        {
            "url": "https://flux.li/android/external/main.php?hash={hash}",
            "referer": "https://linkvertise.com"
        }
    ]
    
    async with ClientSession() as session:
        for i, endpoint in enumerate(endpoints):
            url = endpoint["url"]
            referer = endpoint["referer"]
            content, status, error_content = await fetch(session, url, referer)
            if error_content:
                return {
                    "status": "error",
                    "message": f"Failed to bypass at step {i} | Status code: {status}",
                    "content": error_content
                }

            if i == len(endpoints) - 1:  # End of the bypass
                match = re.search(key_regex, content)
                if match:
                    return {
                        "status": "success",
                        "key": match.group(1)
                    }
                else:
                    return {
                        "status": "error",
                        "message": "Bypassed not successfully! No key found.",
                        "content": content
                    }



@app.route('/api/fluxus', methods=['GET'])
def handle_request():
    start_time = time.time()  # Start time
    link = request.args.get('link')
    if not link:
        return jsonify({"error": "No link provided"}), 400

    hwid = link.split("HWID=")[-1]
    result = asyncio.run(process_link(hwid))
    end_time = time.time()  # End time
    execution_time = end_time - start_time  # Calculate execution time
    result['execution_time'] = execution_time  # Add execution time to the result
    return jsonify(result)


@app.route('/')
def huy():
    return 'Error use /commands for api commands'






@app.route('/api/bloxfruits/stock', methods=['GET'])
def get_bloxfruits_stock():
    try:
        # Realiza la solicitud a la API externa
        response = requests.get('https://nexusapi-qg8a.onrender.com/api/supported')
        response.raise_for_status()  # Lanza un error si el status code no es 200
        
        # Convertir la respuesta a JSON
        data = response.json()
        
        # Retorna el JSON con una indentación de 2 espacios
        return jsonify(data)
    
    except requests.exceptions.RequestException as e:
        # Captura cualquier error que ocurra durante la solicitud
        return jsonify({'error': str(e)}), 500





@app.route('/TestHub/addlink', methods=['GET'])
def get_fluxus_data():
    # Obtener el parámetro 'link' desde la solicitud (query parameter)
    link = request.args.get('url')
    
    if not link:
        return jsonify({'error': 'Missing Link'}), 400

    # Construir la URL con el parámetro link
    api_url = f'https://ethos.kys.gay/api/free/bypass?url={link}'
    
    try:
        # Hacer la solicitud a la API externa
        response = requests.get(api_url)
        response.raise_for_status()  # Lanza un error si el estado no es 200
        
        # Convertir la respuesta a JSON
        data = response.json()
        
        # Verificar si hay algún error en la respuesta
        if 'error' in data:
            return jsonify({'error': 'no support'}), 400
        
        # Acceder al campo "result" en la respuesta
        if 'result' in data:
            result_data = data['result']
        else:
            return jsonify({'error': 'error2'}), 500

        # Retornar los datos de key en formato JSON
        return jsonify({'Here the link bypassed': data})
    
    except requests.exceptions.HTTPError:
        # Captura errores HTTP y devuelve un mensaje genérico
        return jsonify({'error': 'No Support.'}), 400
    except requests.exceptions.RequestException:
        # Captura cualquier otro error que ocurra durante la solicitud
        return jsonify({'error': 'No Support.'}), 500













def generate_random_hwid_flux(length=96):
    letters_and_digits = string.ascii_lowercase + string.digits
    return ''.join(random.choice(letters_and_digits) for _ in range(length))


def generate_random_hwid_arc(length=18):
    letters_and_digits = string.ascii_lowercase + string.digits
    return ''.join(random.choice(letters_and_digits) for _ in range(length))


def generate_random_id(length=64):
    letters_and_digits = string.ascii_lowercase + string.digits
    return ''.join(random.choice(letters_and_digits) for _ in range(length))

#
@app.route('/flux_gen', methods=['GET'])
def flux_gen():
    random_hwid = generate_random_hwid_flux()
    url = f"https://flux.li/android/external/start.php?HWID={random_hwid}"
    return jsonify({"url": url})


@app.route('/arc_gen', methods=['GET'])
def arc_gen():
    random_hwid = generate_random_hwid_arc()
    url = f"https://spdmteam.com/key-system-1?hwid={random_hwid}&zone=Europe/Rome&os=android"
    return jsonify({"url": url})


@app.route('/delta_gen', methods=['GET'])
def delta_gen():
    random_id = generate_random_id()
    url = f"https://gateway.platoboost.com/a/8?id={random_id}"
    return jsonify({"url": url})


@app.route('/hydro_gen', methods=['GET'])
def hydro_gen():
    random_id = generate_random_id()
    url = f"https://gateway.platoboost.com/a/2589?id={random_id}"
    return jsonify({"url": url})


@app.route('/boost.ink', methods=['GET'])
def extract():
    # Get the URL from query parameters
    url = request.args.get('url')
    
    if not url:
        return jsonify({"error": "URL is required"}), 400
    
    try:
        # Fetch the webpage content
        response = requests.get(url)
        html_content = response.text

        # Regex pattern to match the Base64 value inside bufpsvdhmjybvgfncqfa attribute
        pattern = r'bufpsvdhmjybvgfncqfa="([^"]+)"'

        # Find all matches
        matches = re.findall(pattern, html_content)

        if matches:
            # Extract the first match
            base64_value = matches[0]

            # Decode the Base64 value
            decoded_value = base64.b64decode(base64_value).decode("utf-8")
            
            return jsonify({"decoded_value": decoded_value})
        else:
            return jsonify({"error": "No matches found"}), 404
    except Exception as e:
        return jsonify({"error": str(e)}), 500





# Endpoint de status/health
@app.route('/status', methods=['GET'])
def health_check():
    response = {
        'status': 'Working',
        'timestamp': datetime.datetime.now().isoformat()
    }
    return jsonify(response), 200




i = 0
platoboost = "https://gateway.platoboost.com/a/8?id="
discord_webhook_url = "https://discord.com/api/webhooks/1292008449940656224/Rzb56DV33ZhROyQcEm1qEftF44_CRUxqpMowa3ZyGwhFt8W3moPGJkuLYPb8ZaE0Woq8" # enter your webhook if security check detected



def time_convert(n):
    hours = n // 60
    minutes = n % 60
    return f"{hours} Hours {minutes} Minutes"

def send_discord_webhook(link):
    payload = {
        "embeds": [{
            "title": "Security Check!",
            "description": f"**Please solve the Captcha**: [Open]({link})",
            "color": 5763719
        }]
    }

    try:
        response = requests.post(discord_webhook_url, json=payload)
        response.raise_for_status()
    except requests.exceptions.RequestException as error:
        print(f"\033[31m ERROR \033[0m Error: {error}")

def sleep(ms):
    time.sleep(ms / 1000)

def get_turnstile_response():
    time.sleep(1)
    return "turnstile-response"

def delta(url):
    start_time = time.time()
    try:
        parsed_url = urlparse(url)
        query_params = parse_qs(parsed_url.query)
        id = query_params.get('id', [None])[0]

        if not id:
            raise ValueError("Invalid URL: 'id' parameter is missing")

        response = requests.get(f"https://api-gateway.platoboost.com/v1/authenticators/8/{id}")
        response.raise_for_status()
        already_pass = response.json()

        if 'key' in already_pass:
            time_left = time_convert(already_pass['minutesLeft'])
            print(f"\033[32m INFO \033[0m Time left:  \033[32m{time_left}\033[0m - KEY: \033[32m{already_pass['key']}\033[0m")
            return {
                "status": "success",
                "key": already_pass['key'],
                "time_left": time_left
            }

        captcha = already_pass.get('captcha')

        if captcha:
            print("\033[32m INFO \033[0m hCaptcha detected! Trying to resolve...")
            # If captcha exists, make sure to solve it before continuing
            response = requests.post(
                f"https://api-gateway.platoboost.com/v1/sessions/auth/8/{id}",
                json={
                    "captcha": get_turnstile_response(),
                    "type": "Turnstile"
                }
            )
        else:
            # if no captcha, continue without it
            response = requests.post(
                f"https://api-gateway.platoboost.com/v1/sessions/auth/8/{id}",
                json={}
            )

        if response.status_code != 200:
            security_check_link = f"{platoboost}{id}"
            send_discord_webhook(security_check_link)
            raise Exception("Security Check, Notified on Discord!")

        loot_link = response.json()
        sleep(1000)
        decoded_lootlink = requests.utils.unquote(loot_link['redirect'])
        parsed_loot_url = urlparse(decoded_lootlink)
        r_param = parse_qs(parsed_loot_url.query)['r'][0]
        decoded_base64 = base64.b64decode(r_param).decode('utf-8')
        tk = parse_qs(urlparse(decoded_base64).query)['tk'][0]
        sleep(5000)

        response = requests.put(f"https://api-gateway.platoboost.com/v1/sessions/auth/8/{id}/{tk}")
        response.raise_for_status()

        response_plato = requests.get(f"https://api-gateway.platoboost.com/v1/authenticators/8/{id}")
        pass_info = response_plato.json()

        if 'key' in pass_info:
            time_left = time_convert(pass_info['minutesLeft'])
            execution_time = time.time() - start_time
            print(f"\033[32m INFO \033[0m Time left:  \033[32m{time_left}\033[0m - KEY: \033[32m{pass_info['key']}\033[0m")
            return {
                "status": "success",
                "key": pass_info['key'],
                
                "time taken": f"{execution_time:.2f} seconds"
            }

    except Exception as error:
        print(f"\033[31m ERROR \033[0m Error: {error}")
        execution_time = time.time() - start_time
        return {
            "status": "error",
            "error": "please solve the hcaptcha nigga",
            "time taken": f"{execution_time:.2f} seconds"
        }

@app.route('/bypass/delta', methods=['GET'])
def deltax():
    url = request.args.get('url')
    if not url:
        return jsonify({"error": "Missing 'url' parameter"}), 400

    result = delta(url)
    return jsonify(result)







@app.route('/TestHub/executorbypass', methods=['GET'])
def handle_api_request():  # Cambiado de 'handle_request' a 'handle_api_request'
    start_time = time.time()  # Start time
    url = request.args.get('url')

    if not url:
        return jsonify({"error": "Missing 'url' Put Your Link"}), 400

    # Caso 1: Verificar si la URL empieza con "https://gateway.platoboost.com/a/8?id="
    if url.startswith("https://gateway.platoboost.com/a/8?id="):
        result = delta(url)  # Ejecutar la lógica delta
        return jsonify(result)

    # Caso 2: Verificar si la URL empieza con "https://flux.li/android/external/start.php?HWID="
    elif url.startswith("https://flux.li/android/external/start.php?HWID="):
        hwid = url.split("HWID=")[-1]  # Extraer el HWID de la URL
        result = asyncio.run(process_link(hwid))  # Ejecutar la lógica de fluxus
        end_time = time.time()  # End time
        execution_time = end_time - start_time  # Calculate execution time
        result['execution_time'] = execution_time  # Add execution time to the result
        return jsonify(result)

    # Si la URL no coincide con ninguno de los casos
    return jsonify({"error": "Invalid URL/no support"}), 400









# Ruta principal para buscar scripts
@app.route('/search_scripts', methods=['GET'])
def search_scripts():
    # Obtener el parámetro 'q' de la URL
    query = request.args.get('q')
    
    # Verificar si se proporcionó un parámetro de búsqueda
    if not query:
        return jsonify({"error": "No query provided"}), 400
    
    # URL base de la API externa
    url = f"https://scriptblox.com/api/script/search?q={query}"
    
    try:
        # Realizar la solicitud GET a la API externa
        response = requests.get(url)
        
        # Verificar si la solicitud fue exitosa
        if response.status_code == 200:
            data = response.json()  # Convertir la respuesta en JSON
            
            # Verificar si contiene resultados
            if 'result' in data and 'scripts' in data['result']:
                scripts_info = []
                
                # Recorrer los scripts y extraer title, script y key
                for script in data['result']['scripts']:
                    script_data = {
                        "title": script.get("title"),    # Agregar el título
                        "script": script.get("script"),  # El código del script
                        "key": script.get("key")         # Si requiere clave o no
                    }
                    scripts_info.append(script_data)
                
                # Incluir la lista de scripts dentro de un campo "result"
                result = {
                    "result": scripts_info
                }
                
                return jsonify(result)  # Devolver la respuesta con el campo "result"
            else:
                return jsonify({"error": "No results found"}), 404
        else:
            return jsonify({"error": f"Error from API, status code: {response.status_code}"}), response.status_code
    
    except Exception as e:
        return jsonify({"error": str(e)}), 500





if __name__ == '__main__':
    app.run(host='0.0.0.0')

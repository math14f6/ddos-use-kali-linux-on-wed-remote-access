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
    git clone https://github.com/your-username/flask-ddos-control.git
    cd flask-ddos-control
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


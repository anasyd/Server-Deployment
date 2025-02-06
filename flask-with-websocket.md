Flask deployment guide that covers:

- Preparing your Flask app for WebSocket support using Eventlet
- Running the Flask app with Gunicorn (using Eventlet as the worker)
- Configuring Nginx as a reverse proxy (with WebSocket support)
- Creating a systemd service file to manage your Flask application

---

# Flask Deployment Guide with WebSocket Support

## 1. Update Your Flask Application for Eventlet

To support WebSockets using Eventlet, update your Flask app (typically in your `app.py` or main file) with the following at the very top:

```python
import eventlet
eventlet.monkey_patch()
```

Then, initialize your SocketIO instance with Eventlet and allow CORS:

```python
from flask_socketio import SocketIO
# Assuming your Flask app is defined as `app`
socketio = SocketIO(app, async_mode='eventlet', cors_allowed_origins="*")
```

Ensure your `requirements.txt` includes the necessary packages:

```
Flask
Flask-SocketIO
eventlet
gunicorn
```

## 2. Set Up Your Python Environment

1. **Clone Your Flask Repository** (use your SSH configuration if needed):
    
    ```sh
    git clone git@github.com:user/flask_repo.git
    cd flask_repo
    ```
    
2. **Create and Activate a Virtual Environment:**
    
    ```sh
    python3 -m venv env
    source env/bin/activate
    ```
    
3. **Install Dependencies:**
    
    ```sh
    pip install -r requirements.txt
    ```
    

## 3. Running the Flask App with Gunicorn and Eventlet

Use Gunicorn with the Eventlet worker to serve your app. Test it locally with:

```sh
gunicorn -k eventlet --workers 1 --bind 0.0.0.0:5000 --timeout 120 app:app
```

> **Note:** Replace `app:app` with the appropriate module and application variable if different.

## 4. Create a systemd Service for Your Flask App

To manage your Flask app as a service, create a systemd unit file. For example:

4. **Create the Service File:**
    
    ```sh
    sudo nano /etc/systemd/system/your_app.service
    ```
    
5. **Paste the Following Configuration:**  
    (Adjust paths, usernames, and filenames as needed.)
    
    ```ini
    [Unit]
    Description=Gunicorn instance to serve your Flask app with WebSocket support
    After=network.target
    
    [Service]
    User=ubuntu
    Group=ubuntu
    WorkingDirectory=/home/ubuntu/your_app
    Environment="PATH=/home/ubuntu/your_app/env/bin"
    ExecStart=/home/ubuntu/your_app/env/bin/gunicorn -k eventlet --workers 1 --bind 0.0.0.0:5000 --timeout 120 --access-logfile /home/ubuntu/your_app/logs/access.log --error-logfile /home/ubuntu/your_app/logs/error.log app:app
    
    [Install]
    WantedBy=multi-user.target
    
```
    
6. **Reload systemd, Start and Enable the Service:**
    
    ```sh
    sudo systemctl daemon-reload
    sudo systemctl start your_app.service
    sudo systemctl enable your_app.service
    ```
    

## 5. Configuring Nginx as a Reverse Proxy for Flask with WebSocket Support

You can use the `conf.d` method to configure Nginx for your Flask application.

7. **Create an Nginx Configuration File:**
    
    ```sh
    sudo nano /etc/nginx/conf.d/your_app.conf
    ```
    
8. **Paste the Following Configuration:**  
    (Adjust `server_name`, IP addresses, and ports as necessary.)
    
    ```nginx
    server {
        listen 80;
        server_name your_domain.com;
    
        location / {
            proxy_pass http://localhost:5000;  # Your Flask app is running on port 5000
            proxy_http_version 1.1;
    
            # WebSocket-specific headers
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
    
            # Common proxy headers
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
    
            # Optional: Disable caching for WebSocket connections
            proxy_cache_bypass $http_upgrade;
    
            # Optional: Increase timeout for long-lived WebSocket connections
            proxy_read_timeout 600;
        }
    }
    ```
    
9. **Test and Restart Nginx:**
    
    ```sh
    sudo nginx -t
    sudo systemctl restart nginx
    ```
    

Your Flask application, with WebSocket support via Eventlet, should now be accessible through Nginx on port 80.

---

**DONE**

# Traffic Route Optimization App

This repository contains a Flask-based web application that uses Dijkstra's algorithm and the Google Maps API to find the shortest traffic route between locations. This README will guide you through the process of setting up the application on a Linux system and deploying it using Gunicorn and Nginx.

## Prerequisites

Before you begin, ensure you have the following:

- A Linux-based system (e.g., Ubuntu)
- A **Google Maps API Key**
- A domain or public IP address (optional but recommended)
- Basic knowledge of Linux commands

## 1. Update System Packages

Start by updating your system to ensure all packages are up-to-date:

```bash
sudo apt-get update
sudo apt-get upgrade
2. Install Required Software
2.1 Install Python and Pip
The application requires Python 3 and Pip, a package manager for Python. To install them, run:

bash
Copy code
sudo apt-get install python3 python3-pip
2.2 Install Flask and Other Python Packages
Next, install Flask and the required Python libraries:

bash
Copy code
pip3 install flask googlemaps gunicorn networkx
2.3 Install Nginx
Nginx will serve as the reverse proxy for your Flask application:

bash
Copy code
sudo apt-get install nginx
3. Set Up the Flask Application
3.1 Clone or Create Your Flask App
Navigate to the directory where you want to host the application and clone this repository or create a new one.

bash
Copy code
git clone https://github.com/your-repo-url/flask-app.git
cd flask-app
3.2 Test the Flask App
Run the Flask app to ensure it works:

bash
Copy code
python3 app.py
The app should be accessible locally at http://localhost:5000.

3.3 Update Flask for Production
In the app.py, ensure the Flask app binds to 0.0.0.0 to be accessible over the network:

python
Copy code
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
4. Set Up Gunicorn
Gunicorn is a production-ready WSGI HTTP server for Python web applications.

4.1 Install Gunicorn
bash
Copy code
pip3 install gunicorn
4.2 Run Gunicorn
To serve the Flask app using Gunicorn, navigate to your app's directory and run:

bash
Copy code
gunicorn --bind 127.0.0.1:5000 app:app
Replace app:app with your app’s entry point (i.e., the filename app.py and the Flask app object app).
5. Configure Nginx as a Reverse Proxy
Nginx will forward incoming traffic to Gunicorn, which will serve your Flask app.

5.1 Edit Nginx Configuration
Open the default Nginx configuration file:

bash
Copy code
sudo nano /etc/nginx/sites-available/default
Replace the content or modify it with the following configuration:

nginx
Copy code
server {
    listen 80;
    server_name your_server_ip_or_domain;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
5.2 Test Nginx Configuration
bash
Copy code
sudo nginx -t
If no errors, restart Nginx:

bash
Copy code
sudo systemctl restart nginx
5.3 Open Ports in the Firewall
Allow port 80 for HTTP traffic:

bash
Copy code
sudo ufw allow 'Nginx Full'
sudo ufw enable
6. Set Up Gunicorn as a Systemd Service (Optional)
To ensure Gunicorn runs automatically and restarts after a reboot, you can configure it as a systemd service.

6.1 Create a Gunicorn Service File
bash
Copy code
sudo nano /etc/systemd/system/flaskapp.service
Add the following content:

ini
Copy code
[Unit]
Description=Gunicorn instance to serve Flask app
After=network.target

[Service]
User=your_linux_username
Group=www-data
WorkingDirectory=/path/to/your/app
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind 127.0.0.1:5000 app:app

[Install]
WantedBy=multi-user.target
6.2 Enable and Start the Service
bash
Copy code
sudo systemctl daemon-reload
sudo systemctl start flaskapp
sudo systemctl enable flaskapp
Check the status of the service:

bash
Copy code
sudo systemctl status flaskapp
7. Access Your Application
Your application should now be running and accessible via your server's public IP address or domain name:

URL: http://your-server-ip-or-domain
Checking Logs for Troubleshooting
Nginx logs:

bash
Copy code
sudo tail -f /var/log/nginx/error.log
Gunicorn logs:

bash
Copy code
sudo journalctl -u flaskapp
8. Additional Considerations
8.1 Setting Up HTTPS with Let's Encrypt (Optional)
You can secure your app using HTTPS with Let's Encrypt:

Install Certbot:

bash
Copy code
sudo apt-get install certbot python3-certbot-nginx
Run Certbot to generate SSL certificates:

bash
Copy code
sudo certbot --nginx -d your_domain
Certbot will configure Nginx to serve your app via HTTPS.

8.2 Debugging 502 Bad Gateway Errors
If you encounter a 502 Bad Gateway error, try these steps:

Ensure Gunicorn is running:

bash
Copy code
ps aux | grep gunicorn
Check Nginx error logs:

bash
Copy code
sudo tail -f /var/log/nginx/error.log
Restart both services:

bash
Copy code
sudo systemctl restart nginx
sudo systemctl restart flaskapp
9. Conclusion
You have successfully set up the Traffic Route Optimization app using Flask, Gunicorn, and Nginx on a Linux server. You can further enhance the deployment by adding HTTPS for security and monitoring logs for any issues.



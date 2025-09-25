# Django App Deployment Guide - Ubuntu

Deploy Django app on Ubuntu

**App Location**: `/home/ubuntu/my_project`

## Prerequisites
- Instance running Ubuntu
- Domain access to configure DNS (yourdomain.com)
- Django app with `requirements.txt` and `manage.py`

## Step 1: Prepare Your Ubuntu Instance

Update system and install required packages:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv nginx -y
```

Step 2: Set Up Virtual Environment and Install Dependencies

Clone and navigate to the Django app directory and create a virtual environment:
```bash
cd /home/ubuntu/
git clone https://github.com/your-username/my-project
cd my-project

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Step 3: Test Your Django App

Verify the Django app works correctly by running:
```bash
python manage.py runserver
```
Test locally, then stop the development server (Ctrl+C).

Step 4: Create a Gunicorn Service File

Create systemd service file for Gunicorn:
```bash
sudo nano /etc/systemd/system/my-project.service

[Unit]
Description=Gunicorn instance to serve my-project
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/my_project
Environment="PATH=/home/ubuntu/my_project/venv/bin"
ExecStart=/home/ubuntu/my_project/venv/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/my_project/my-project.sock my_project.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
```

Step 5: Start and Enable the Service

Start Gunicorn and enable it to run on boot:
```bash
sudo systemctl start my-project
sudo systemctl enable my-project
sudo systemctl status my-project
```

Step 6: Configure Nginx

Create Nginx configuration file for your domain:
```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

Add this configuration:
```bash
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/my_project/my-project.sock;
    }
}
```

Step 7: Enable the Nginx Site

Enable the site and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

Step 8: Configure Domain DNS

In your domain registrar/DNS provider:

Create an A record for the domain:

Name/Host: @ (or leave blank, depending on provider)

Type: A

Value/Points to: Your Ubuntu instance's public IP address

TTL: 300 (or default)

Note: The @ symbol represents the root domain (e.g., yourdomain.com). Some DNS providers may use a blank field instead. Refer to their specific instructions.

Step 9: Test Your Deployment

Wait for DNS propagation (5-30 minutes), then test:
```bash
curl -I http://yourdomain.com
```

Step 10: Final Verification

Your site should now be accessible at:

http://yourdomain.com

Troubleshooting Commands

If you encounter issues:
```bash
# Check service status
sudo systemctl status my-project

# Check Nginx status
sudo systemctl status nginx

# View service logs
sudo journalctl -u my-project

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Grant execute permissions, allowing Nginx to traverse the directory path
sudo chmod o+x /home/ubuntu /home/ubuntu/my_project

# Test Nginx configuration
sudo nginx -t
```

Important Notes

⚠️ Ensure your Django app’s wsgi.py is properly configured.

⚠️ Remove or comment out python manage.py runserver calls in production.

💡 Ensure your instance has adequate resources for expected traffic.

📊 Consider setting up monitoring and automated backups.

Final Result

Your Django app should now be live at: http://yourdomain.com

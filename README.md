
## 1. Create Project Directory

```bash
mkdir moodle-docker && cd moodle-docker
```

---

## 2.  `docker-compose.yml`

Create this file with the following content:

```yaml
version: '3.8'

services:
  mariadb:
    image: bitnami/mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      - MARIADB_ROOT_PASSWORD=rootpass
      - MARIADB_DATABASE=bitnami_moodle
      - MARIADB_USER=bn_moodle
      - MARIADB_PASSWORD=moodlepass
    volumes:
      - mariadb_data:/bitnami/mariadb

  moodle:
    image: bitnami/moodle:latest
    container_name: moodle
    restart: always
    environment:
      - MOODLE_USERNAME=admin
      - MOODLE_PASSWORD=adminpass123
      - MOODLE_EMAIL=admin@example.com
      - MOODLE_DATABASE_HOST=mariadb
      - MOODLE_DATABASE_PORT_NUMBER=3306
      - MOODLE_DATABASE_USER=bn_moodle
      - MOODLE_DATABASE_NAME=bitnami_moodle
      - MOODLE_DATABASE_PASSWORD=moodlepass
    depends_on:
      - mariadb
    volumes:
      - moodle_data:/bitnami/moodle
      - moodledata_data:/bitnami/moodledata
    ports:
      - "8088:8080"
    networks:
      - moodle-net

volumes:
  mariadb_data:
  moodle_data:
  moodledata_data:

networks:
  moodle-net:
    driver: bridge
```

---

## 3.  Run the Containers

```bash
docker compose up -d
```

Wait a few minutes. Then test via:

```
http://<your-server-ip>:8088
```

---

## 4.  NGINX Reverse Proxy Configuration

Create a file:

```bash
sudo nano /etc/nginx/sites-available/moodle.example.com
```

Add this config (note the port `8088`):

```nginx
server {
    listen 80;
    server_name moodle.example.com;

    location / {
        proxy_pass http://127.0.0.1:8088;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the config:

```bash
sudo ln -s /etc/nginx/sites-available/moodle.example.com /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## 5.  Enable HTTPS with Let's Encrypt (Optional)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d moodle.example.com
```

---

##  Test It

Open:

```
https://moodle.example.com
```

You should see the Moodle interface.

---



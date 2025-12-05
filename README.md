# Zot Registry

Registry privée légère sur VPS Docker.

**URL** : https://registry.haybot.fr

## Installation

### 1. Copier les fichiers sur le VPS

```bash
scp -r registry/ user@VPS_IP:/opt/registry/
```

### 2. Créer les utilisateurs htpasswd

```bash
cd /opt/registry

# Installer htpasswd si nécessaire
apt install apache2-utils -y

# Créer le fichier htpasswd
htpasswd -cB htpasswd artur
htpasswd -B htpasswd mathieu
```

### 3. DNS

Créer l'enregistrement A sur Cloudflare :
- `registry` → IP du VPS

### 4. Certificat TLS

```bash
# Installer certbot si nécessaire
apt install certbot -y

# Générer le certificat
certbot certonly --standalone -d registry.haybot.fr
```

### 5. Nginx

```bash
# Copier la config
cp nginx-registry.conf /etc/nginx/sites-available/registry
ln -s /etc/nginx/sites-available/registry /etc/nginx/sites-enabled/

# Tester et recharger
nginx -t && systemctl reload nginx
```

### 6. Démarrer Zot

```bash
cd /opt/registry
docker compose up -d
```

## Utilisation

### Login

```bash
docker login registry.haybot.fr
# Username: artur
# Password: ***
```

### Push une image

```bash
docker tag mon-app:latest registry.haybot.fr/mon-app:latest
docker push registry.haybot.fr/mon-app:latest
```

### Pull une image

```bash
docker pull registry.haybot.fr/mon-app:latest
```

### Depuis K3s

Créer le secret dans chaque namespace :

```bash
kubectl create secret docker-registry registry-secret \
  --docker-server=registry.haybot.fr \
  --docker-username=artur \
  --docker-password=MOT_DE_PASSE \
  -n mon-namespace
```

Utiliser dans le Deployment :

```yaml
spec:
  imagePullSecrets:
    - name: registry-secret
  containers:
    - name: app
      image: registry.haybot.fr/mon-app:latest
```

## Vérification

```bash
# Status
docker compose ps

# Logs
docker compose logs -f

# Test API
curl -u artur https://registry.haybot.fr/v2/_catalog
```

## Maintenance

```bash
# Restart
docker compose restart

# Mise à jour
docker compose pull
docker compose up -d

# Backup data
docker run --rm -v zot-data:/data -v $(pwd):/backup alpine tar czf /backup/zot-backup.tar.gz /data
```

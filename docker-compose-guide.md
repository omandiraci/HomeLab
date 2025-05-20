# Docker Compose Yapılandırma Kılavuzu

Bu kılavuz, Homelab Otomation projesinde kullanılan Docker Compose yapılandırmasını detaylı olarak açıklamaktadır.

## Genel Yapı

Docker Compose dosyası iki ana servis içermektedir:
1. Traefik (Reverse Proxy)
2. Portainer (Konteyner Yönetimi)

## Traefik Servisi

### Temel Yapılandırma
```yaml
services:
  traefik:
    image: traefik:v2.5
    platform: linux/arm64
    container_name: traefik
    restart: unless-stopped
```

#### Açıklamalar:
- **image**: Traefik'in 2.5 sürümünü kullanır
- **platform**: Apple Silicon (M1/M2) işlemciler için ARM64 mimarisini belirtir
- **container_name**: Konteyner adını "traefik" olarak belirler
- **restart**: Konteyner çöktüğünde otomatik yeniden başlatma politikası

### Güvenlik Ayarları
```yaml
security_opt:
  - no-new-privileges:true
```
- Konteyner içinde yeni ayrıcalıklar edinmeyi engeller
- Güvenlik açıklarını önlemek için önemli bir ayar

### Port Yapılandırması
```yaml
ports:
  - "80:80"    # HTTP
  - "443:443"  # HTTPS
  - "8080:8080" # Dashboard
```
- HTTP trafiği için 80 portu
- HTTPS trafiği için 443 portu
- Traefik dashboard'u için 8080 portu

### Volume Yapılandırması
```yaml
volumes:
  - /etc/localtime:/etc/localtime:ro
  - /var/run/docker.sock:/var/run/docker.sock:ro
  - ./traefik/config:/etc/traefik
  - ./traefik/certificates:/certificates
```
- **localtime**: Sistem saat dilimi (salt okunur)
- **docker.sock**: Docker API erişimi (salt okunur)
- **config**: Traefik yapılandırma dosyaları
- **certificates**: SSL sertifikaları

### Komut Parametreleri
```yaml
command:
  - --api.insecure=true
  - --providers.docker=true
  - --providers.docker.exposedbydefault=false
  - --providers.file.directory=/etc/traefik
  - --entrypoints.web.address=:80
  - --entrypoints.websecure.address=:443
  - --log.level=DEBUG
```
- **api.insecure**: Dashboard API'sini etkinleştirir
- **providers.docker**: Docker provider'ı etkinleştirir
- **exposedbydefault**: Varsayılan olarak tüm servisleri expose etmeyi devre dışı bırakır
- **entrypoints**: HTTP ve HTTPS giriş noktalarını tanımlar
- **log.level**: Debug seviyesinde loglama

### Traefik Etiketleri
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.traefik.rule=Host(`traefik.localhost`)"
  - "traefik.http.routers.traefik.service=api@internal"
  - "traefik.http.routers.traefik.entrypoints=web"
```
- Dashboard'a erişim için host kuralı
- API servisini dahili olarak yapılandırır
- Web endpoint'ini kullanır

## Portainer Servisi

### Temel Yapılandırma
```yaml
portainer:
  image: portainer/portainer-ce:latest
  platform: linux/arm64
  container_name: portainer
  restart: unless-stopped
  ports:
    - "9000:9000"
```
- En son Portainer CE sürümünü kullanır
- Apple Silicon uyumluluğu
- Otomatik yeniden başlatma
- 9000 portu üzerinden doğrudan erişim

### Volume Yapılandırması
```yaml
volumes:
  - /etc/localtime:/etc/localtime:ro
  - /var/run/docker.sock:/var/run/docker.sock:ro
  - portainer_data:/data
```
- Sistem saat dilimi
- Docker API erişimi
- Kalıcı veri depolama

### Portainer Etiketleri
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.portainer.rule=Host(`portainer.localhost`)"
  - "traefik.http.routers.portainer.entrypoints=web"
  - "traefik.http.services.portainer.loadbalancer.server.port=9000"
```
- Traefik entegrasyonu
- Host kuralı ile erişim
- Portainer'ın 9000 portunu kullanması

## Ağ Yapılandırması
```yaml
networks:
  proxy:
    driver: bridge
```
- Bridge tipinde "proxy" adında bir ağ oluşturur
- Servisler arası iletişim için kullanılır

## Volume Yapılandırması
```yaml
volumes:
  portainer_data:
```
- Portainer için kalıcı veri depolama alanı

## Kullanım Örnekleri

### Servisleri Başlatma
```bash
docker-compose up -d
```

### Servisleri Durdurma
```bash
docker-compose down
```

### Logları Görüntüleme
```bash
docker-compose logs -f
```

### Belirli Bir Servisi Yeniden Başlatma
```bash
docker-compose restart traefik
```

## Erişim Noktaları

1. **Traefik Dashboard**:
   - URL: http://traefik.localhost:8080

2. **Portainer**:
   - URL: http://portainer.localhost veya http://localhost:9000

## Güvenlik Önerileri

1. SSL sertifikalarını güvenli bir şekilde saklayın
2. Traefik yapılandırma dosyalarını düzenli yedekleyin
3. Portainer verilerini düzenli yedekleyin
4. Güvenlik güncellemelerini takip edin
5. Log dosyalarını düzenli kontrol edin 
# Docker Compose Kullanım Kılavuzu

Bu kılavuz, Docker Compose ile konteyner yönetimi için kullanılan temel komutları ve yapılandırma detaylarını açıklamaktadır.

## Temel Docker Compose Komutları

### 1. Servisleri Başlatma
```bash
docker-compose up -d
```
- `-d` parametresi arka planda çalıştırır
- Tüm servisleri docker-compose.yml dosyasına göre başlatır
- Ağ oluşturur ve tüm konteynerleri başlatır

### 2. Servisleri Durdurma
```bash
docker-compose down
```
- Tüm konteynerleri durdurur
- Ağları kaldırır
- Volumeleri siler (--volumes parametresi ile)

### 3. Logları Görüntüleme
```bash
docker-compose logs -f
```
- Tüm servislerin loglarını gösterir
- `-f` parametresi gerçek zamanlı logları gösterir
- Belirli bir servis için: `docker-compose logs <servis_adi>`

## Homelab Otomation Docker Compose Yapılandırması

```yaml
services:
  traefik:
    image: traefik:v2.5
    platform: linux/arm64  # Apple Silicon için platform belirtimi
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik/config:/etc/traefik
      - ./traefik/certificates:/certificates
    command:
      - --api.insecure=true
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --providers.file.directory=/etc/traefik
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --log.level=DEBUG
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.localhost`)"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.entrypoints=web"
    networks:
      - proxy

  portainer:
    image: portainer/portainer-ce:latest
    platform: linux/arm64
    container_name: portainer
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - portainer_data:/data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.localhost`)"
      - "traefik.http.routers.portainer.entrypoints=web"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
    networks:
      - proxy

networks:
  proxy:
    driver: bridge

volumes:
  portainer_data:
```

## Yapılandırma Detayları

### 1. Traefik Reverse Proxy
- **Platform**: Apple Silicon (M1/M2) için linux/arm64
- **Portlar**: 
  - 80: HTTP
  - 443: HTTPS
  - 8080: Dashboard
- **Volumes**:
  - `/etc/localtime`: Sistem saat dilimi
  - `/var/run/docker.sock`: Docker API erişimi
  - `./traefik/config`: Traefik yapılandırma dosyaları
  - `./traefik/certificates`: SSL sertifikaları
- **Özellikler**:
  - Docker provider aktif
  - HTTP ve HTTPS endpoint'leri
  - Debug log seviyesi
  - Dashboard erişimi (traefik.localhost)

### 2. Portainer
- **Platform**: Apple Silicon (M1/M2) için linux/arm64
- **Portlar**:
  - 9000: Web arayüzü
- **Volumes**:
  - `/etc/localtime`: Sistem saat dilimi
  - `/var/run/docker.sock`: Docker API erişimi
  - `portainer_data`: Kalıcı veri depolama
- **Özellikler**:
  - Traefik ile entegrasyon
  - Web arayüzü (portainer.localhost veya localhost:9000)
  - Port 9000 üzerinden doğrudan erişim

## Güvenlik Özellikleri

1. **Konteyner İzolasyonu**:
   ```yaml
   security_opt:
     - no-new-privileges:true
   ```

2. **Read-only Sistem Dosyaları**:
   ```yaml
   volumes:
     - /etc/localtime:/etc/localtime:ro
   ```

3. **Ağ İzolasyonu**:
   ```yaml
   networks:
     - proxy
   ```

## Erişim Noktaları

1. **Traefik Dashboard**:
   - URL: http://traefik.localhost:8080
   - Port: 8080

2. **Portainer**:
   - URL: http://portainer.localhost veya http://localhost:9000
   - Port: 9000

## Kurulum Adımları

1. **Dizin Yapısı Oluşturma**:
   ```bash
   mkdir -p traefik/config traefik/certificates
   ```

2. **Traefik Yapılandırma Dosyası**:
   ```bash
   touch traefik/config/traefik.yml
   ```

3. **Servisleri Başlatma**:
   ```bash
   docker-compose up -d
   ```

## Sorun Giderme

1. **Traefik Dashboard Erişimi**:
   ```bash
   docker-compose logs traefik
   ```

2. **Portainer Erişimi**:
   ```bash
   docker-compose logs portainer
   ```

3. **Ağ Bağlantısı**:
   ```bash
   docker network inspect proxy
   ```

## En İyi Uygulamalar

1. Traefik yapılandırma dosyalarını düzenli yedekleyin
2. SSL sertifikalarını güvenli bir şekilde saklayın
3. Portainer verilerini düzenli yedekleyin
4. Log dosyalarını düzenli kontrol edin
5. Güvenlik güncellemelerini takip edin

## Sık Kullanılan Parametreler

1. **build**: Dockerfile'dan imaj oluşturma
2. **restart**: Yeniden başlatma politikası
3. **networks**: Özel ağ yapılandırması
4. **env_file**: Ortam değişkenleri dosyası
5. **healthcheck**: Sağlık kontrolü ayarları

## Örnek Kullanım Senaryoları

1. **Geliştirme Ortamı Kurulumu**:
   ```bash
   docker-compose -f docker-compose.dev.yml up -d
   ```

2. **Üretim Ortamı Kurulumu**:
   ```bash
   docker-compose -f docker-compose.prod.yml up -d
   ```

3. **Belirli Servisleri Çalıştırma**:
   ```bash
   docker-compose up -d traefik
   ```

4. **Servisleri Yeniden Başlatma**:
   ```bash
   docker-compose restart traefik
   ```

5. **Servisleri Güncelleme**:
   ```bash
   docker-compose up -d --build
   ```

## Sorun Çözme

1. **Port Çakışması**:
   ```bash
   netstat -tuln | grep 80
   ```

2. **Disk Kullanımı Kontrolü**:
   ```bash
   docker system df
   ```

3. **Log Kontrolü**:
   ```bash
   docker-compose logs -f
   ```

4. **Ağ Sorunları**:
   ```bash
   docker network inspect proxy
   ```

5. **Volume Kontrolü**:
   ```bash
   docker volume ls
   ```

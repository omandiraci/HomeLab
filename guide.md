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

## Docker Compose Yapılandırma Örneği

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Yapılandırma Açıklamaları

1. **version**: Docker Compose sürümü
2. **services**: Çalıştırılacak servisler
3. **image**: Kullanılacak Docker imajı
4. **ports**: Port yönlendirme (host:container)
5. **volumes**: Kalıcı veri depolama
6. **depends_on**: Bağımlılık yönetimi
7. **environment**: Ortam değişkenleri

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
   docker-compose up -d web
   ```

4. **Servisleri Yeniden Başlatma**:
   ```bash
   docker-compose restart web
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
   docker network inspect <network_name>
   ```

5. **Volume Kontrolü**:
   ```bash
   docker volume ls
   ```

## En İyi Uygulamalar

1. Her servis için ayrı volume kullanın
2. Ortam değişkenlerini .env dosyasında tutun
3. Dockerfile'ları optimize edin
4. Log yönetimi için log driver kullanın
5. Kaynak sınırlamaları ekleyin (CPU, memory)

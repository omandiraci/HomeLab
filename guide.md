# HomeLab Kullanım Kılavuzu

Bu rehber, Docker Compose yapılandırma dosyasını anlamanıza yardımcı olacaktır.

## Kullanılan Servisler

### Traefik
- **Açıklama**: Traefik, bir ters proxy ve yük dengeleyicidir.
- **Portlar**: 80, 443, 8080
- **Kullanım**: `docker-compose up -d` komutu ile başlatılır.

### Portainer
- **Açıklama**: Portainer, Docker konteynerlerini yönetmek için bir arayüzdür.
- **Port**: 9000
- **Kullanım**: `docker-compose up -d` komutu ile başlatılır.

## Komutlar

1. Tüm servisleri başlat:
   ```bash
   git clone https://github.com/omandiraci/HomeLab.git
   cd HomeLab
   docker-compose up -d
   ```

2. Servisleri durdur:
   ```bash
   docker-compose down
   ```

3. Logları görüntüle:
   ```bash
   docker-compose logs
   ```

## Sorun Giderme

- **Traefik çalışmıyor**: `docker logs traefik` komutu ile logları kontrol edin.
- **Portainer erişilemiyor**: `docker logs portainer` komutu ile logları kontrol edin.

## Önemli Notlar

- Docker ve Docker Compose'un en son sürümlerini kullanın.
- Port çakışmalarına dikkat edin.

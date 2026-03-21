# HifzNet Routing

Кастомные geo-файлы для VPN-сервиса HifzNet. Автоматически собираются через GitHub Actions каждый понедельник.

## Как это работает

Три слоя трафика:

| Слой | Правило | Результат |
|------|---------|-----------|
| 🇷🇺 Direct | `geosite:hifznet` + `geoip:ru` + `geoip:hifznet-direct` | Российские сайты идут напрямую — банки и госсервисы не видят VPN |
| 🚫 Block | `geosite:hifznet-block` | Экстремизм и торренты не открываются вообще |
| 🌍 Proxy | всё остальное | YouTube, Telegram, GitHub — через VPN-сервер |

## Файлы для скачивания

| Файл | Назначение |
|------|-----------|
| [geosite.dat](https://github.com/IlkoMila/hifznet-routing/releases/latest/download/geosite.dat) | Списки доменов |
| [geoip.dat](https://github.com/IlkoMila/hifznet-routing/releases/latest/download/geoip.dat) | IP-диапазоны |

## Категории geosite.dat

### `geosite:hifznet` — прямое подключение
Все домены .ru/.su/.рф автоматически покрываются через catch-all правила.
Дополнительно прописаны зарубежные домены российских сервисов:
- **VK/Mail.ru**: userapi.com, vk-cdn.net, vkuservideo.net и др.
- **Яндекс**: yastatic.net, yandex.cloud, turbopages.org и др.
- **Сбер, Ozon, Wildberries**: зарубежная CDN-инфраструктура
- **Банки**: tinkoff.ru, alfabank.ru, vtb.ru и др.
- **Операторы**: mts.ru, beeline.ru, megafon.ru и др.

### `geosite:hifznet-block` — строгая блокировка
Блокируется у всех пользователей без исключений:
- Терроризм и экстремизм (ISIS, Аль-Каида, неонацизм и др.)
- Организации, запрещённые в РФ
- Торрент-трекеры (Rutracker, The Pirate Bay и др.)

### `geosite:hifznet-ads-block` — опциональная блокировка рекламы
Подключать по желанию. Может ломать deeplink-редиректы в мобильных приложениях:
- Крупные рекламные сети (DoubleClick, AdSense, AppNexus и др.)
- Трекеры и аналитика (AppsFlyer, Adjust, Mixpanel, Branch и др.)
- Российские рекламные сети

## Категории geoip.dat

| Категория | Содержимое |
|-----------|-----------|
| `geoip:ru` | Стандартные российские IP-диапазоны |
| `geoip:private` | Локальные сети (192.168.x.x, 10.x.x.x и др.) |
| `geoip:hifznet-direct` | Зарубежные IP российских сервисов по ASN |

### Как работает geoip:hifznet-direct

Яндекс и VK имеют дата-центры и CDN-узлы за пределами России (Амстердам, Франкфурт). Стандартный `geoip:ru` их не покрывает — трафик уходит через VPN и сервисы видят нидерландский IP.

Решение: каждую неделю автоматически запрашиваем RIPE API для ASN российских сервисов и добавляем все анонсированные IP-префиксы в категорию `geoip:hifznet-direct`.

ASN покрыты: Яндекс (AS13238, AS200350, AS202611), VK/Mail.ru (AS47764, AS47541, AS47542), Сбер (AS35237), Ozon (AS44386), Wildberries (AS57073).

## Настройка routing в клиенте

```json
{
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      { "type": "field", "domain": ["geosite:hifznet-block"], "outboundTag": "block" },
      { "type": "field", "domain": ["geosite:hifznet"], "outboundTag": "direct" },
      { "type": "field", "ip": ["geoip:ru", "geoip:private", "geoip:hifznet-direct"], "outboundTag": "direct" },
      { "type": "field", "port": "0-65535", "outboundTag": "proxy" }
    ]
  }
}
```

## Обновление и поддержка

- **Добавить домен в direct**: отредактировать `data/hifznet`, сборка запустится автоматически
- **Добавить домен в блок**: отредактировать `data/hifznet-block` (строгий) или `data/hifznet-ads-block` (реклама)
- **Добавить ASN**: отредактировать `asn/russian-services.txt`
- **Принудительная пересборка**: Actions → Run workflow

# NFQWS-MENU (OpenWrt / Keenetic)

Интерактивное меню установки и управления пакетами **nfqws-keenetic**, **nfqws2-keenetic** и **nfqws-keenetic-web** для роутеров на **OpenWrt** (нативно, opkg и apk) и **Keenetic / Netcraze** с **Entware**.

Это форк [**rndnaame/nfqws-menu**](https://github.com/rndnaame/nfqws-menu), адаптированный под чистый OpenWrt: автоматическое определение ОС, нативные фиды пакетов nfqws, procd-сервисы, DNS/hosts через UCI и dnsmasq, поддержка apk (OpenWrt 25.x). Поведение на Keenetic/Entware сохранено без изменений.

Репозиторий также служит хранилищем готовых **стратегий** обхода DPI, **blobs** и **lists**.

- Скрипт: [`nfqws-menu.sh`](nfqws-menu.sh) (текущая версия **0.1.0**)
- Стратегии: [`strategies/`](strategies/)
- Hosts: [`hosts`](hosts)
- Контрольные суммы: [`SHA256SUMS`](SHA256SUMS), [`strategies/blobs/SHA256SUMS`](strategies/blobs/SHA256SUMS)

### Официальные проекты

| Пакет | Репозиторий |
| --- | --- |
| nfqws-keenetic (v1) | https://github.com/nfqws/nfqws-keenetic |
| nfqws2-keenetic (v2) | https://github.com/nfqws/nfqws2-keenetic |
| Веб-интерфейс | https://github.com/nfqws/nfqws-keenetic-web |

### Источник стратегий

Стратегии в этом репозитории сделаны на базе проекта\
[**Flowseal/zapret-discord-youtube**](https://github.com/Flowseal/zapret-discord-youtube)\
и адаптированы под формат конфигов `nfqws-keenetic` / `nfqws2-keenetic`.

Подготовлены пользователем [**@Nare51**](https://github.com/Nare51) с использованием искусственного интеллекта.

---

## Быстрый старт

### OpenWrt

SSH на роутер (логин `root`), затем:

```bash
opkg update && opkg install curl && curl -sSL https://raw.githubusercontent.com/Dea1hwa1cher/nfqws-menu-openwrt/main/nfqws-menu.sh -o /usr/bin/nfqws-menu.sh && sh /usr/bin/nfqws-menu.sh
```

или с wget:

```bash
wget -O /usr/bin/nfqws-menu.sh https://raw.githubusercontent.com/Dea1hwa1cher/nfqws-menu-openwrt/main/nfqws-menu.sh && sh /usr/bin/nfqws-menu.sh
```

На OpenWrt 25.x+ с `apk` команда та же — скрипт сам определит пакетный менеджер. После первого запуска создаётся symlink для быстрого старта:

```bash
menu        # /usr/bin/menu → /usr/bin/nfqws-menu.sh
```

### Keenetic / Netcraze (Entware)

Подключитесь к Entware (SSH, порт 222 или 22, логин `root`):

```bash
opkg update && opkg install curl libnghttp2 && curl -sSL https://raw.githubusercontent.com/Dea1hwa1cher/nfqws-menu-openwrt/main/nfqws-menu.sh -o /opt/nfqws-menu.sh && sh /opt/nfqws-menu.sh
```

После первого запуска создаётся symlink `/opt/bin/menu` → `/opt/nfqws-menu.sh`:

```bash
menu
```

---

## Поддержка OpenWrt

Скрипт определяет ОС автоматически (OpenWrt / ImmortalWrt / Keenetic / Netcraze / Padavan) и подстраивает пути, пакетные фиды и сервисы:

| | Keenetic / Netcraze (Entware) | OpenWrt |
| --- | --- | --- |
| Конфиги | `/opt/etc/nfqws*/` | `/etc/nfqws*/` |
| Сервис | `/opt/etc/init.d/S51nfqws*` | `/etc/init.d/nfqws-keenetic`, `/etc/init.d/nfqws2-keenetic` (procd) |
| Пакетный менеджер | `opkg` | `opkg` (OpenWrt ≤24.10) или `apk` (25.x+) — определяется автоматически |
| Фид пакетов | `https://nfqws.github.io/nfqws-keenetic/$ARCH` | `https://nfqws.github.io/nfqws-keenetic/openwrt` (+ GPG-ключ: `.pub` через `opkg-key add` / `.pem` в `/etc/apk/keys`) |
| Symlink меню | `/opt/bin/menu` | `/usr/bin/menu` |
| Веб-интерфейс | `nfqws-keenetic-web` из фида `.../all` | нативно из фида `https://nfqws.github.io/nfqws-keenetic-web/openwrt` |
| Пункт 7 (hosts) | `ndmc ip host` | блок в `/etc/hosts` + reload dnsmasq |
| Пункт 8 (DoT/DoH) | `ndmc dns-proxy` | `https-dns-proxy` + dnsmasq per-domain (DoH) |
| Пункт 12 (KeenKit) | доступен | только Keenetic (нужен `ndmc`) |

Стратегии при применении автоматически переписывают пути blob/list-файлов под префикс пакета (`/opt/etc/nfqws*` → `/etc/nfqws*`).

Сторонние утилиты на OpenWrt:

| Утилита | OpenWrt |
| --- | --- |
| 10. dpi-detector | ставится нативно (бинарь в `/usr/local/bin`) |
| 14. MagiTrickle | нативный пакет (opkg/apk-фид, сервис `/etc/init.d/magitrickle`) |
| 16. tg-ws-proxy-rs | нативная ветка установщика (LuCI, `/usr/bin`, `/etc/init.d/tg-ws-proxy-rs`) |
| 11. awg-manager, 13. usque, 15. telemt, o. opera-proxy | только Entware — в меню выводится предупреждение |

Проверено на реальном устройстве: Xiaomi Mi Router R3, OpenWrt 24.10.1 (mipsel_24kc, fw4).

---

## Что делает меню

При запуске скрипт:

1. Определяет ОС (`/etc/openwrt_release` → `os-release` → `uci` → `ndmc`/Entware → Padavan → RCI) и архитектуру: **`opkg print-architecture`** / **`apk --print-arch`** → при ошибке **opkg.conf** → fallback `uname -m`. На части Keenetic `uname`/RCI отдают `mips` вместо `mipsel` — поэтому opkg приоритетнее.
2. Заголовок: **Модель / Title / ARCH** — на Keenetic из RCI (`/rci/show/version`), на OpenWrt из `/tmp/sysinfo/model` и `/etc/openwrt_release`. ARCH зелёный (из opkg/openwrt_release) или жёлтый (из conf/uname). Предупреждение, если нет модулей NFQUEUE: на OpenWrt — `kmod-nft-queue` (fw4) / `kmod-ipt-nfqueue` (fw3), на Keenetic — компонент «Модули ядра подсистемы Netfilter».
3. Показывает **только установленные** компоненты (версии и статус) одним проходом awk по status/process-кэшу:
   - пакеты NFQWS / web, usque-keenetic, **tg-ws-proxy-rs**, magitrickle;
   - dpi-detector, awg-manager (⚡ при sing-box / amneziawg), KeenKit, telemt, opera-proxy, cron, dropbear;
   - **⚡** — сервис запущен; **⭡** — доступна более новая версия (фоновая проверка, **TTL 10 мин**, перерисовка меню без Enter).
4. Интерфейс **RU / EN** (пункт **77**): подписи меню и общие промпты (`Your choice [0]:` / `Ваш выбор [0]:`). Подпись `Language: RU|EN` всегда латиницей (удобно для Telnet).
5. Предлагает меню:

```
[::]  КОМПОНЕНТЫ
      1.  Установка NFQWS / NFQWS2

[::]  СТРАТЕГИИ / СПИСКИ
      2.  Выбор стратегии
      3.  Обновить IPSet List
      4.  Загрузить rkn.list (125k+ доменов)
      5.  Обход блокировки DoT/DoH
      6.  Смена активных fake:blob
      7.  Обновление hosts
      8.  Управление DoT/DoH

[::]  УТИЛИТЫ
      10. dpi-detector
      11. awg-manager
      12. KeenKit
      13. usque-keenetic
      14. MagiTrickle
      15. telemt / telemt-panel
      16. TG WS Proxy Rust

[::]  СЕРВИС [S]  |  Language: RU|EN [77]
      88. Удаление пакетов
      99. Обновить скрипт
      00. Выход
```

Горячие клавиши: **S** — сервисные утилиты (UPX, Dropbear fix, upgrade пакетов); **U** — обновление всех пакетов; **77** — смена языка RU↔EN.

Пример блока статуса:

```
Установленные компоненты:
  nfqws2-keenetic        1.3.1 ⚡
  nfqws-keenetic-web     3.0.24 ⚡
  tg-ws-proxy-rs         2.x ⚡
  dpi-detector           …
  ⭡ - доступна более новая версия
```

### 1. Установка NFQWS / NFQWS2

Подменю:

```
1) nfqws-keenetic
2) nfqws2-keenetic
3) nfqws-keenetic-web
4) Установка / обновление .ipk (обход DPI)
0) Назад
```

- **1–2** — зависимости (`ca-certificates`, `wget-ssl`), официальный репозиторий и установка пакета:
  - **OpenWrt** — нативный фид `https://nfqws.github.io/<repo>/openwrt` с GPG-ключом (opkg: `opkg-key add` .pub; apk: .pem в `/etc/apk/keys`);
  - **Entware** — фид `https://nfqws.github.io/<repo>/$ARCH`.
  При v2, если стоит v1 — предложение удалить старый пакет. После установки — предложение поставить веб.
- **3** — веб-интерфейс (lighttpd + PHP, порт **90**): `http://<IP-роутера>:90`. На OpenWrt ставится из нативного фида; логин/пароль — учётные данные root роутера (отключается в `/etc/nfqws_web.conf`: `enabled = false`). На Entware — фид `.../all`, логин/пароль Entware (`root` / `keenetic` по умолчанию).
- **4** — прямая установка **.ipk** (обход DPI; только Entware): версии с github.io, скачивание с GitHub Releases → ghproxy → github.io, затем `opkg install` локального файла (конфиги при необходимости извлекаются из .ipk).

> Фикс TLS reasm с **NFQWS2 ≥ 1.3.1** входит в официальный пакет.

### 2. Выбор стратегии

- Если ни одна версия NFQWS не установлена — предлагает установить.
- Показывает список `.conf` из:
  - `strategies/nfqws1/` — для v1
  - `strategies/nfqws2/` — для v2
- Список стратегий: **`SHA256SUMS`** → GitHub API → **локальный offline-кэш** (`${CONF_BASE_DIR}/nfqws-menu/strategies/…`, на OpenWrt `/etc/nfqws-menu/…`).
- Скачивание: основной raw GitHub → зеркала **jsDelivr / Fastly jsDelivr / ghproxy** → fallback через туннель-интерфейсы (`awg0`, `t2s0`, …); успешный файл сохраняется в offline-кэш.
- Делает **бэкап** текущего конфига и применяет стратегию.
- Перед записью **нормализует CRLF → LF** (иначе `source`/iptables: `: not found`, `invalid port/service`).
- На OpenWrt пути blob/list-файлов в стратегии автоматически переписываются под `/etc/nfqws*`.
- **Пункт 99** — восстановление из backup: `.bak.*` от свежего к старому (самый свежий выделен), рядом имя стратегии из `general (…)` в файле; `fix_isp_interface`, перезапуск сервиса.
- После применения выполняет пост-настройку:

#### ISP_INTERFACE / IPV6_ENABLED

- Определяет интерфейс провайдера из default route (`ip route` / `route`), сравнивает с `ISP_INTERFACE` в конфиге и при необходимости предлагает установить правильное значение. При чтении значения снимаются `\r`, кавычки и пробелы (защита от CRLF в выводе меню).
- Проверяет наличие **глобального IPv6** (`scope global`: 2a00, 2a02, 2606, 2001, …) на выбранном `ISP_INTERFACE` (или default route) и выставляет `IPV6_ENABLED=0|1`. Несколько iface через пробел — достаточно одного с global.

#### Восстановление rkn.list и POLICY_*

- Если до смены стратегии в `MODE_LIST` уже был `--hostlist=…/rkn.list`, привязка **восстанавливается** автоматически.
- **`POLICY_NAME` / `POLICY_EXCLUDE`** (Keenetic): стандартные значения (`nfqws` / `0`) не трогаются; **нестандартные** предлагается восстановить (**по умолчанию: Нет**).

#### Проверка blobs (наличие + SHA256)

Парсит установленный конфиг и находит используемые `.bin` (`--blob=…`, `--dpi-desync-fake-tls=…`, `--dpi-desync-fake-quic=…` и абсолютные пути).

- Сверяет локальные файлы с эталоном [`strategies/blobs/SHA256SUMS`](strategies/blobs/SHA256SUMS) (`sha256sum` или `openssl dgst -sha256`; кэш SUMS ~1 ч).
- Отсутствующие или **устаревшие/повреждённые** (хеш не совпал) предлагает скачать заново; после загрузки — повторная проверка SHA256 (при несовпадении файл удаляется).

#### Обновление lists

По запросу обновляет из `strategies/lists/`: `user.list`, `exclude.list`, `ipset.list`, `ipset_exclude.list`.  
`auto.list` **не трогается** — его заполняет демон.

После всех шагов соответствующий сервис перезапускается.

### 3. Обновление IPSet List

Скачивает актуальный IP/CIDR-список из\
[Flowseal/zapret-discord-youtube](https://github.com/Flowseal/zapret-discord-youtube)\
и записывает в `ipset.list`:

| Версия | Entware | OpenWrt |
| --- | --- | --- |
| nfqws-keenetic (v1) | `/opt/etc/nfqws/ipset.list` | `/etc/nfqws/ipset.list` |
| nfqws2-keenetic (v2) | `/opt/etc/nfqws2/lists/ipset.list` | `/etc/nfqws2/lists/ipset.list` |

Бэкап, очистка пустых строк/комментариев, перезапуск сервиса. При двух установленных версиях — можно обновить обе.

### 4. Загрузить rkn.list (125k+ доменов)

Доступно при установленном **nfqws-keenetic** (v1) и/или **nfqws2-keenetic** (v2). При обеих версиях — выбор: 1 / 2 / обе.

- Если `rkn.list` уже есть — показывает размер в **КБ** (без медленного подсчёта 125k строк) и спрашивает, обновлять ли список (**по умолчанию: Нет**). При отказе скачивание пропускается.
- Скачивает большой список доменов РКН из [IndeecFOX/zapret4rocket](https://github.com/IndeecFOX/zapret4rocket)\
  (`extra_strats/TCP/RKN/List.txt`). Сначала **GitHub raw**, затем jsDelivr / Fastly / ghproxy; при недоступности — зеркало `mizulina.shit.vc`.
- Записывает: v1 → `${CONF_BASE_DIR}/nfqws/rkn.list`, v2 → `${CONF_BASE_DIR}/nfqws2/lists/rkn.list` (Entware: `/opt/…`, OpenWrt: `/etc/…`).
- Добавляет `--hostlist=…/rkn.list` в `MODE_LIST` соответствующего конфига (с бэкапом), если его ещё нет. Вставка через **awk** (устойчиво к CRLF, пробелам, пустым кавычкам).
- **Перезапуск** сервиса только при реальных изменениях (обновлён список и/или изменён `MODE_LIST`). Если hostlist уже был в конфиге и список не обновляли — перезапуск не выполняется.

### 5. Обход блокировки DoT/DoH

Доступно при установленном **nfqws-keenetic** и/или **nfqws2-keenetic** (при обеих версиях — выбор v1 / v2 / обе).

Добавляет в `NFQWS_ARGS_CUSTOM` стратегию обхода блокировок публичных DoT/DoH DNS:

- **nfqws1** — [`strategies/dns_filter_nfqws1`](strategies/dns_filter_nfqws1) (классический `--dpi-desync`)
- **nfqws2** — [`strategies/dns_filter_nfqws2`](strategies/dns_filter_nfqws2) (lua-desync)

Покрывает Cloudflare, Google, AdGuard, NextDNS, Quad9, DNS.SB, Alibaba, Xbox-DNS, Geohide и др.; при необходимости добавляет порт `853` в `TCP_PORTS` / `UDP_PORTS` и перезапускает соответствующий сервис. Стратегия сначала скачивается с GitHub, при недоступности сети — встроенный fallback (пути blob-файлов подставляются под платформу).

> На OpenWrt работает полностью; Keenetic-привязки `dns-proxy` (пресеты п. 8.3) пропускаются — там нужен `ndmc`.

### 6. Смена активных fake:blob

Позволяет заменить используемые в конфиге `fake:blob=NAME` на другой `.bin`-файл (переназначает путь в `--blob=NAME:…`).

- Быстрый разбор конфига (v1/v2) одним проходом **awk**: секции `NFQWS_*ARGS*`, map `--blob=name:path`, уникальные `fake:blob=` (hex пропускаются).
- Показывает список найденных имён по секциям.
- Кандидаты `.bin`: локальные → имена из **`strategies/blobs/SHA256SUMS`** → GitHub API → встроенный fallback.
- При выборе файла из репозитория — скачивает его, обновляет `--blob=…`, бэкап конфига.
- Перезапуск сервиса — по подтверждению (по умолчанию Да).

### 7. Обновление hosts

Статические DNS-привязки, платформенно-зависимый бэкенд:

- **Keenetic / Netcraze** — `ndmc` (`ip host DOMAIN IP` / `no ip host DOMAIN` + сохранение конфигурации роутера).
- **OpenWrt** — управляемый блок с маркерами `# >>> nfqws-menu hosts >>>` в `/etc/hosts`; dnsmasq подхватывает его автоматически, после изменений сервис перечитывается. Удаление — чистка блока, чужие записи в `/etc/hosts` не затрагиваются.

Общее:

- Скачивает файл `hosts` из репозитория.
- Секции задаются комментариями `# Имя секции`; внутри — строки `IP DOMAIN`.
- Можно выбрать одну или несколько секций, **все**, либо **88** — удалить записи, **99** — просмотреть текущие.
- Только **IPv4**; битые домены (`..`, ведущая `.`) пропускаются; **один IP на домен**.
- Предупреждение, если уникальных записей **> 64** (лимит Keenetic `ip host`).

### 8. Управление DoT/DoH

Управление DNS-over-TLS / DNS-over-HTTPS:

- **Keenetic/Netcraze** — через `ndmc` (`dns-proxy tls/https upstream ...`), сохранение конфига роутера.
- **OpenWrt** — через пакет `https-dns-proxy` (DoH): серверы как uci-секции, привязка домена — `server=/домен/127.0.0.1#порт` в `dhcp.@dnsmasq[0].server`. Режим выборочных доменов: остальной DNS идёт как раньше. DoT этим меню на OpenWrt не управляется.

Подменю (Keenetic):

```
1) Добавить DoT сервер(ы)
2) Добавить DoH сервер(ы)
3) Привязать домен к DNS (Пресеты)
4) Удалить сервер(ы)
0) Назад
```

Подменю (OpenWrt):

```
1) Добавить DoH сервер (https-dns-proxy)
2) Привязать домен к DoH серверу
3) Удалить привязку домена
4) Удалить DoH сервер
0) Назад
```

Возможности:

- Просмотр текущих DoT/DoH и персональных привязок к доменам
- Счётчик слотов **N/8** (Keenetic) — только секция **System** (Policy0/1 и `*-filters` не учитываются)
- Пресеты DoT/DoH: Яндекс, Cloudflare, Google, Quad9, CleanBrowsing, OpenDNS, DNS.SB, OpenNameServer, AdGuard (Default/Family), ControlD Free, DNS4EU, Alibaba DNS, DNSPod, **NullsProxy**, Xbox-DNS, Comss, Malw Link, Geo Hide, Cloudflare Gateway, **AstraCat**, **Bezmezhau**, **DNS-AI**, **Mafioznik**, **Shecan Free** и др. или ручной ввод
- Быстрая привязка доменов (Keenetic, пресеты): CleanBrowsing → instagram.com / cdninstagram.com, sw.ext.io → rutor, Malw Link → ntc.party, Xbox-DNS → Twitch, NullsProxy → Supercell, либо свой домен
- Удаление upstream-ов с сохранением конфигурации

> Keenetic-часть требует `ndmc`.\
> При активном **Интернет-фильтре** часть DoT может помечаться как *disregarded*.

### 10. dpi-detector

Инструмент для анализа цензуры трафика в России: обнаруживает и классифицирует блокировки сайтов, хостингов и CDN (TCP16–20 блокировки), а также подмену DNS-запросов провайдером.

Актуальная версия [dpi-detector](https://github.com/Runnin4ik/dpi-detector) (ветка `rust`). Установщик сам выбирает каталог: `/opt/bin` (Entware) или `/usr/local/bin` (OpenWrt — персистентный overlay).

Если бинарник уже есть — очищает дубликаты (`/tmp`, `/opt/root`) и сразу запускает его. После установки — та же очистка дубликатов.

### 11. awg-manager

Веб-интерфейс для управления AmneziaWG VPN-туннелями на роутерах Keenetic. В тестовом режиме добавлена поддержка Sing-box (vless tcp, hysteria, trojan и др.).

Установщик [awg-compressed](https://github.com/rndnaame/awg-compressed). **Только Entware** — на OpenWrt пункт выводит предупреждение.

В статусе: `awg-manager` или `awg-manager [+SB]` при наличии sing-box.

### 12. KeenKit

Многофункциональный скрипт, упрощающий взаимодействие с роутером на базе KeeneticOS. **Только Keenetic / Netcraze**.

- Есть `/opt/keenkit.sh` — **сразу запускает**.
- Иначе — установщик [KeenKit](https://github.com/spatiumstas/KeenKit).

### 13. usque-keenetic

Адаптация неофициального клиента Cloudflare WARP с режимом MASQUE для роутеров Keenetic / Netcraze. **Только Entware**.

Через [`installer_usque.sh`](installer_usque.sh) (пункт меню 13 или напрямую):

```bash
curl -fsSL https://raw.githubusercontent.com/Dea1hwa1cher/nfqws-menu-openwrt/main/installer_usque.sh | sh
```

Подменю:

1. **Установка** — пакет [usque-keenetic](https://github.com/side-effect-tm/usque-keenetic) (opkg, репозиторий `side-effect-tm.github.io/usque-keenetic/all`)
2. **Обновление bin** — бинарник [Diniboy1123/usque](https://github.com/Diniboy1123/usque) с GitHub Releases → `/opt/usr/bin/usque`
3. **Удаление** — `opkg remove usque-keenetic` + conf репозитория; опционально `/opt/etc/usque`

- Init: `/opt/etc/init.d/S51usque` (start | stop | restart | status)
- Конфиг: `/opt/etc/usque/usque.conf`
- Бинарник: `/opt/usr/bin/usque`

### 14. MagiTrickle

Утилита для точечной маршрутизации сетевого трафика по заданным доменным именам. Нативно работает и на OpenWrt (её `add_repo.sh` сам поддерживает opkg и apk), и на Entware.

Установка / обновление [MagiTrickle](http://bin.magitrickle.dev/):

- **Не установлен** — добавление репозитория (`add_repo.sh`) + установка пакета + запуск сервиса
- **Установлен** → обновление пакета + restart сервиса
- Init: OpenWrt — `/etc/init.d/magitrickle` (procd), Entware — `/opt/etc/init.d/S99magitrickle`
- Удаление — в п. 88 (с опциональным удалением файла репозитория)

### 15. telemt / telemt-panel

Telegram MTProto-прокси на Rust (полная реализация официального алгоритма + расширения). **Только Entware** (рекомендуется **aarch64**).

Подменю:

```
1. Установка telemt
2. Установка telemt-panel
3. Эмуляция systemD
4. Удаление
0. Назад
```

- Установщики запускаются с привязкой к `/dev/tty` (интерактивные скрипты).
- П. 3 — эмуляция systemctl для Entware ([anch665/keendev](https://github.com/anch665/keendev) и аналоги), если нужна совместимость с unit-скриптами.
- Статус telemt / telemt-panel отображается в блоке установленных компонентов; удаление — здесь или в п. 88.

### S. Сервисные утилиты

В главном меню: клавиша **S** (или раздел «СЕРВИС»).

| Пункт | Действие |
| --- | --- |
| 1 | **Сжать bin/sbin (UPX)** — `upx --lzma --best` для `/opt/bin`, `/opt/sbin`, `/opt/usr/bin`, `/opt/libexec` (при необходимости ставит `upx`). Entware |
| 2 | **Dropbear fix** — правка конфига/init Entware dropbear (с опцией сброса пароля root или без). Entware |
| U | **Обновить все пакеты** — `opkg update && opkg upgrade` / `apk update && apk upgrade` (также клавиша **U** из главного меню) |

### 16. TG WS Proxy Rust (tg-ws-proxy-rs)

[tg-ws-proxy-rs](https://github.com/valnesfjord/tg-ws-proxy-rs) — MTProto-прокси на Rust. В пакетных репозиториях его нет: ставит штатный `install.sh` проекта, который умеет обе платформы — **OpenWrt 24.10+ (с LuCI-интеграцией)** и Entware. Меню добавляет:

- **секрет и порт прежней Go-установки** (если осталась) — ссылки `tg://` не меняются, Go-сервис останавливается и снимается с автозапуска;
- **выбор сборки** — обычная или компактная (UPX), таблицей: размер на флеше и потребление памяти. По умолчанию обычная; компактная — когда флеш-памяти мало (образ распаковывается в ОЗУ и не вытесняется);
- **подбор параметров по замерам** — CF-домены и варианты лестницы проверяются штатной пробой бинаря, затем выводится список с временем каждого, и путь выбираете вы (Enter — быстрейший);
- **проверка связи до выдачи ссылки** — `resPQ` от DC Telegram, то есть вся цепочка, а не «порт открыт».

Пути: OpenWrt — `/usr/bin/tg-ws-proxy-rs`, `/etc/init.d/tg-ws-proxy-rs`, конфиги `/etc/tg-ws-proxy-rs/`; Entware — `/opt/bin/…`, `/opt/etc/init.d/S99tg-ws-proxy-rs`, конфиги `/opt/etc/tg-ws-proxy-rs/`.

Проба есть с версии **2.4.5**; у более старого бинаря меню предложит обновиться.

Секрет меню пишет само, до установки: генератор секрета в `install.sh` проекта использует `od -An`, которого BusyBox не знает, и на боксе без готового секрета установка падает. Если секрет остался от Go-версии, переносится он — тогда ссылки клиентов не меняются.

### 77. Change language

Мгновенное переключение интерфейса **ru ↔ en** (файл `${CONF_BASE_DIR}/nfqws-menu/nfqws-menu.lang`).

### 88. Удаление пакетов

Показывает установленные компоненты и позволяет удалить выборочно:

```
Удаление:
  [N] nfqws2-keenetic / nfqws-keenetic-web / dpi-detector /
      awg-manager / tg-ws-proxy-rs / usque-keenetic /
      magitrickle / opera-proxy / KeenKit / telemt
  [a] Удалить все пакеты NFQWS
  [b] Удалить резервные копии (.bak.* / *-opkg)
  [0] Назад
```

- Пакеты NFQWS — `opkg remove` / `apk del` (по платформе)
- **dpi-detector** — бинарник (путь по платформе)
- **awg-manager** — `opkg remove` + `rm -rf /opt/etc/awg-manager` (Entware)
- **tg-ws-proxy-rs** — бинарь, init, конфиги (пути по платформе)
- **usque-keenetic** — `opkg remove` + conf репозитория (Entware)
- **magitrickle** — удаление пакета + запрос на удаление файла репозитория (путь по платформе)
- **KeenKit** — удаление `/opt/keenkit.sh`
- **a)** — все пакеты NFQWS + dpi-detector
- **b)** — `*.bak.*`, `*.conf-opkg`, `*.list-opkg` в каталогах конфигов nfqws (пути по платформе)

### 99. Обновить скрипт

Скачивает свежую версию **этого форка**, перезаписывает скрипт (`/usr/bin/nfqws-menu.sh` на OpenWrt, `/opt/nfqws-menu.sh` на Entware), обновляет symlink `menu`, сразу перезапускает меню (`exec`).

---

## Changelog

### 0.1.0 (форк Dea1hwa1cher/nfqws-menu-openwrt)

Первый релиз адаптации под OpenWrt. База — апстрим 0.9.41; вся история апстрима (0.4.0–0.9.41) — в [исходном репозитории](https://github.com/rndnaame/nfqws-menu#changelog).

- **Определение ОС** — OpenWrt / ImmortalWrt / Keenetic / Netcraze / Padavan / generic; переменные платформы (пути, статус opkg, фиды), модель/версия из `/tmp/sysinfo/model` и `/etc/openwrt_release`
- **Пакеты** — нативный фид `https://nfqws.github.io/<repo>/openwrt` с GPG-ключом; поддержка `apk` (OpenWrt 25.x: `packages.adb` в `/etc/apk/repositories.d`, `.pem` в `/etc/apk/keys`); обёртки `pkg_update/install/remove` над opkg/apk
- **Пути** — все конфиги/сервисы/списки через `CONF_BASE_DIR`/`INIT_BASE_DIR`; сервисы procd `nfqws-keenetic` / `nfqws2-keenetic`; sed-подмена `/opt/etc/nfqws*` → `/etc/nfqws*` при применении стратегий
- **П. 7 (hosts)** — на OpenWrt блок-маркеры в `/etc/hosts` + reload dnsmasq (на Keenetic — `ndmc ip host`)
- **П. 8 (DoT/DoH)** — на OpenWrt менеджер `https-dns-proxy` (uci) + привязки доменов через dnsmasq per-domain
- **П. 5 (DoT/DoH-стратегия)** — полностью работает на OpenWrt (ndmc-привязки пропускаются)
- **Web** — нативная установка `nfqws-keenetic-web` из фида `.../openwrt`
- **Утилиты** — dpi-detector (`/usr/local/bin`), MagiTrickle (нативный пакет), tg-ws-proxy-rs (OpenWrt/LuCI-ветка установщика); awg-manager / usque / telemt / opera-proxy — предупреждение «только Entware»
- **Keenetic-only функции** (KeenKit, ndmc-меню) скрыты/подсвечены предупреждением
- **Совместимость busybox ash** (проверено на устройстве OpenWrt 24.10): без переносов строк с операторами `||`/`&&`/`|` в начале продолжения; обновление скрипта (п. 99) и стратегии качаются из этого форка

---

## Структура репозитория

```
nfqws-menu-openwrt/
├── nfqws-menu.sh          # Главный скрипт меню
├── installer_usque.sh     # Установщик usque-keenetic (Entware)
├── installer_telemt.sh    # Установщик telemt / telemt-panel (Entware)
├── menu-dns.sh            # DNS-менеджер (Keenetic / ndmc)
├── hosts                  # Секции IP DOMAIN для п. 7
├── SHA256SUMS             # Хеши файлов репозитория (список стратегий без API)
├── README.md
├── .github/workflows/
│   └── sha256sums.yml     # Автообновление SHA256SUMS
└── strategies/
    ├── blobs/
    │   ├── *.bin          # Бинарные шаблоны
    │   └── SHA256SUMS     # Эталоны для проверки blobs на роутере
    ├── lists/             # Готовые списки доменов / IP
    │   ├── user.list
    │   ├── exclude.list
    │   ├── ipset.list
    │   └── ipset_exclude.list
    ├── nfqws1/            # Стратегии для nfqws-keenetic (v1)
    └── nfqws2/            # Стратегии для nfqws2-keenetic (v2)
```

### Как добавить свою стратегию

1. Файл `имя.conf` в `strategies/nfqws1/` или `strategies/nfqws2/`.
2. Полный конфиг (`ISP_INTERFACE=`, `NFQWS_ARGS=` / `NFQWS_BASE_ARGS=` и т.д.). Пути blob/list-файлов можно писать в Entware-форме (`/opt/etc/nfqws/…`) — на OpenWrt они будут переписаны автоматически.
3. При необходимости — `.bin` в `strategies/blobs/`, списки в `strategies/lists/`.
4. После push workflow обновит `SHA256SUMS` / `strategies/blobs/SHA256SUMS`; скрипт подхватит стратегию из SUMS (или через GitHub API).

---

## Требования

### OpenWrt

1. OpenWrt 23.05+ (opkg) или 25.x+ (apk); ImmortalWrt — тоже работает.
2. Модули NFQUEUE: `kmod-nft-queue` (fw4 / nftables, 22.03+) или `kmod-ipt-nfqueue` + `iptables-mod-nfqueue` (fw3). Меню предупреждает, если их нет.
3. Для веб-интерфейса ничего дополнительно ставить не нужно — пакет придёт с фидом.

### Keenetic / Netcraze

1. Установлен **Entware** (внутренняя память или USB).
2. В веб-интерфейсе — **модули ядра Netfilter** (`OPKG → Kernel modules for Netfilter`).\
   На старых прошивках компонент появляется после включения IPv6.
3. Рекомендуется отключить DNS провайдера и настроить DoT/DoH.
4. Команды выполняются **в среде Entware**, не в CLI Keenetic.

---

## Полезные команды вручную

OpenWrt:

```bash
# Статус сервисов
service nfqws-keenetic status          # v1
service nfqws2-keenetic status         # v2
/etc/init.d/magitrickle status         # MagiTrickle
/etc/init.d/tg-ws-proxy-rs status      # TG WS Proxy Rust

# Перезапуск
service nfqws-keenetic restart
service nfqws2-keenetic restart

# Информация о пакете
opkg info nfqws-keenetic               # или apk info nfqws-keenetic (25.x+)

# Конфиги
vi /etc/nfqws/nfqws.conf
vi /etc/nfqws2/nfqws2.conf

# Привязки доменов DNS (п. 8, https-dns-proxy)
uci show https-dns-proxy

# Быстрый запуск меню
menu
```

Keenetic / Netcraze (Entware):

```bash
# Статус сервисов
/opt/etc/init.d/S51nfqws status          # v1
/opt/etc/init.d/S51nfqws2 status         # v2
/opt/etc/init.d/S99tg-ws-proxy-rs status # TG WS Proxy Rust
/opt/etc/init.d/S51usque status          # usque
/opt/etc/init.d/S99magitrickle status    # MagiTrickle

# Порт веб-интерфейса
netstat -lnt | grep ':90'

# Информация о пакете
opkg info nfqws-keenetic
opkg info nfqws2-keenetic
opkg info nfqws-keenetic-web

# Конфиги
vi /opt/etc/nfqws/nfqws.conf
vi /opt/etc/nfqws2/nfqws2.conf
vi /opt/etc/tg-ws-proxy-rs/config.conf
vi /opt/etc/tg-ws-proxy-rs/secret.conf
vi /opt/etc/usque/usque.conf

# Интерфейс провайдера
ip route | grep ^default

# DNS-proxy (DoT/DoH)
ndmc -c show dns-proxy

# Быстрый запуск меню
menu
```

---

## Лицензия / отказ от ответственности

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Материалы подготовлены в ознакомительных и научно-технических целях.\
Использование на свой страх и риск. Автор не несёт ответственности за последствия.

Стратегии адаптированы на основе [Flowseal/zapret-discord-youtube](https://github.com/Flowseal/zapret-discord-youtube),\
подготовлены [@Nare51](https://github.com/Nare51) с использованием искусственного интеллекта.\
Официальные пакеты NFQWS: [nfqws](https://github.com/nfqws).\
База меню: [rndnaame/nfqws-menu](https://github.com/rndnaame/nfqws-menu); адаптация под OpenWrt: [@Dea1hwa1cher](https://github.com/Dea1hwa1cher).

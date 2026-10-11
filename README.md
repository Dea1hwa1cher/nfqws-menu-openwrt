# NFQWS-MENU (OpenWrt)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Интерактивное меню установки и управления пакетами **nfqws-keenetic**, **nfqws2-keenetic** и **nfqws-keenetic-web** для роутеров на **OpenWrt** (нативно, opkg и apk).

Это форк [**rndnaame/nfqws-menu**](https://github.com/rndnaame/nfqws-menu), адаптированный исключительно под чистый OpenWrt: автоматическое определение архитектуры, нативные фиды пакетов nfqws, procd-сервисы, DNS/hosts через UCI и dnsmasq, поддержка apk (OpenWrt 25.x).

Репозиторий также служит хранилищем готовых **стратегий** обхода DPI, **blobs** и **lists**.

- Скрипт: [`nfqws-menu.sh`](nfqws-menu.sh) (текущая версия **0.1.1**)
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

SSH на роутер (логин `root`).

> **Рекомендуется jsDelivr** — `raw.githubusercontent.com` у части провайдеров отвечает **404** (DPI), хотя файл в репозитории есть.

### Установка через jsDelivr (предпочтительно)

```bash
opkg update && opkg install curl wget-ssl
wget -O /usr/bin/nfqws-menu.sh https://cdn.jsdelivr.net/gh/Dea1hwa1cher/nfqws-menu-openwrt@main/nfqws-menu.sh && chmod +x /usr/bin/nfqws-menu.sh && sh /usr/bin/nfqws-menu.sh
```

или curl:

```bash
curl -fsSL -o /usr/bin/nfqws-menu.sh https://cdn.jsdelivr.net/gh/Dea1hwa1cher/nfqws-menu-openwrt@main/nfqws-menu.sh && chmod +x /usr/bin/nfqws-menu.sh && sh /usr/bin/nfqws-menu.sh
```

Запасное зеркало (Fastly):

```bash
wget -O /usr/bin/nfqws-menu.sh https://fastly.jsdelivr.net/gh/Dea1hwa1cher/nfqws-menu-openwrt@main/nfqws-menu.sh && chmod +x /usr/bin/nfqws-menu.sh && sh /usr/bin/nfqws-menu.sh
```

### Установка с GitHub raw (если не режется)

```bash
wget -O /usr/bin/nfqws-menu.sh https://raw.githubusercontent.com/Dea1hwa1cher/nfqws-menu-openwrt/main/nfqws-menu.sh && chmod +x /usr/bin/nfqws-menu.sh && sh /usr/bin/nfqws-menu.sh
```

или ghproxy:

```bash
wget -O /usr/bin/nfqws-menu.sh https://ghproxy.net/https://raw.githubusercontent.com/Dea1hwa1cher/nfqws-menu-openwrt/main/nfqws-menu.sh && chmod +x /usr/bin/nfqws-menu.sh && sh /usr/bin/nfqws-menu.sh
```

На OpenWrt 25.x+ с `apk` вместо `opkg install` используется `apk add curl wget-ssl` (или curl / wget из базы). После первого запуска создаётся symlink для быстрого старта:

```bash
menu        # /usr/bin/menu → /usr/bin/nfqws-menu.sh
```

---

## Особенности на OpenWrt

Скрипт определяет ОС автоматически (OpenWrt / ImmortalWrt) и настраивает пути, пакетные фиды и сервисы:

| Параметр | Реализация на OpenWrt |
| --- | --- |
| Конфиги | `/etc/nfqws/nfqws.conf`, `/etc/nfqws2/nfqws2.conf` |
| Сервисы | `/etc/init.d/nfqws-keenetic`, `/etc/init.d/nfqws2-keenetic` (procd) |
| Пакетный менеджер | `opkg` (OpenWrt ≤24.10) или `apk` (25.x+) — определяется автоматически |
| Фид пакетов | `https://nfqws.github.io/nfqws-keenetic/openwrt` (+ GPG-ключ: `.pub` через `opkg-key add` / `.pem` в `/etc/apk/keys`) |
| Symlink меню | `/usr/bin/menu` → `/usr/bin/nfqws-menu.sh` |
| Веб-интерфейс | Нативно из фида `https://nfqws.github.io/nfqws-keenetic-web/openwrt` |
| Hosts (п. 7) | Блок с маркерами `# >>> nfqws-menu hosts >>>` в `/etc/hosts` + reload dnsmasq |
| DoH (п. 8) | Пакет `https-dns-proxy` (uci) + dnsmasq per-domain (DoH) |

Стратегии в репозитории сконфигурированы под нативные пути OpenWrt (`/etc/nfqws*`, `/var/log*`).

Сторонние утилиты на OpenWrt:

| Утилита | OpenWrt |
| --- | --- |
| 10. dpi-detector | ставится нативно (бинарь в `/usr/local/bin`) |
| 14. MagiTrickle | нативный пакет (opkg/apk-фид, сервис `/etc/init.d/magitrickle`) |
| 16. tg-ws-proxy-rs | нативная ветка установщика (LuCI, `/usr/bin`, `/etc/init.d/tg-ws-proxy-rs`) |

Проверено на реальном устройстве: Xiaomi Mi Router R3, OpenWrt 24.10.1 (mipsel_24kc, fw4).

---

## Что делает меню

При запуске скрипт:

1. Определяет ОС (`/etc/openwrt_release` → `/etc/os-release` → `uci`) и архитектуру: **`opkg print-architecture`** / **`apk --print-arch`** → при ошибке **opkg.conf** → fallback `uname -m`.
2. Заголовок: **Модель / Title / ARCH** — из `/tmp/sysinfo/model` и `/etc/openwrt_release`. ARCH зелёный (из opkg/openwrt_release) или жёлтый (из conf/uname). Предупреждение, если нет модулей NFQUEUE: `kmod-nft-queue` (fw4) / `kmod-ipt-nfqueue` (fw3).
3. Показывает **только установленные** компоненты (версии и статус) одним проходом awk по status/process-кэшу:
   - пакеты NFQWS / web, **tg-ws-proxy-rs**, magitrickle;
   - dpi-detector, cron, dropbear;
   - **⚡** — сервис запущен; **⭡** — доступна более новая версия (фоновая проверка, **TTL 10 мин**, перерисовка меню без Enter).
4. Интерфейс **RU / EN** (пункт **77**): подписи меню и общие промпты (`Your choice [0]:` / `Ваш выбор [0]:`). Подпись `Language: RU|EN` всегда латиницей.
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
      14. MagiTrickle
      16. TG WS Proxy Rust

[::]  СЕРВИС [S]  |  Language: RU|EN [77]
      88. Удаление пакетов
      99. Обновить скрипт
      00. Выход
```

Горячие клавиши: **S** — сервисные утилиты (UPX, upgrade пакетов); **U** — обновление всех пакетов; **77** — смена языка RU↔EN.

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
0) Назад
```

- **1–2** — зависимости (`ca-certificates`, `wget-ssl`), официальный репозиторий и установка пакета: нативный фид `https://nfqws.github.io/<repo>/openwrt` с GPG-ключом (opkg: `opkg-key add` .pub; apk: .pem в `/etc/apk/keys`). При v2, если стоит v1 — предложение удалить старый пакет. После установки — предложение поставить веб.
- **3** — веб-интерфейс (lighttpd + PHP, порт **90**): `http://<IP-роутера>:90`. Ставится из нативного фида; логин/пароль — учётные данные root роутера (отключается в `/etc/nfqws_web.conf`: `enabled = false`).

> Фикс TLS reasm с **NFQWS2 ≥ 1.3.1** входит в официальный пакет.

### 2. Выбор стратегии

- Если ни одна версия NFQWS не установлена — предлагает установить.
- Показывает список `.conf` из:
  - `strategies/nfqws1/` — для v1
  - `strategies/nfqws2/` — для v2
- Список стратегий: **`SHA256SUMS`** → GitHub API → **локальный offline-кэш** (`/etc/nfqws-menu/strategies/…`).
- Скачивание: основной raw GitHub → зеркала **jsDelivr / Fastly jsDelivr / ghproxy** → fallback через туннель-интерфейсы (`awg0`, `t2s0`, …); успешный файл сохраняется в offline-кэш.
- Делает **бэкап** текущего конфига и применяет стратегию.
- Перед записью **нормализует CRLF → LF** (иначе `source`/iptables: `: not found`, `invalid port/service`).
- На OpenWrt пути blob/list-файлов в стратегии автоматически переписываются под `/etc/nfqws*`.
- **Пункт 99** — восстановление из backup: `.bak.*` от свежего к старому (самый свежий выделен), рядом имя стратегии из `general (…)`; `fix_isp_interface`, перезапуск сервиса.
- После применения выполняет пост-настройку:

#### ISP_INTERFACE / IPV6_ENABLED

- Определяет интерфейс провайдера из default route (`ip route` / `route`), сравнивает с `ISP_INTERFACE` в конфиге и при необходимости предлагает установить правильное значение. При чтении значения снимаются `\r`, кавычки и пробелы (защита от CRLF в выводе меню).
- Проверяет наличие **глобального IPv6** (`scope global`: 2a00, 2a02, 2606, 2001, …) на выбранном `ISP_INTERFACE` (или default route) и выставляет `IPV6_ENABLED=0|1`. Несколько iface через пробел — достаточно одного с global.

#### Восстановление rkn.list

- Если до смены стратегии в `MODE_LIST` уже был `--hostlist=…/rkn.list`, привязка **восстанавливается** автоматически.

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

| Версия | Путь |
| --- | --- |
| nfqws-keenetic (v1) | `/etc/nfqws/ipset.list` |
| nfqws2-keenetic (v2) | `/etc/nfqws2/lists/ipset.list` |

Бэкап, очистка пустых строк/комментариев, перезапуск сервиса. При двух установленных версиях — можно обновить обе.

### 4. Загрузить rkn.list (125k+ доменов)

Доступно при установленном **nfqws-keenetic** (v1) и/или **nfqws2-keenetic** (v2). При обеих версиях — выбор: 1 / 2 / обе.

- Если `rkn.list` уже есть — показывает размер в **КБ** (без медленного подсчёта 125k строк) и спрашивает, обновлять ли список (**по умолчанию: Нет**). При отказе скачивание пропускается.
- Скачивает большой список доменов РКН из [IndeecFOX/zapret4rocket](https://github.com/IndeecFOX/zapret4rocket)\
  (`extra_strats/TCP/RKN/List.txt`). Сначала **GitHub raw**, затем jsDelivr / Fastly / ghproxy; при недоступности — зеркало `mizulina.shit.vc`.
- Записывает: v1 → `/etc/nfqws/rkn.list`, v2 → `/etc/nfqws2/lists/rkn.list`.
- Добавляет `--hostlist=…/rkn.list` в `MODE_LIST` соответствующего конфига (с бэкапом), если его ещё нет. Вставка через **awk** (устойчиво к CRLF, пробелам, пустым кавычкам).
- **Перезапуск** сервиса только при реальных изменениях (обновлён список и/или изменён `MODE_LIST`). Если hostlist уже был в конфиге и список не обновляли — перезапуск не выполняется.

### 5. Обход блокировки DoT/DoH

Доступно при установленном **nfqws-keenetic** и/или **nfqws2-keenetic** (при обеих версиях — выбор v1 / v2 / обе).

Добавляет в `NFQWS_ARGS_CUSTOM` стратегию обхода блокировок публичных DoT/DoH DNS:

- **nfqws1** — [`strategies/dns_filter_nfqws1`](strategies/dns_filter_nfqws1) (классический `--dpi-desync`)
- **nfqws2** — [`strategies/dns_filter_nfqws2`](strategies/dns_filter_nfqws2) (lua-desync)

Покрывает Cloudflare, Google, AdGuard, NextDNS, Quad9, DNS.SB, Alibaba, Xbox-DNS, Geohide и др.; при необходимости добавляет порт `853` в `TCP_PORTS` / `UDP_PORTS` и перезапускает соответствующий сервис. Стратегия сначала скачивается с GitHub, при недоступности сети — встроенный fallback (пути blob-файлов подставляются под платформу).

### 6. Смена активных fake:blob

Позволяет заменить используемые в конфиге `fake:blob=NAME` на другой `.bin`-файл (переназначает путь в `--blob=NAME:…`).

- Быстрый разбор конфига (v1/v2) одним проходом **awk**: секции `NFQWS_*ARGS*`, map `--blob=name:path`, уникальные `fake:blob=` (hex пропускаются).
- Показывает список найденных имён по секциям.
- Кандидаты `.bin`: локальные → имена из **`strategies/blobs/SHA256SUMS`** → GitHub API → встроенный fallback.
- При выборе файла из репозитория — скачивает его, обновляет `--blob=…`, бэкап конфига.
- Перезапуск сервиса — по подтверждению (по умолчанию Да).

### 7. Обновление hosts

Статические DNS-привязки через управляемый блок с маркерами `# >>> nfqws-menu hosts >>>` в `/etc/hosts`; dnsmasq подхватывает его автоматически, после изменений сервис перечитывается. Удаление — чистка блока, чужие записи в `/etc/hosts` не затрагиваются.

- Скачивает файл `hosts` из репозитория.
- Секции задаются комментариями `# Имя секции`; внутри — строки `IP DOMAIN`.
- Можно выбрать одну или несколько секций, **все**, либо **88** — удалить записи, **99** — просмотреть текущие.
- Только **IPv4**; битые домены (`..`, ведущая `.`) пропускаются; **один IP на домен**.

### 8. Управление DoT/DoH

Управление DNS через пакет `https-dns-proxy` (DoH): серверы как uci-секции, привязка домена — `server=/домен/127.0.0.1#порт` в `dhcp.@dnsmasq[0].server`. Режим выборочных доменов: остальной DNS идёт как раньше.

Подменю:

```
1) Добавить DoH сервер (https-dns-proxy)
2) Привязать домен к DoH серверу
3) Удалить привязку домена
4) Удалить DoH сервер
0) Назад
```

Возможности:
- Просмотр текущих DoH-серверов и персональных привязок к доменам
- Пресеты DoH: Cloudflare, Google, Quad9, AdGuard, DNS.SB, CleanBrowsing и др., либо ручной ввод
- Удаление upstream-ов и привязок с автоперезапуском сервисов

### 10. dpi-detector

Инструмент для анализа цензуры трафика в России: обнаруживает и классифицирует блокировки сайтов, хостингов и CDN (TCP16–20 блокировки), а также подмену DNS-запросов провайдером.

Актуальная версия [dpi-detector](https://github.com/Runnin4ik/dpi-detector) (ветка `rust`). Бинарник сохраняется в `/usr/local/bin` (OpenWrt персистентный overlay).

Если бинарник уже есть — сразу запускает его.

### 14. MagiTrickle

Утилита для точечной маршрутизации сетевого трафика по заданным доменным именам.

Установка / обновление [MagiTrickle](http://bin.magitrickle.dev/):
- **Не установлен** — добавление репозитория (`add_repo.sh`) + установка пакета + запуск сервиса
- **Установлен** → обновление пакета + restart сервиса
- Init: `/etc/init.d/magitrickle` (procd)
- Удаление — в п. 88 (с опциональным удалением файла репозитория)

### 16. TG WS Proxy Rust (tg-ws-proxy-rs)

[tg-ws-proxy-rs](https://github.com/valnesfjord/tg-ws-proxy-rs) — MTProto-прокси на Rust с LuCI-интеграцией на OpenWrt 24.10+. Меню добавляет:

- **секрет и порт прежней установки** (если осталась) — ссылки `tg://` не меняются;
- **выбор сборки** — обычная или компактная (UPX), таблицей: размер на флеше и потребление памяти;
- **подбор параметров по замерам** — CF-домены и варианты лестницы проверяются штатной пробой бинаря, затем выводится список с временем каждого;
- **проверка связи до выдачи ссылки** — `resPQ` от DC Telegram, то есть вся цепочка, а не «порт открыт».

Пути: `/usr/bin/tg-ws-proxy-rs`, `/etc/init.d/tg-ws-proxy-rs`, конфиги `/etc/tg-ws-proxy-rs/`.

### S. Сервисные утилиты

В главном меню: клавиша **S** (или раздел «СЕРВИС»).

| Пункт | Действие |
| --- | --- |
| 1 | **Сжать bin/sbin (UPX)** — `upx --lzma --best` для исполняемых файлов |
| U | **Обновить все пакеты** — `opkg update && opkg upgrade` / `apk update && apk upgrade` (также клавиша **U** из главного меню) |

### 77. Change language

Мгновенное переключение интерфейса **ru ↔ en** (файл `/etc/nfqws-menu/nfqws-menu.lang`).

### 88. Удаление пакетов

Показывает установленные компоненты и позволяет удалить выборочно:

```
Удаление:
  [N] nfqws2-keenetic / nfqws-keenetic-web / dpi-detector /
      tg-ws-proxy-rs / magitrickle
  [a] Удалить все пакеты NFQWS
  [b] Удалить резервные копии (.bak.* / *-opkg)
  [0] Назад
```

- Пакеты NFQWS — `opkg remove` / `apk del`
- **dpi-detector** — `/usr/local/bin/dpi-detector`
- **tg-ws-proxy-rs** — бинарь, init, конфиги
- **magitrickle** — удаление пакета + запрос на удаление файла репозитория
- **a)** — все пакеты NFQWS + dpi-detector
- **b)** — `*.bak.*`, `*.conf-opkg`, `*.list-opkg` в каталогах конфигов nfqws

### 99. Обновить скрипт

Скачивает свежую версию **этого форка**, перезаписывает скрипт (`/usr/bin/nfqws-menu.sh`), обновляет symlink `menu`, сразу перезапускает меню (`exec`).

---

## Changelog

### 0.1.1

Ревью кода после адаптации: мёртвый код, дубли, найденные баги.

- **Зеркала установки** — добавлены быстрые зеркала jsDelivr / Fastly / ghproxy / GitHub raw
- **П. 99 (обновить скрипт)** — обновление пишет в тот же файл, на который ссылается `menu` (`menu_script_target()`), и перелинковывает symlink; раньше обновление в `/usr/bin/nfqws-menu.sh` оставляло `menu` указывать на старую копию
- **П. 7 (hosts, OpenWrt)** — новые записи **добавляются к существующему блоку**, а не заменяют его (раньше ранее добавленные домены молча стирались); общий хелпер `hosts_strip_block()`/`hosts_existing_pairs()`
- **П. 8 (DoH, OpenWrt)** — `hdp_instances()` считает **эффективный** порт инстанса (пакет выдаёт `listen_port` по счётчику от 5053 на каждый инстанс), поэтому `hdp_next_port()` больше не может выдать уже занятый порт
- **П. 16 (tg-ws-proxy-rs)** — архитектура OpenWrt содержит суффиксы через `_` (`mipsel_24kc`, `aarch64_cortex-a53`): цель определяется по префиксу, добавлен fallback на `apk --print-arch` и `uname -m`; таблица размеров больше не пустая на типовых роутерах
- **П. 88 (MagiTrickle)** — на apk файл репозитория ищется по подстроке в `/etc/apk/repositories.d`, а не только по имени `/etc/opkg/magitrickle.conf`
- **П. 7** — имена секций hosts с пробелами больше не теряются (чтение построчно вместо word splitting)
- `/opt/etc/nfqws*` → `/etc/nfqws*` в стратегиях вынесено в `strategy_retarget_paths()` (было три копии)
- Очистка Keenetic-only кода: удалены `dropbear_fix`, `menu-dns.sh`, `ndmc_cli`, RCI-проверки, KeenKit
- Добавлен `.gitattributes` — `.sh`/`.conf`/`hosts`/`SHA256SUMS` всегда LF (CRLF ломает busybox ash)

### 0.1.0 (форк Dea1hwa1cher/nfqws-menu-openwrt)

Первый релиз адаптации под OpenWrt. База — апстрим 0.9.41; вся история апстрима (0.4.0–0.9.41) — в [исходном репозитории](https://github.com/rndnaame/nfqws-menu#changelog).

- **Определение ОС** — OpenWrt / ImmortalWrt; переменные платформы (пути, статус opkg, фиды), модель/версия из `/tmp/sysinfo/model` и `/etc/openwrt_release`
- **Пакеты** — нативный фид `https://nfqws.github.io/<repo>/openwrt` с GPG-ключом; поддержка `apk` (OpenWrt 25.x: `packages.adb` в `/etc/apk/repositories.d`, `.pem` в `/etc/apk/keys`); обёртки `pkg_update/install/remove` над opkg/apk
- **Пути** — все конфиги/сервисы/списки через `CONF_BASE_DIR`/`INIT_BASE_DIR`; сервисы procd `nfqws-keenetic` / `nfqws2-keenetic`; sed-подмена `/opt/etc/nfqws*` → `/etc/nfqws*` при применении стратегий
- **П. 7 (hosts)** — на OpenWrt блок-маркеры в `/etc/hosts` + reload dnsmasq
- **П. 8 (DoT/DoH)** — на OpenWrt менеджер `https-dns-proxy` (uci) + привязки доменов через dnsmasq per-domain
- **П. 5 (DoT/DoH-стратегия)** — полностью работает на OpenWrt
- **Web** — нативная установка `nfqws-keenetic-web` из фида `.../openwrt`
- **Утилиты** — dpi-detector (`/usr/local/bin`), MagiTrickle (нативный пакет), tg-ws-proxy-rs (OpenWrt/LuCI-ветка установщика)
- **Совместимость busybox ash** (проверено на устройстве OpenWrt 24.10): без переносов строк с операторами `||`/`&&`/`|` в начале продолжения; обновление скрипта (п. 99) и стратегии качаются из этого форка

---

## Структура репозитория

```
nfqws-menu-openwrt/
├── nfqws-menu.sh          # Главный скрипт меню
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
2. Полный конфиг (`ISP_INTERFACE=`, `NFQWS_ARGS=` / `NFQWS_BASE_ARGS=` и т.д.).
3. При необходимости — `.bin` в `strategies/blobs/`, списки в `strategies/lists/`.
4. После push workflow обновит `SHA256SUMS` / `strategies/blobs/SHA256SUMS`; скрипт подхватит стратегию из SUMS (или через GitHub API).

---

## Требования

1. OpenWrt 23.05+ (opkg) или 25.x+ (apk); ImmortalWrt — тоже работает.
2. Модули NFQUEUE: `kmod-nft-queue` (fw4 / nftables, 22.03+) или `kmod-ipt-nfqueue` + `iptables-mod-nfqueue` (fw3). Меню предупреждает, если их нет.
3. Для веб-интерфейса ничего дополнительно ставить не нужно — пакет придёт с фидом.

---

## Полезные команды вручную

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
opkg info nfqws2-keenetic
opkg info nfqws-keenetic-web

# Конфиги
vi /etc/nfqws/nfqws.conf
vi /etc/nfqws2/nfqws2.conf

# Привязки доменов DNS (п. 8, https-dns-proxy)
uci show https-dns-proxy

# Быстрый запуск меню
menu
```

---

## Отказ от ответственности

Материалы подготовлены в ознакомительных и научно-технических целях.\
Использование на свой страх и риск. Автор не несёт ответственности за последствия.

Стратегии адаптированы на основе [Flowseal/zapret-discord-youtube](https://github.com/Flowseal/zapret-discord-youtube),\
подготовлены [@Nare51](https://github.com/Nare51) с использованием искусственного интеллекта.\
Официальные пакеты NFQWS: [nfqws](https://github.com/nfqws).\
База меню: [rndnaame/nfqws-menu](https://github.com/rndnaame/nfqws-menu); адаптация под OpenWrt: [@Dea1hwa1cher](https://github.com/Dea1hwa1cher).

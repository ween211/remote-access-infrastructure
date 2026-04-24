# Инфраструктура удалённого доступа

Инфраструктурный проект для развёртывания и обслуживания собственного сервера удалённого доступа на базе **RustDesk Server** и **MeshCentral**.

Проект показывает, как на Linux VPS можно поднять сервер удалённого доступа с:

- RustDesk relay/id-сервером;
- веб-панелью MeshCentral;
- запуском через Docker Compose;
- управлением сервисами через systemd;
- production-запуском Node.js-приложения;
- диагностикой сетевых портов и процессов;
- безопасными примерами конфигураций без приватных данных.

## Стек

- Ubuntu Server
- Docker
- Docker Compose
- RustDesk Server
- MeshCentral
- Node.js
- systemd
- SSH
- Linux networking

## Структура проекта

```text
remote-access-infrastructure/
├── README.md
├── rustdesk/
│   └── docker-compose.example.yml
├── meshcentral/
│   └── config.example.json
├── systemd/
│   └── meshcentral.service.example
└── docs/
    └── diagnostics.md
```
## Что делает проект
Проект представляет собой self-hosted инфраструктуру для удалённого доступа и администрирования устройств.

RustDesk Server используется как сервер маршрутизации и relay-сервер для удалённых подключений. В нём работают два основных сервиса:

hbbs — ID-сервер RustDesk;
hbbr — relay-сервер RustDesk.

MeshCentral используется как веб-панель для удалённого управления устройствами. Он запускается как production Node.js-сервис под управлением systemd.

RustDesk Server

RustDesk развёрнут через Docker Compose.

Пример конфигурации:

services:
  hbbs:
    container_name: hbbs
    image: rustdesk/rustdesk-server:latest
    command: hbbs
    volumes:
      - ./data:/root
    network_mode: "host"
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    image: rustdesk/rustdesk-server:latest
    command: hbbr
    volumes:
      - ./data:/root
    network_mode: "host"
    restart: unless-stopped

Полный пример находится в файле:

rustdesk/docker-compose.example.yml
MeshCentral

MeshCentral развёрнут как production Node.js-сервис и управляется через systemd.

Сервис запускается от отдельного Linux-пользователя и использует ограниченные capabilities, чтобы слушать привилегированные порты без запуска всего процесса от root.

Пример systemd unit-файла:

[Unit]
Description=MeshCentral Server
After=network.target

[Service]
Type=simple
User=mesh
WorkingDirectory=/home/mesh/meshcentral
ExecStart=/usr/bin/node /home/mesh/meshcentral/node_modules/meshcentral/meshcentral
Environment=NODE_ENV=production
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

Полный пример находится в файле:

systemd/meshcentral.service.example
Порты

В данной инфраструктуре используются следующие порты:

22      SSH
80      MeshCentral HTTP redirect
443     MeshCentral HTTPS
4433    MeshCentral Intel AMT / MPS
21115   RustDesk hbbs
21116   RustDesk hbbs TCP/UDP
21117   RustDesk hbbr relay
21118   RustDesk hbbs web client
21119   RustDesk hbbr
Диагностика

Проверка состояния сервисов:

systemctl status meshcentral.service --no-pager
docker ps
ss -tulpn

Просмотр логов MeshCentral:

journalctl -u meshcentral.service -n 100 --no-pager
journalctl -u meshcentral.service -f

Проверка контейнеров RustDesk:

docker logs hbbs --tail=100
docker logs hbbr --tail=100

Дополнительные команды для диагностики находятся в файле:

docs/diagnostics.md

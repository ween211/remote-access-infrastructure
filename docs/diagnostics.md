# Diagnostics

This document contains useful commands for checking the remote access infrastructure status.

## Check running services

```bash
systemctl status meshcentral.service --no-pager
docker ps

Check open ports
ss -tulpn

Expected ports:

22      SSH
80      MeshCentral HTTP redirect
443     MeshCentral HTTPS
4433    MeshCentral Intel AMT / MPS
21115   RustDesk hbbs
21116   RustDesk hbbs TCP/UDP
21117   RustDesk hbbr relay
21118   RustDesk hbbs web client
21119   RustDesk hbbr

Check MeshCentral process
ps aux | grep -Ei 'mesh|node' | grep -v grep
readlink -f /proc/PROCESS_ID/cwd
tr '\0' '\n' < /proc/PROCESS_ID/cmdline

Check RustDesk containers
docker ps
docker logs hbbs --tail=100
docker logs hbbr --tail=100

Restart services
sudo systemctl restart meshcentral.service
docker restart hbbs hbbr

View MeshCentral logs
journalctl -u meshcentral.service -n 100 --no-pager
journalctl -u meshcentral.service -f

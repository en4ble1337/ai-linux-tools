```bash
(crontab -l; echo "0 3 * * 0 docker container prune -f >> /var/log/docker-prune.log 2>&1 && docker image prune -a -f >> /var/log/docker-prune.log 2>&1") | crontab -
```

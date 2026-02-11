```(crontab -l 2>/dev/null; echo "0 3 * * 0 docker image prune -a -f >> /var/log/docker-prune.log 2>&1") | crontab -```

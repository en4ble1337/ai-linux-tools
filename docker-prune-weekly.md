# Basic Weekly
```bash
(crontab -l; echo "0 3 * * 0 docker container prune -f >> /var/log/docker-prune.log 2>&1 && docker image prune -a -f >> /var/log/docker-prune.log 2>&1") | crontab -
```

## 80% disk utilization
```bash
(crontab -l; echo '0 */6 * * * USAGE=$(df /var/lib/docker | awk '\''NR==2 {print $5}'\'' | tr -d '\''%'\''); if [ "$USAGE" -gt 80 ]; then docker container prune -f >> /var/log/docker-prune.log 2>&1 && docker image prune -a -f >> /var/log/docker-prune.log 2>&1; fi') | crontab -
```

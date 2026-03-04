apt-get install -y --allow-downgrades containerd.io=1.7.28-1~ubuntu.24.04~noble
apt-mark hold containerd.io
systemctl restart containerd docker
docker run --rm hello-world

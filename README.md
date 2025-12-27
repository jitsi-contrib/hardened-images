# Hardened Images

### Volumes

```bash
mkdir -p ~/.jitsi-meet-volumes/prosody/{config,prosody-plugins-custom}
mkdir -p ~/.jitsi-meet-volumes/{jibri,jicofo,jigasi,jvb,web}

mkdir -p ~/.jitsi-meet-volumes/storage/{jibri,prosody,transcripts}
chmod 777 ~/.jitsi-meet-volumes/storage/jibri
chmod 777 ~/.jitsi-meet-volumes/storage/prosody
chmod 777 ~/.jitsi-meet-volumes/storage/transcripts

mkdir -p ~/.jitsi-meet-volumes/tmp/{web-crontabs,web-load-test}
chmod 777 ~/.jitsi-meet-volumes/tmp/web-crontabs
chmod 777 ~/.jitsi-meet-volumes/tmp/web-load-test
```

### Environment file

```bash
cp env.example .env
bash gen-passwords.sh

mv .env .env.myconfig

MY_IP=172.18.18.1

cat <<EOF >>.env.myconfig

# Customized values
PUBLIC_URL=https://${MY_IP}:8443
JVB_ADVERTISE_IPS=${MY_IP}
EOF
```

### Running

```bash
# pull
docker compose --env-file .env.myconfig -f docker-compose.yml pull

# run
docker compose --env-file .env.myconfig -f docker-compose.yml up

# stop
docker compose --env-file .env.myconfig -f docker-compose.yml down --remove-orphans
```

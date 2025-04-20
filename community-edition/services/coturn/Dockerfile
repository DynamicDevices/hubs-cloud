#from node:lts as build
#run mkdir certs && openssl req -x509 -newkey rsa:4096 -sha256 -days 36500 -nodes -keyout certs/key.pem -out certs/cert.pem -subj '/CN=dialog'
from coturn/coturn:alpine
#from coturn/coturn:debian
#copy --from=build /certs /certs
copy cert.pem key.pem /certs/
user root:root
run apk add curl netcat-openbsd
#run apt update && apt install -y curl netcat-openbsd

COPY entrypoint.sh /entrypoint.sh
run chmod +x /entrypoint.sh
# user nobody:nogroup
volume ["/var/lib/coturn"]
cmd /entrypoint.sh

# Teleport

Uppdatera repository och sätt tidszon.

```bash
$ timedatectl set-timezone Europe/Stockholm
$ apt update && apt upgrade -y
```

Skaffa certifikat för wildcard-domän från Let's Encrypt.

```bash
$ apt-get install certbot -y
$ certbot certonly --standalone -d *.test227.arxlight.com -n --agree-tos --email=exampel@email.com
```

Ladda ner Teleport, konfigurera den med domännamnet, genrera konfigurationsfil och starta Teleport.

```bash
$ curl https://goteleport.com/static/install.sh | bash -s 12.4.3

$ teleport configure --acme --acme-email=exampel@email.com --cluster-name=test227.arxlight.com | \
$ sudo tee /etc/teleport.yaml > /dev/null

$ systemctl enable teleport --now
```

Lägg in CA pin i konfigurationsfilen och filsökvägen till wildcard-domäns certifikat och nyckel.

```bash
$ tctl status
Cluster      test227.arxlight.com
Version      12.4.3
host CA      never updated
user CA      never updated
db CA        never updated
openssh CA   never updated
jwt CA       never updated
saml_idp CA  never updated
oidc_idp CA  never updated
CA pin       sha256:[CA pin]
```
```bash
$ vim /etc/teleport.yaml
```
```yml
version: v3
teleport:
  nodename: test227.arxlight.com
  data_dir: /var/lib/teleport
  log:
    output: stderr
    severity: INFO
    format:
      output: text
  ca_pin: "sha256:[CA pin]"
  diag_addr: ""
auth_service:
  enabled: "yes"
  listen_addr: 0.0.0.0:3025
  cluster_name: test227.arxlight.com
  proxy_listener_mode: multiplex
ssh_service:
  enabled: "yes"
  commands:
  - name: hostname
    command: [hostname]
    period: 1m0s
proxy_service:
  enabled: "yes"
  web_listen_addr: 0.0.0.0:443
  public_addr: test227.arxlight.com:443
  https_keypairs:
  - key_file: /etc/letsencrypt/live/*.test227.arxlight.com/privkey.pem
    cert_file: /etc/letsencrypt/live/*.test227.arxlight.com/fullchain.pem
  https_keypairs_reload_interval: 0s
  acme:
    enabled: "yes"
    email: karl.hubinette@hotmail.se
```
```bash
$ systemctl restart teleport
```

Nu kan man nå Teleport webbgränssnitt genom https://test227.arxlight.com och kommer till inloggningssidan.
Skapa användare genom Teleports admin verktyg "tctl".

```bash
$ tctl users add [Usrname] --logins=root --roles=editor,auditor,access
```

* --logins - Maskininloggning, t. ex om jag har användare root kan jag logga in på en maskin genom SSH som root.
* --roles - Vad användare kan göra på teleports web interface. editor = roll tillåter användare att ändra klusterkonfiguration. auditor =  roll för att se granskningsloggar. access = roll för att komma åt klusterresurser.
  
När man kör de kommandot kommer du se ut ungefär såhär:

```bash
User "(Usr)" has been created but requires a password. Share this URL with the user to complete user setup, link is valid for 1h:
https://test227.arxlight.com:443/web/invite/f421d64a26c1e4d08f0387f9aa587b7a

NOTE: Make sure test227.arxlight.com:443 points at a Teleport proxy which users can access.
```
Kopiera länken och klistra in det i en webbläsare skapa ett lösenord skanna QR-koden med din valda authenticator app (t. ex Google Authenticator).

## Ubuntu klient

Uppdatera repository och sätt tidszon.

```bash
$ timedatectl set-timezone Europe/Stockholm
$ apt update && apt upgrade -y
```

Installera Teleport.

```bash
$ sudo curl https://apt.releases.teleport.dev/gpg \
-o /usr/share/keyrings/teleport-archive-keyring.asc

$ source /etc/os-release

$ echo "deb [signed-by=/usr/share/keyrings/teleport-archive-keyring.asc] \
https://apt.releases.teleport.dev/${ID?} ${VERSION_CODENAME?} stable/v12" \
| sudo tee /etc/apt/sources.list.d/teleport.list > /dev/null

$ sudo apt-get update
$ sudo apt-get install teleport
```

Sen för att få maskinen att gå med i klustret behövs CA pin och en token som finns i Teleport servern.

I teleport servern.
```bash
$ tctl tokens add --type=node
The invite token: [Token]
This token will expire in 60 minutes.

Run this on the new node to join the cluster:

> teleport start \
   --roles=node \
   --token=[Token] \
   --ca-pin=sha256:[CA pin] \
   --auth-server=192.168.1.100:3025

Please note:

  - This invitation token will expire in 60 minutes
  - 192.168.1.100:3025 must be reachable from the new node
```

Gå tillbaka till din klient skapa teleport.yml i /etc skriv in nodename, token, proxy server och ca_pin.

```bash
$ vim /etc/teleport.yaml
```
```yml
version: v3
teleport:
  nodename: Ubuntu
  data_dir: /var/lib/teleport
  join_params:
    token_name: [TOKEN]
    method: token
  proxy_server: test227.arxlight.com:443
  log:
    output: stderr
    severity: INFO
    format:
      output: text
  ca_pin: "sha256:[CA pin]"
  diag_addr: ""
auth_service:
  enabled: "no"
ssh_service:
  enabled: "yes"
  commands:
  - name: hostname
    command: [hostname]
    period: 1m0s
proxy_service:
  enabled: "no"
  https_keypairs: []
  https_keypairs_reload_interval: 0s
  acme: {}
```

```bash
$ systemctl enable teleport --now #Starta Teleport
$ rm -rf /var/lib/teleport #Om man får felmeddelanden
```

## Mediawiki klient

Uppdatera repository och sätt tidszon.

```bash
$ timedatectl set-timezone Europe/Stockholm
$ apt update && apt upgrade -y
```

Installera Teleport.

```bash
$ sudo curl https://apt.releases.teleport.dev/gpg \
-o /usr/share/keyrings/teleport-archive-keyring.asc

$ source /etc/os-release

$ echo "deb [signed-by=/usr/share/keyrings/teleport-archive-keyring.asc] \
https://apt.releases.teleport.dev/${ID?} ${VERSION_CODENAME?} stable/v12" \
| sudo tee /etc/apt/sources.list.d/teleport.list > /dev/null

$ sudo apt-get update
$ sudo apt-get install teleport
```

Sen för att få maskinen att gå med i klustret behövs CA pin och en token som finns i Teleport servern.

I teleport servern.
```bash
$ tctl tokens add --type=node,app
The invite token: [Token]
This token will expire in 60 minutes.

Fill out and run this command on a node to make the application available:

> teleport app start \
   --token=[Token] \
   --ca-pin=sha256:[CA pin] \
   --auth-server=test227.arxlight.com:443 \
   --name=example-app                    `# Change "example-app" to the name of your application.` \
   --uri=http://localhost:8080           `# Change "http://localhost:8080" to the address of your application.`

Your application will be available at example-app.test227.arxlight.com:443.

Please note:

  - This invitation token will expire in 60 minutes.
  - test227.arxlight.com:443 must be reachable from the new application service.
  - Update DNS to point example-app.test227.arxlight.com:443 to the Teleport proxy.
  - Add a TLS certificate for example-app.test227.arxlight.com:443 to the Teleport proxy under "https_keypairs".
```

Gå tillbaka till din klient skapa teleport.yml i /etc skriv in nodename, token, proxy server, ca_pin och app tjänsten.

```bash
$ vim /etc/teleport.yaml
```

```yml
version: v3
teleport:
  nodename: Mediawiki
  data_dir: /var/lib/teleport
  join_params:
    token_name: [TOKEN]
    method: token
  proxy_server: test227.arxlight.com:443
  log:
    output: stderr
    severity: INFO
    format:
      output: text
  ca_pin: "sha256:[CA pin]"
  diag_addr: ""
auth_service:
  enabled: "no"
ssh_service:
  enabled: "yes"
  commands:
  - name: hostname
    command: [hostname]
    period: 1m0s
proxy_service:
  enabled: "no"
  https_keypairs: []
  https_keypairs_reload_interval: 0s
  acme: {}
app_service:
  enabled: "yes"
  debug_app: true
  apps:
  - name: "wiki"
    uri: "http://192.168.1.150"
```

```bash
$ systemctl enable teleport --now #Starta Teleport
$ rm -rf /var/lib/teleport #Om man får felmeddelanden
```

## Rollbaserad åtkomstkontroll

För att göra Rollbased åtkomst åt Frank användaren börja med att lägga till etiketten "environment: ubuntu" i SSH-tjänsten.

```bash
$ vim /etc/teleport.yaml
```
```yml
version: v3
teleport:
  nodename: ssh
  data_dir: /var/lib/teleport
  join_params:
    token_name: 532ef5a7eb2ef8fa81e0384b6495ae57
    method: token
  proxy_server: test227.arxlight.com:443
  log:
    output: stderr
    severity: INFO
    format:
      output: text
  ca_pin: "sha256:62031fc1225d344cbdc78243c7ea6880364bd2f471cc3a9791dd52df9e6a036b"
  diag_addr: ""
auth_service:
  enabled: "no"
ssh_service:
  enabled: "yes"
  commands:
  - name: hostname
    command: [hostname]
    period: 1m0s
  labels:
    environment: ubuntu
proxy_service:
  enabled: "no"
  https_keypairs: []
  https_keypairs_reload_interval: 0s
  acme: {}
```

Starta om Teleport.
```bash
$ systemctl restart teleport
```

Sen i webbgränssnittet gå till Management -> Roles -> CREATE NEW ROLE skriv in följande.
```yml
kind: role
version: v6
metadata:
  name: ubuntu
spec:
  allow:
    logins: ['frank']
    node_labels:
      'environment': 'ubuntu'
```
Spara och gå vidare till Users -> CREATE NEW USER, USERNAME: Frank USER ROLES: ubuntu, när man trycker spara sen får man en länk som man kan skicka till användaren för att skapa lösenord och koppla en autentiseringsapp.
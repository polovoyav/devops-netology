# pcs-devsys-diplom-01

## 1. Создайте виртуальную машину Linux.

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.hostname = "sandbox"
  config.vm.network "public_network"
  config.vm.provider "virtualbox" do |v|
	v.memory = 4096
  end
end
```
## 2. Установите ufw и разрешите к этой машине сессии на порты 22 и 443, при этом трафик на интерфейсе localhost (lo) должен ходить свободно на все порты.
```
vagrant@sandbox:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
vagrant@sandbox:~$ sudo ufw allow 22/tcp
Rule added
Rule added (v6)
vagrant@sandbox:~$ sudo ufw allow 443/tcp
Rule added
Rule added (v6)
vagrant@sandbox:~$ sudo ufw allow from 127.0.0.1
Rule added
```

## 3. Установите hashicorp vault (инструкция по ссылке).

### Добавляем HashiCorp GPG key:
```
vagrant@sandbox:~$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
OK
```

### Добавляем репозиторий HashiCorp:
```
vagrant@sandbox:~$ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
Hit:1 http://us.archive.ubuntu.com/ubuntu focal InRelease
Get:2 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:3 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:4 http://us.archive.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:5 https://apt.releases.hashicorp.com focal InRelease [14.6 kB]
Get:6 https://apt.releases.hashicorp.com focal/main amd64 Packages [50.1 kB]
Fetched 401 kB in 11s (35.0 kB/s)
Reading package lists... Done
```

### Обновляем репозитории и устанавливаем vault:
```
vagrant@sandbox:~$ sudo apt-get update && sudo apt-get install vault
Hit:1 http://us.archive.ubuntu.com/ubuntu focal InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease
Hit:4 http://us.archive.ubuntu.com/ubuntu focal-security InRelease
Err:5 https://apt.releases.hashicorp.com focal InRelease
  Could not connect to apt.releases.hashicorp.com:443 (44.216.227.213), connection timed out
Reading package lists... Done
W: Failed to fetch https://apt.releases.hashicorp.com/dists/focal/InRelease  Could not connect to apt.releases.hashicorp.com:443 (44.216.227.213), connection timed out
W: Some index files failed to download. They have been ignored, or old ones used instead.
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  vault
0 upgraded, 1 newly installed, 0 to remove and 55 not upgraded.
Need to get 69.6 MB of archives.
After this operation, 188 MB of additional disk space will be used.
Get:1 https://apt.releases.hashicorp.com focal/main amd64 vault amd64 1.9.4 [69.6 MB]
Fetched 69.6 MB in 2min 0s (579 kB/s)
Selecting previously unselected package vault.
(Reading database ... 40620 files and directories currently installed.)
Preparing to unpack .../archives/vault_1.9.4_amd64.deb ...
Unpacking vault (1.9.4) ...
Setting up vault (1.9.4) ...
Generating Vault TLS key and self-signed certificate...
Generating a RSA private key
..++++
........................................................................................++++
writing new private key to 'tls.key'
-----
Vault TLS key and self-signed certificate have been generated in '/opt/vault/tls'.
```
### Проверяем установку vault:
```
vagrant@sandbox:~$ vault
Usage: vault <command> [args]

Common commands:
    read        Read data and retrieves secrets
    write       Write data, configuration, and secrets
    delete      Delete secrets and configuration
    list        List data or secrets
    login       Authenticate locally
    agent       Start a Vault agent
    server      Start a Vault server
    status      Print seal and HA status
    unwrap      Unwrap a wrapped secret

Other commands:
    audit          Interact with audit devices
    auth           Interact with auth methods
    debug          Runs the debug command
    kv             Interact with Vault's Key-Value storage
    lease          Interact with leases
    monitor        Stream log messages from a Vault server
    namespace      Interact with namespaces
    operator       Perform operator-specific tasks
    path-help      Retrieve API help for paths
    plugin         Interact with Vault plugins and catalog
    policy         Interact with policies
    print          Prints runtime configurations
    secrets        Interact with secrets engines
    ssh            Initiate an SSH session
    token          Interact with tokens
```

## 4. Cоздайте центр сертификации по инструкции (ссылка) и выпустите сертификат для использования его в настройке веб-сервера nginx (срок жизни сертификата - месяц).

```
vagrant@sandbox:~$ sudo apt-get install jq
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  libjq1 libonig5
The following NEW packages will be installed:
  jq libjq1 libonig5
0 upgraded, 3 newly installed, 0 to remove and 55 not upgraded.
Need to get 313 kB of archives.
After this operation, 1,062 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 libonig5 amd64 6.9.4-1 [142 kB]
Get:2 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 libjq1 amd64 1.6-1ubuntu0.20.04.1 [121 kB]
Get:3 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 jq amd64 1.6-1ubuntu0.20.04.1 [50.2 kB]
Fetched 313 kB in 7s (44.3 kB/s)
Selecting previously unselected package libonig5:amd64.
(Reading database ... 40626 files and directories currently installed.)
Preparing to unpack .../libonig5_6.9.4-1_amd64.deb ...
Unpacking libonig5:amd64 (6.9.4-1) ...
Selecting previously unselected package libjq1:amd64.
Preparing to unpack .../libjq1_1.6-1ubuntu0.20.04.1_amd64.deb ...
Unpacking libjq1:amd64 (1.6-1ubuntu0.20.04.1) ...
Selecting previously unselected package jq.
Preparing to unpack .../jq_1.6-1ubuntu0.20.04.1_amd64.deb ...
Unpacking jq (1.6-1ubuntu0.20.04.1) ...
Setting up libonig5:amd64 (6.9.4-1) ...
Setting up libjq1:amd64 (1.6-1ubuntu0.20.04.1) ...
Setting up jq (1.6-1ubuntu0.20.04.1) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
```
### Устанавливаем часовой пояс:
```
vagrant@sandbox:~$ timedatectl set-timezone Europe/Moscow
==== AUTHENTICATING FOR org.freedesktop.timedate1.set-timezone ===
Authentication is required to set the system timezone.
Authenticating as: vagrant
Password:
==== AUTHENTICATION COMPLETE ===
vagrant@sandbox:~$ date
Mon 07 Mar 2022 02:54:42 PM MSK
```

### Открываем файл конфигурации vault, настраиваем порт, отключаем tls:
```
vagrant@sandbox:~$ sudo nano /etc/vault.d/vault.hcl
# HTTP listener
listener "tcp" {
  address = "127.0.0.1:8201"
  tls_disable = 1
}
```

### Запускаем сервер vault в отдельном терминале:
```
vagrant@sandbox:~$ sudo vault server -config /etc/vault.d/vault.hcl
==> Vault server configuration:

                     Cgo: disabled
              Go Version: go1.17.7
              Listener 1: tcp (addr: "127.0.0.1:8201", cluster address: "127.0.0.1:8202", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
              Listener 2: tcp (addr: "0.0.0.0:8200", cluster address: "0.0.0.0:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "enabled")
               Log Level: info
                   Mlock: supported: true, enabled: true
           Recovery Mode: false
                 Storage: file
                 Version: Vault v1.9.4
             Version Sha: fcbe948b2542a13ee8036ad07dd8ebf8554f56cb

==> Vault server started! Log data will stream in below:

2022-03-07T15:34:53.189+0300 [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
2022-03-07T15:34:53.192+0300 [WARN]  no `api_addr` value specified in config or in VAULT_API_ADDR; falling back to detection if possible, but this value should be manually set
2022-03-07T15:34:54.231+0300 [INFO]  core: Initializing VersionTimestamps for core
```

### Создаем переменную окружения:
```
vagrant@sandbox:~$ export VAULT_ADDR=http://127.0.0.1:8201
```

### Проверяем состояние vault:
```
vagrant@sandbox:~$ vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        false
Sealed             true
Total Shares       0
Threshold          0
Unseal Progress    0/0
Unseal Nonce       n/a
Version            1.9.4
Storage Type       file
HA Enabled         false
```

### Инициализируем и распечатываем vault:
```
vagrant@sandbox:~$ vault operator init
Unseal Key 1: EZYRI+mOkZyI9FsC8Z+ijRhlsz/upa0uW24b1xOk0tK6
Unseal Key 2: SFz3wTVzhEv9G+HKbk9YQNwazwESLXTBFztiO2lgZWqb
Unseal Key 3: McwCdCjuHVl44Upps5+D3rcnBiZBNpP9SsrdnJ8bEWZA
Unseal Key 4: I5A4ZAv3cGwa1DRDLW7i4eABFs84KGQqo72mg0RKfNQC
Unseal Key 5: Om+KxBt68H6i06mQ5GToXKdpBjicZ5hwOVdn8f5S16V/

Initial Root Token: s.qNuttdx2AfNWURdNaAS6Y79k

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 keys to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.

vagrant@sandbox:~$ vault operator unseal
Unseal Key (will be hidden):
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    1/3
Unseal Nonce       357fb4dd-356d-744f-093c-a0ad855ecf56
Version            1.9.4
Storage Type       file
HA Enabled         false
vagrant@sandbox:~$ vault operator unseal
Unseal Key (will be hidden):
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    2/3
Unseal Nonce       357fb4dd-356d-744f-093c-a0ad855ecf56
Version            1.9.4
Storage Type       file
HA Enabled         false
vagrant@sandbox:~$ vault operator unseal
Unseal Key (will be hidden):
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.9.4
Storage Type    file
Cluster Name    vault-cluster-53c06b9e
Cluster ID      86339f51-941c-66c1-b805-1019c6db5283
HA Enabled      false
```

### vault распечатан, готов к работе. Логинимся на сервер vault с помощью Initial Root Token:
```
vagrant@sandbox:~$ vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.qNuttdx2AfNWURdNaAS6Y79k
token_accessor       qtj4ppNkdqF0cMncbCA7Udcu
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

### Создаем корневой центр сертификации по пути pki:
```
vagrant@sandbox:~$ vault secrets enable pki
Success! Enabled the pki secrets engine at: pki/
```

### Настраиваем время жизни сертификатов для pki:
```
vagrant@sandbox:~$ vault secrets tune -max-lease-ttl=87600h pki
Success! Tuned the secrets engine at: pki/
```

### Генерируем корневой сертификат CA, сохраняем в CA_cert.crt:
```
vagrant@sandbox:~$ vault write -field=certificate pki/root/generate/internal \
>     common_name="example.com" \
>     ttl=87600h > CA_cert.crt

vagrant@sandbox:~$ ll
total 52
drwxr-xr-x 5 vagrant vagrant 4096 Mar  8 14:17 ./
drwxr-xr-x 3 root    root    4096 Dec 19 22:42 ../
-rw------- 1 vagrant vagrant 2297 Mar  8 12:51 .bash_history
-rw-r--r-- 1 vagrant vagrant  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 vagrant vagrant 3771 Feb 25  2020 .bashrc
-rw-rw-r-- 1 vagrant vagrant 1171 Mar  8 14:17 CA_cert.crt
...
```

### Настраиваем url для CA и Certificate Revocation List (CRL):
```
vagrant@sandbox:~$ vault write pki/config/urls \
>      issuing_certificates="$VAULT_ADDR/v1/pki/ca" \
>      crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
Success! Data written to: pki/config/urls
```

### Cоздаём промежуточный центр сертификации по пути pki_int:
```
vagrant@sandbox:~$ vault secrets enable -path=pki_int pki
Success! Enabled the pki secrets engine at: pki_int/
```

### Настраиваем время жизни сертификатов для pki_int:
```
vagrant@sandbox:~$ vault secrets tune -max-lease-ttl=43800h pki_int
Success! Tuned the secrets engine at: pki_int/
```

### Создаём промежуточный сертификат CSR, сохраняем в pki_intermediate.csr:
```
vagrant@sandbox:~$ vault write -format=json pki_int/intermediate/generate/internal \
>      common_name="example.com Intermediate Authority" \
>      | jq -r '.data.csr' > pki_intermediate.csr
```

### Подписываем промежуточный сертификат закрытым ключом корневого CA и сохраняем в intermediate.cert.pem:
```
vagrant@sandbox:~$ vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr \
>      format=pem_bundle ttl="43800h" \
>      | jq -r '.data.certificate' > intermediate.cert.pem

vagrant@sandbox:~$ ll
total 60
drwxr-xr-x 5 vagrant vagrant 4096 Mar  8 14:49 ./
drwxr-xr-x 3 root    root    4096 Dec 19 22:42 ../
-rw------- 1 vagrant vagrant 2297 Mar  8 12:51 .bash_history
-rw-r--r-- 1 vagrant vagrant  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 vagrant vagrant 3771 Feb 25  2020 .bashrc
-rw-rw-r-- 1 vagrant vagrant 1171 Mar  8 14:17 CA_cert.crt
drwx------ 3 vagrant vagrant 4096 Mar  7 13:25 .cache/
-rw-rw-r-- 1 vagrant vagrant 1326 Mar  8 14:49 intermediate.cert.pem
drwxrwxr-x 3 vagrant vagrant 4096 Mar  7 14:15 .local/
-rw-rw-r-- 1 vagrant vagrant  924 Mar  8 14:45 pki_intermediate.csr
```

### Импортируем подписанный промежуточный сертификат обратно в vault:
```
vagrant@sandbox:~$ vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem
Success! Data written to: pki_int/intermediate/set-signed
```

### Создаем роль example-dot-com, разрешаем поддомены:
```
vagrant@sandbox:~$ vault write pki_int/roles/example-dot-com \
>      allowed_domains="example.com" \
>      allow_subdomains=true \
>      max_ttl="720h"
Success! Data written to: pki_int/roles/example-dot-com
```

### Запрашиваем создание сертификата для домена test.example.com на месяц:
```
vagrant@sandbox:~$ vault write pki_int/issue/example-dot-com common_name="test.example.com" ttl="720h"
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDpjCCAo6gAwIBAgIUbWftbElyhjmdaxqIVw73JyUiybcwDQYJKoZIhvcNAQEL
BQAwFjEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMjIwMzA4MTE0ODU5WhcNMjcw
MzA3MTE0OTI5WjAtMSswKQYDVQQDEyJleGFtcGxlLmNvbSBJbnRlcm1lZGlhdGUg
QXV0aG9yaXR5MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1//Fxk6t
5k+vwiOkmm9Au/q+rRpGMGGCUvGxpxIsrV+X7jF5ZKp0O0s8R7XR6F1fjN+92i2S
f1mePzuLJuJnHzPFqxbcol4Y7uu7jD/HeTEBuVGuWp1dD3X+qSrxfE58rdof8GPU
ayF+fQFSq4fOne5MP2we7gb8sEp0ya8rAcql72yhDf920semoT+qTO0Buoa2UEUd
+sNrMiUcG2c1HDiEQMq5B8k3Ss0DmIf+UfPK69/6EtE0jWDlx6KqnBJIyO1tXYne
cX9HC4D1nQMiww96Obf4zivNoTkklTD/7sWuwBJzI3u0zokR6x9kNhEVn/nK3h+U
JJjED4PJ0XLjTwIDAQABo4HUMIHRMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8E
BTADAQH/MB0GA1UdDgQWBBQfi3WCzSsuJiKQwXqsMkTnuKYx+zAfBgNVHSMEGDAW
gBRXSxpw8FLbT8gZXuCyFgy0SmUH8jA7BggrBgEFBQcBAQQvMC0wKwYIKwYBBQUH
MAKGH2h0dHA6Ly8xMjcuMC4wLjE6ODIwMS92MS9wa2kvY2EwMQYDVR0fBCowKDAm
oCSgIoYgaHR0cDovLzEyNy4wLjAuMTo4MjAxL3YxL3BraS9jcmwwDQYJKoZIhvcN
AQELBQADggEBAHLrlI0V/6JWEhofah4J64Dm7PzS1k9AMLa1n+QjimDzEaRuk/ju
9ZNDgTI/icKf+Rzc+Lal6bHKg/AoYqZjA00s4OyPtshcNzlzMqkceoK+fzr631ss
Rd8prdIgQDVygBZqqQaDGwxqTzaciYq1dj5P4/4YaZiLHJv1XD9xAYn9VQKR783U
wRHuGKuHr7kpqK/zvoKu9htZ/xJ+yjbS8SmpZp8zZWmuJY0tH7ideFeoCiv8NhSP
LQavewgDBM8EH0wNN9S8lZRTGHpfmUVDxtDGEQXffLoZMt7hnVPQhF2PJs3pjxQ9
qPuna5NanXpcSXnkvHyUAeF9eHmxzS4TrkI=
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZjCCAk6gAwIBAgIUMyDnlm4ybBCSUVv94V8dYt+8XVMwDQYJKoZIhvcNAQEL
BQAwLTErMCkGA1UEAxMiZXhhbXBsZS5jb20gSW50ZXJtZWRpYXRlIEF1dGhvcml0
eTAeFw0yMjAzMDgxMjA2NTdaFw0yMjA0MDcxMjA3MjdaMBsxGTAXBgNVBAMTEHRl
c3QuZXhhbXBsZS5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDy
uCGy6pfa4w8rS3sNGhR84XviliZWRVMzbeMjEGyyQKtYDX5+UuDQFswK/ev0wzA+
llyGxZvnqLqrqjjCyjAUGEalY+UN1zd/RanQbqjssMok4jXAO31KhaRa62afvWxs
Rp5HlaV0B3gY3sLZi1caBVETUTFEbBNyB2vHCC+40cQ8Sr8OtTQIAg/jvR2fup87
0ycG0jCYX91mMe7heL+F1kKrNptIDH7jM4gR4lVhV95a2Y/g+gWiH71qZhS7TKGb
xgYAuwVXhcQroL76DIexHY5XWXRvm1RFNbPwD/oXXxeFzD3wxx7wbDJ9lm0DTcxP
P7ltbfBlq+V6BEBRl1SbAgMBAAGjgY8wgYwwDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQU3/Jnd/31bELE1dUX
LgKixeYLJukwHwYDVR0jBBgwFoAUH4t1gs0rLiYikMF6rDJE57imMfswGwYDVR0R
BBQwEoIQdGVzdC5leGFtcGxlLmNvbTANBgkqhkiG9w0BAQsFAAOCAQEAVyEj+rc+
+Q7hR2vlzExv4FYjxkdt4n0imopco9VEj5a3O+ImOQF/sXCbs4quC8EfiaWUugk/
UcDu9nvF2nHFW+WvEybVo7MhII0uklJcVjYNaNMljgVxSKEBr6p9CtsbtLhwlxzU
OxicGpBOxb8Rjwwe48cu3Wp4fjULHflmBf31Yv/f1Cf0I4fWRuoY0N7wPaeAqGEt
HohIS55wrTbQ8HsKVBnkYW9JG0WtseMjJRA1AkxUuYCUYMhqSUB5NIqhCBuE/8Di
YyVm5/azPmKokkOTeRfvc3yjc6bD5cXdOUb1uJmPJ9jpGzWVIvg02JXSoEcGVKZ2
LaRvGx7nqQKrVA==
-----END CERTIFICATE-----
expiration          1649333247
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDpjCCAo6gAwIBAgIUbWftbElyhjmdaxqIVw73JyUiybcwDQYJKoZIhvcNAQEL
BQAwFjEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMjIwMzA4MTE0ODU5WhcNMjcw
MzA3MTE0OTI5WjAtMSswKQYDVQQDEyJleGFtcGxlLmNvbSBJbnRlcm1lZGlhdGUg
QXV0aG9yaXR5MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1//Fxk6t
5k+vwiOkmm9Au/q+rRpGMGGCUvGxpxIsrV+X7jF5ZKp0O0s8R7XR6F1fjN+92i2S
f1mePzuLJuJnHzPFqxbcol4Y7uu7jD/HeTEBuVGuWp1dD3X+qSrxfE58rdof8GPU
ayF+fQFSq4fOne5MP2we7gb8sEp0ya8rAcql72yhDf920semoT+qTO0Buoa2UEUd
+sNrMiUcG2c1HDiEQMq5B8k3Ss0DmIf+UfPK69/6EtE0jWDlx6KqnBJIyO1tXYne
cX9HC4D1nQMiww96Obf4zivNoTkklTD/7sWuwBJzI3u0zokR6x9kNhEVn/nK3h+U
JJjED4PJ0XLjTwIDAQABo4HUMIHRMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8E
BTADAQH/MB0GA1UdDgQWBBQfi3WCzSsuJiKQwXqsMkTnuKYx+zAfBgNVHSMEGDAW
gBRXSxpw8FLbT8gZXuCyFgy0SmUH8jA7BggrBgEFBQcBAQQvMC0wKwYIKwYBBQUH
MAKGH2h0dHA6Ly8xMjcuMC4wLjE6ODIwMS92MS9wa2kvY2EwMQYDVR0fBCowKDAm
oCSgIoYgaHR0cDovLzEyNy4wLjAuMTo4MjAxL3YxL3BraS9jcmwwDQYJKoZIhvcN
AQELBQADggEBAHLrlI0V/6JWEhofah4J64Dm7PzS1k9AMLa1n+QjimDzEaRuk/ju
9ZNDgTI/icKf+Rzc+Lal6bHKg/AoYqZjA00s4OyPtshcNzlzMqkceoK+fzr631ss
Rd8prdIgQDVygBZqqQaDGwxqTzaciYq1dj5P4/4YaZiLHJv1XD9xAYn9VQKR783U
wRHuGKuHr7kpqK/zvoKu9htZ/xJ+yjbS8SmpZp8zZWmuJY0tH7ideFeoCiv8NhSP
LQavewgDBM8EH0wNN9S8lZRTGHpfmUVDxtDGEQXffLoZMt7hnVPQhF2PJs3pjxQ9
qPuna5NanXpcSXnkvHyUAeF9eHmxzS4TrkI=
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA8rghsuqX2uMPK0t7DRoUfOF74pYmVkVTM23jIxBsskCrWA1+
flLg0BbMCv3r9MMwPpZchsWb56i6q6o4wsowFBhGpWPlDdc3f0Wp0G6o7LDKJOI1
wDt9SoWkWutmn71sbEaeR5WldAd4GN7C2YtXGgVRE1ExRGwTcgdrxwgvuNHEPEq/
DrU0CAIP470dn7qfO9MnBtIwmF/dZjHu4Xi/hdZCqzabSAx+4zOIEeJVYVfeWtmP
4PoFoh+9amYUu0yhm8YGALsFV4XEK6C++gyHsR2OV1l0b5tURTWz8A/6F18Xhcw9
8Mce8GwyfZZtA03MTz+5bW3wZavlegRAUZdUmwIDAQABAoIBAQDVZulSGoqbS+mp
Yjr1YQnEZqf6p5NbZKZrYGFFEgrPiqZby7l8rbe1nWWXAHw1/mtri94cLKAPxT0i
843mTciJ0nJCthrM2X2U7BYLCVCCkkmYtj8WHffhzrCbs77OEcjDRi+htsP6yRph
m3HrvWQ+n5ASEN/lTbSnHDk5jEwRDL9DvAuNCDYq+X7wx4pyQdmw5T+KhOsZU/kd
XCXkaYahluaqdHGRdS0ybEceVZ6dAwnfIzAw4vzckzkWCrP8idzW4N8olOOzDkFc
BszFTc0gzHGdut0pkx6bZO8BZZKORwOgHIXmM92gp6sISQ3zunPyba6v/EtBHCMy
Nl4PpgMhAoGBAPYxEkB7TOMUtCqjmrYzube8axLndFQtUH/RmwWbGU7d3xgCrNu6
D1KCADtfhpViMTTwKnCvHs2lb+UsO5BFQMI9NHu5ZjlTSHlw/NPjmav7RzPM1jk9
k1u/Bd3n8LwEfa2KSMi5sRLUH8WbU6Lu3lba6lTc3wt4swOrTOcWxtARAoGBAPxj
pRSLAvhVP3KkZwEwXuEPqA2noWcL68o14E46CaT81ZvdSGYN69Wk6nK85wKz8KM2
uZeWCWqTM1YG469y5adXk6jCQ5RqM9OsZnz/nClIpwEwzi/FXykRkbbS6GMa+QqX
DBjOPtHikA4uSBxe7Ac18K/Sus02Z+lNDMPcpgXrAoGAGGBJd1r+BOny4NnsIqhC
dj/QtkF2kjMeall9qPNvcRCk20LAjM0CYV//1+EgeUAMsToqaZfdS+1ag+16RFGn
FNRskueLiJeHcIbQccZyW8X4idd5SpefNJFYal1hx6wGKZ2skTtZngykzLMRd+uA
1ka8twRbjU45I17dsW/e5ZECgYACpkklXhA56ysvWOp76LYWRsPM4OWg308qYEZq
i9FuGflkGX6ZE6xWIdUUP0dtj9YG4JwyKzEOHV18mbXqMZqXlyPFscRj2nmQz06E
IWrSRCD8sjZ36eqF09ljwfMOl0nmAo/t8rxWF7rLa1D2e0ifCsHsFpo/bULKADUS
GU7k4QKBgQDwktB0xRY0AsT8nVgWcIgrG/uvBqkEWOFYJw2A4i8HmGWgnrRFyrkz
BKyI/l/QvetFIfYc8AweNwoLR0dcTS+DRJuRTwY0x4PKhrCMVhJ062vWOQDsuftO
9G1q8vx/6mJBn1GNiboOCGmt8xwiZquzqUWsZNSp7mmpgjLGl8a1HA==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       33:20:e7:96:6e:32:6c:10:92:51:5b:fd:e1:5f:1d:62:df:bc:5d:53
```

## 5. Установите корневой сертификат созданного центра сертификации в доверенные в хостовой системе.

### Создаем в Windows папку vagrantshare, настроим ее в virtualbox, положим туда CA_cert.crt:
```
vagrant@sandbox:~$ mkdir ~/vagrantshare
vagrant@sandbox:~$ sudo mount -t vboxsf vagrantshare ~/vagrantshare  
vagrant@sandbox:~$ cp CA_cert.crt ~/vagrantshare  
vagrant@sandbox:~$ sudo umount vagrantshare  
```

### Устанавливаем корневой сертификат CA_cert.crt в Доверенные корневые центры сертификации на хостовой Windows-машине:
[Картинка CA_cert.png]

## 6. Установите nginx.
```
vagrant@sandbox:~$ sudo apt update
vagrant@sandbox:~$ sudo apt install nginx
vagrant@sandbox:~$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-03-08 16:14:53 MSK; 1min 53s ago
       Docs: man:nginx(8)
   Main PID: 2380 (nginx)
      Tasks: 3 (limit: 4617)
     Memory: 5.3M
     CGroup: /system.slice/nginx.service
             ├─2380 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;             ├─2381 nginx: worker process
             └─2382 nginx: worker process
```

### Проверяем, какие порты слушает nginx:
```
vagrant@sandbox:~$ sudo lsof -nP -i | grep LISTEN
systemd-r  686 systemd-resolve   13u  IPv4  19968      0t0  TCP 127.0.0.53:53 (LISTEN)
sshd       755            root    3u  IPv4  23820      0t0  TCP *:22 (LISTEN)
sshd       755            root    4u  IPv6  23822      0t0  TCP *:22 (LISTEN)
vault     1295            root    8u  IPv4  26599      0t0  TCP 127.0.0.1:8201 (LISTEN)
vault     1295            root    9u  IPv4  27938      0t0  TCP *:8200 (LISTEN)
nginx     2380            root    6u  IPv4  31584      0t0  TCP *:80 (LISTEN)
nginx     2380            root    7u  IPv6  31585      0t0  TCP *:80 (LISTEN)
nginx     2381        www-data    6u  IPv4  31584      0t0  TCP *:80 (LISTEN)
nginx     2381        www-data    7u  IPv6  31585      0t0  TCP *:80 (LISTEN)
nginx     2382        www-data    6u  IPv4  31584      0t0  TCP *:80 (LISTEN)
nginx     2382        www-data    7u  IPv6  31585      0t0  TCP *:80 (LISTEN)
```

### Получаем список профилей приложений, известных ufw и разрешаем профиль Nginx HTTP - трафик на порту 80:
```
vagrant@sandbox:~$ sudo ufw app list
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH

vagrant@sandbox:~$ sudo ufw allow 'Nginx HTTP'
Rule added
Rule added (v6)

vagrant@sandbox:~$ sudo ufw status
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
Anywhere                   ALLOW       127.0.0.1
Nginx HTTP                 ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

### Получаем IP адрес bridged интерфейса, видим 192.168.1.19:
```
vagrant@sandbox:~$ hostname -I
10.0.2.15 192.168.1.19
```

### Проверяем nginx в браузере хостовой Windows-машины:
[Картинка nginx_welcome1.png]

### Для подключения по имени test.example.com добавляем в файл %SystemRoot%/system32/drivers/etc/hosts запись:
```
192.168.1.19 test.example.com
```

### Проверяем nginx в браузере хостовой Windows-машины:
[Картинка nginx_welcome2.png]

## 7. По инструкции (ссылка) настройте nginx на https, используя ранее подготовленный сертификат.

### Создаем папку ~/vault_nginx_cert. Запрашиваем у vault сертификат для домена test.example.com в формате json. При помощи jq выгружаем сертификат сайта и промежуточного ЦС в файл test.example.com.crt, ключ в файл test.example.com.key в нашу папку.
```
vagrant@sandbox:~$ mkdir vault_nginx_cert

vagrant@sandbox:~$ vault write -format=json pki_int/issue/example-dot-com common_name="test.example.com" ttl="720h" > ~/vault_nginx_cert/test.example.com.crt

vagrant@sandbox:~$ cat ~/vault_nginx_cert/test.example.com.crt | jq -r '.data.certificate' > ~/vault_nginx_cert/test.example.com.crt.pem
vagrant@sandbox:~$ cat ~/vault_nginx_cert/test.example.com.crt | jq -r '.data.issuing_ca' >> ~/vault_nginx_cert/test.example.com.crt.pem
vagrant@sandbox:~$ cat ~/vault_nginx_cert/test.example.com.crt | jq -r '.data.private_key' > ~/vault_nginx_cert/test.example.com.crt.key
```

### Настраиваем nginx на https, 443 порт. Добавляем в конфиг-файл /etc/nginx/nginx.conf в блок http следующие строки:
```
server {
    listen              443 ssl;
    server_name         test.example.com;
    ssl_certificate     /home/vagrant/vault_nginx_cert/test.example.com.crt.pem;
    ssl_certificate_key /home/vagrant/vault_nginx_cert/test.example.com.crt.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
}
```

### Проверяем синтаксис и перезапускаем nginx:
```
vagrant@sandbox:~/vault_nginx_cert$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

vagrant@sandbox:~/vault_nginx_cert$ sudo systemctl restart nginx
```

## 8. Откройте в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.

### Страница https://test.example.com/ открывается корректно, цепочка сертификатов видна.
[Картинка nginx_welcome_https.png]

## 9. Создайте скрипт, который будет генерировать новый сертификат в vault.

### Настроим автозапуск vault и автоматизируем его распечатывание.

#### Сделаем скрипт распечатывания:
```
vagrant@sandbox:~$ mkdir ~/scripts
vagrant@sandbox:~$ nano ~/scripts/unseal_vault.sh                         

#!/usr/bin/env bash

sleep 10
export VAULT_ADDR=http://127.0.0.1:8201
vault operator unseal EZYRI+mOkZyI9FsC8Z+ijRhlsz/upa0uW24b1xOk0tK6 &&
vault operator unseal SFz3wTVzhEv9G+HKbk9YQNwazwESLXTBFztiO2lgZWqb &&
vault operator unseal McwCdCjuHVl44Upps5+D3rcnBiZBNpP9SsrdnJ8bEWZA
echo Vault Unsealed.

vagrant@sandbox:~$ sudo chmod +x ~/scripts/unseal_vault.sh
```

#### Чтобы системная переменная VAULT_ADDR создавалась каждый раз при входе пользователя в систему, дописываем файл /etc/environment:
```
vagrant@sandbox:~$ sudo -i
root@sandbox:~# echo VAULT_ADDR=http://127.0.0.1:8201 >> /etc/environment
root@sandbox:~# cat /etc/environment
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
VAULT_ADDR=http://127.0.0.1:8201
```

#### Дописываем ExecStartPost=/home/vagrant/scripts/unseal_vault.sh unit-файл vault по пути /lib/systemd/system/vault.service:
```
root@sandbox:~# nano /lib/systemd/system/vault.service

[Unit]
Description="HashiCorp Vault - A tool for managing secrets"
Documentation=https://www.vaultproject.io/docs/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/vault.d/vault.hcl
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
EnvironmentFile=/etc/vault.d/vault.env
User=vault
Group=vault
ProtectSystem=full
ProtectHome=read-only
PrivateTmp=yes
PrivateDevices=yes
SecureBits=keep-caps
AmbientCapabilities=CAP_IPC_LOCK
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
NoNewPrivileges=yes
ExecStart=/usr/bin/vault server -config=/etc/vault.d/vault.hcl
ExecStartPost=/home/vagrant/scripts/unseal_vault.sh
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
KillSignal=SIGINT
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
LimitNOFILE=65536
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
```

#### Разрешаем автозапуск vault:
```
root@sandbox:~# systemctl enable vault
Created symlink /etc/systemd/system/multi-user.target.wants/vault.service → /lib/systemd/system/vault.service.
```

### Сделаем скрипт генерации сертификата в vault.
```
vagrant@sandbox:~$ nano ~/scripts/regen_vault_certs.sh
#!/usr/bin/env bash

export VAULT_ADDR=http://127.0.0.1:8201

vault write -format=json pki_int/issue/example-dot-com common_name="test.example.com" ttl="720h" > /home/vagrant/vault_nginx_cert/test.example.com.crt
cat /home/vagrant/vault_nginx_cert/test.example.com.crt | jq -r '.data.certificate' > /home/vagrant/vault_nginx_cert/test.example.com.crt.pem
cat /home/vagrant/vault_nginx_cert/test.example.com.crt | jq -r '.data.issuing_ca' >> /home/vagrant/vault_nginx_cert/test.example.com.crt.pem
cat /home/vagrant/vault_nginx_cert/test.example.com.crt | jq -r '.data.private_key' > /home/vagrant/vault_nginx_cert/test.example.com.crt.key
systemctl restart nginx

vagrant@sandbox:~$ sudo chmod +x ~/scripts/regen_vault_certs.sh
```

### Запускаем скрипт regen_vault_certs.sh вручную, проверяем страничку и сертификат:
[Картинка nginx_welcome_regen.png]

## 10. Поместите скрипт в crontab, чтобы сертификат обновлялся какого-то числа каждого месяца в удобное для вас время.

### Настраиваем cron:
```
vagrant@sandbox:~$ sudo nano /etc/crontab
11 23   8 * *   root    /home/vagrant/scripts/regen_vault_certs.sh >> /var/log/regen_vault_certs.log
```

### Перезапускаем и проверяем cron:
```
vagrant@sandbox:~$ sudo systemctl restart cron
vagrant@sandbox:~$ sudo systemctl status cron
● cron.service - Regular background program processing daemon
     Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-03-08 23:07:04 MSK; 1min 35s ago
       Docs: man:cron(8)
   Main PID: 1906 (cron)
      Tasks: 1 (limit: 4617)
     Memory: 412.0K
     CGroup: /system.slice/cron.service
             └─1906 /usr/sbin/cron -f

Mar 08 23:07:04 sandbox systemd[1]: Started Regular background program processing daemon.
Mar 08 23:07:04 sandbox cron[1906]: (CRON) INFO (pidfile fd = 3)
Mar 08 23:07:04 sandbox cron[1906]: (CRON) INFO (Skipping @reboot jobs -- not system startup)
```

### Удаляем готовые сертификаты:
```
vagrant@sandbox:~/vault_nginx_cert$ ll
total 24
drwxrwxr-x 2 vagrant vagrant 4096 Mar  8 19:40 ./
drwxr-xr-x 8 vagrant vagrant 4096 Mar  8 20:29 ../
-rw-rw-r-- 1 vagrant vagrant 6057 Mar  8 22:46 test.example.com.crt
-rw-rw-r-- 1 vagrant vagrant 1675 Mar  8 22:46 test.example.com.crt.key
-rw-rw-r-- 1 vagrant vagrant 2567 Mar  8 22:46 test.example.com.crt.pem
vagrant@sandbox:~/vault_nginx_cert$ rm test*
```

### Проверяем cron после назначенного времени:
```
vagrant@sandbox:~$ sudo systemctl status cron
● cron.service - Regular background program processing daemon
     Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-03-08 23:07:04 MSK; 4min 53s ago
       Docs: man:cron(8)
   Main PID: 1906 (cron)
      Tasks: 1 (limit: 4617)
     Memory: 1.2M
     CGroup: /system.slice/cron.service
             └─1906 /usr/sbin/cron -f

Mar 08 23:07:04 sandbox systemd[1]: Started Regular background program processing daemon.
Mar 08 23:07:04 sandbox cron[1906]: (CRON) INFO (pidfile fd = 3)
Mar 08 23:07:04 sandbox cron[1906]: (CRON) INFO (Skipping @reboot jobs -- not system startup)
Mar 08 23:11:01 sandbox CRON[1933]: pam_unix(cron:session): session opened for user root by (uid=0)
Mar 08 23:11:01 sandbox CRON[1942]: (root) CMD (/home/vagrant/scripts/regen_vault_certs.sh >> /var/log/regen_vault_certs.log)
Mar 08 23:11:02 sandbox sudo[1955]:     root : TTY=unknown ; PWD=/root ; USER=root ; COMMAND=/usr/bin/systemctl restart nginx
Mar 08 23:11:02 sandbox sudo[1955]: pam_unix(sudo:session): session opened for user root by (uid=0)
Mar 08 23:11:02 sandbox sudo[1955]: pam_unix(sudo:session): session closed for user root
Mar 08 23:11:02 sandbox CRON[1933]: (CRON) info (No MTA installed, discarding output)
Mar 08 23:11:02 sandbox CRON[1933]: pam_unix(cron:session): session closed for user root
```

### Проверяем сертификаты - создались:
```
vagrant@sandbox:~$ ll ~/vault_nginx_cert
total 8
drwxrwxr-x 2 vagrant vagrant 4096 Mar  8 23:11 ./
drwxr-xr-x 8 vagrant vagrant 4096 Mar  8 20:29 ../
-rw-r--r-- 1 root    root    6061 Mar  8 23:11 test.example.com.crt
-rw-r--r-- 1 root    root    1679 Mar  8 23:11 test.example.com.crt.key
-rw-r--r-- 1 root    root    2567 Mar  8 23:11 test.example.com.crt.pem
```

### Проверяем nginx:
```
vagrant@sandbox:~/vault_nginx_cert$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-03-08 23:11:02 MSK; 3s ago
       Docs: man:nginx(8)
    Process: 1815 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (co>
    Process: 1829 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited>
   Main PID: 1831 (nginx)
      Tasks: 3 (limit: 4617)
     Memory: 3.5M
     CGroup: /system.slice/nginx.service
             ├─1831 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─1832 nginx: worker process
             └─1833 nginx: worker process

Mar 08 23:11:02 sandbox systemd[1]: Starting A high performance web server and a reverse >
Mar 08 23:11:02 sandbox systemd[1]: Started A high performance web server and a reverse p
```

### Проверяем страничку и сертификат:
[Картинка cron_cert-regen.png]

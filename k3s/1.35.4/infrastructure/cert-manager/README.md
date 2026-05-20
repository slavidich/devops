# cert-manager + Let's Encrypt (DNS-01) через DuckDNS

Автоматическая выдача сертов в k3s с проверкой домена dns01 через API DuckDNS. Так как домен `slavidich.duckdns.org` указывает на локалку `192.168.0.25`, вариант с http01 нам не подходит (моя локалка это моя крепость (с) Цезарь)

Если вкратце, то при http01 CA просит разместить файл по адресу на веб-сервере, потом CA идет и проверяет его. Если ок - выдает серт. Так как у меня это все в локалке и выносить наружу я не хочу, придется делать через dns01. При dns01 проверка идет через днс запись, СА говорит создать txt запись по адресу типа `acme-challenge.slavidich.duckdns.org` (а эту запись можно сделать только с токеном от duckdns), lets encrypt проверяет, что этот домен ваш и выдает серт

## Схема

```text
Certificate (wildcard)
    → cert-manager
        → ACME DNS-01 Challenge (_acme-challenge.slavidich.duckdns.org)
            → webhook cert-manager-webhook-duckdns
                → DuckDNS API (TXT)
            → Let's Encrypt выдаёт wildcard-сертификат
        → Secret slavidich-wildcard-tls
            → Ingress whoami, echo, … (общий secretName)

cert-manager Controller (фон)
    → следит за сроком действия
    → за ~30 дней до истечения — повторный DNS-01 и обновление Secret
```

---

## 1. Репозитории Helm

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add mmontes https://mmontes11.github.io/charts
helm repo update
```

---

## 2. Установка cert-manager (jetstack)

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \
  --set crds.enabled=true \
  --set 'extraArgs={--dns01-recursive-nameservers-only=true,--dns01-recursive-nameservers=8.8.8.8:53\,1.1.1.1:53}'
```

Параметр `extraArgs` помогает cert-manager проверять TXT-записи через публичные DNS (удобно дома, без своего резолвера).

---

## 3. Webhook DuckDNS (mmontes)

```bash
helm install cert-manager-webhook-duckdns mmontes/cert-manager-webhook-duckdns \
  --namespace cert-manager \
  --set duckdns.token='token' \
  --set clusterIssuer.email='@mail' \
  --set clusterIssuer.production.create=true \
  --set clusterIssuer.staging.create=false \
  --set logLevel=2
```

---

## 4. Wildcard-сертификат (один на все сервисы)

Манифест: [`wildcard-certificate.yaml`](wildcard-certificate.yaml)

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f infrastructure/cert-manager/wildcard-certificate.yaml
```

cert-manager создаст `CertificateRequest` → `Order` → `Challenge`, пройдёт DNS-01 и положит сертификат в Secret `slavidich-wildcard-tls`.

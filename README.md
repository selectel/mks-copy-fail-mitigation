# Copy Fail Mitigation DaemonSets

DaemonSet'ы для исправления уязвимостей семейства Copy Fail на всех worker нодах кластеров MKS.

## Содержание

- [CVE-2026-31431 (Copy Fail)](#cve-2026-31431-copy-fail)
- [CVE-2026-43284, CVE-2026-43500, CopyFail2 (Dirty Flag)](#cve-2026-43284-cve-2026-43500-copyfail2-dirty-flag)

---

## CVE-2026-31431 (Copy Fail)

## Описание уязвимости

**CVE ID:** CVE-2026-31431

**CVE Link:** https://nvd.nist.gov/vuln/detail/CVE-2026-31431

**Вектор атаки и уровень опасности согласно CVSS v.3.1:**

Score: **7.8 HIGH**

Vector: `CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`

**Краткое описание:**

Copy Fail (CVE-2026-31431) — это логическая уязвимость в подсистеме криптографического API ядра Linux, позволяющая обычному пользователю системы получить права суперпользователя (root). PoC работает на всех основных дистрибутивах Linux, выпущенных с 2017 года и до момента выхода патча.

**Особенности:**
- возможность использования как примитив побега из контейнера на хост, из-за использования общего для всего хоста page cache
- не требует удалённого доступа — использование возможно только в случае наличия локальной непривилегированной учетной записи
- использует крипто-API ядра (AF_ALG), который включён по-умолчанию в конфигурациях практически всех популярных дистрибутивов

## Что делает этот DaemonSet

DaemonSet запускает контейнер на каждой worker ноде кластера:

1. Тестирует доступность AF_ALG AEAD интерфейса
2. Cоздает конфигурацию `/etc/modprobe.d/disable-algif.conf`
3. Выполняет `rmmod algif_aead` если модуль загружен
4. Проверяет что уязвимость устранена, вызывая повторную проверку из пункта 1

## Использование

### 1. Скачать DaemonSet

```sh
wget https://raw.githubusercontent.com/selectel/mks-copy-fail-mitigation/refs/heads/main/copy-fail-mitigation-daemonset.yaml
```

Или клонировать репозиторий:

```sh
git clone https://github.com/selectel/mks-copy-fail-mitigation.git
cd mks-copy-fail-mitigation
```

### 2. Применить DaemonSet

```sh
kubectl apply -f copy-fail-mitigation-daemonset.yaml
```

### 3. Проверить статус выполнения

```sh
# Проверить статус DaemonSet
kubectl get daemonset -n kube-system cve-2026-31431-mitigation

# Получить список подов
kubectl -n kube-system get pods -l app=cve-2026-31431-mitigation -o wide
```

### 4. Просмотреть логи выполнения

```sh
# Логи initContainer
kubectl -n kube-system logs -l app=cve-2026-31431-mitigation -c mitigation
```

### 5. Для удаления DaemonSet

```sh
kubectl delete -f copy-fail-mitigation-daemonset.yaml
```

---

## CVE-2026-43284, CVE-2026-43500, CopyFail2 (Dirty Flag)

**CVE ID:** CVE-2026-43284, CVE-2026-43500

**CopyFail2:** без отдельного CVE ID

**CVE Links:**
- https://nvd.nist.gov/vuln/detail/CVE-2026-43284
- https://nvd.nist.gov/vuln/detail/CVE-2026-43500

**Краткое описание:**

Dirty Flag / CopyFail2 — уязвимости в подсистеме IPsec ядра Linux (модули esp4, esp6 и rxrpc), позволяющие локальному непривилегированному пользователю получить права суперпользователя (root).

**Особенности:**
- возможность использования как примитив побега из контейнера на хост
- используют модули ядра esp4 (IPsec IPv4), esp6 (IPsec IPv6) и rxrpc (протокол RxRPC), которые включены по умолчанию в большинстве дистрибутивов

## Что делает этот DaemonSet

DaemonSet запускает контейнер на каждой worker ноде кластера:

1. Проверяет через `lsmod` загружены ли модули `esp4`, `esp6`, `rxrpc`
2. Создает конфигурацию `/etc/modprobe.d/disable-esp-rxrpc.conf` с правилами `install esp4/esp6/rxrpc /bin/false`
3. Выполняет `rmmod` для каждого модуля если он загружен
4. Проверяет что модули больше не загружены через повторный `lsmod`

## Использование

### 1. Скачать DaemonSet

```sh
wget https://raw.githubusercontent.com/selectel/mks-copy-fail-mitigation/refs/heads/main/dirty-flag-copyfail2-mitigation-daemonset.yaml
```

Или клонировать репозиторий:

```sh
git clone https://github.com/selectel/mks-copy-fail-mitigation.git
cd mks-copy-fail-mitigation
```

### 2. Применить DaemonSet

```sh
kubectl apply -f dirty-flag-copyfail2-mitigation-daemonset.yaml
```

### 3. Проверить статус выполнения

```sh
# Проверить статус DaemonSet
kubectl get daemonset -n kube-system dirty-flag-copyfail2-mitigation

# Получить список подов
kubectl -n kube-system get pods -l app=dirty-flag-copyfail2-mitigation -o wide
```

### 4. Просмотреть логи выполнения

```sh
# Логи initContainer
kubectl -n kube-system logs -l app=dirty-flag-copyfail2-mitigation -c mitigation
```

### 5. Для удаления DaemonSet

```sh
kubectl delete -f dirty-flag-copyfail2-mitigation-daemonset.yaml
```
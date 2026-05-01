# CVE-2026-31431 Mitigation DaemonSet

DaemonSet для исправления CVE-2026-31431 на всех worker нодах кластеров MKS.

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
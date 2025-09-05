# Домашнее задание к занятию «Микросервисы: подходы»

> Репозиторий: hw-33\
> Выполнил: Асадбек Асадбеков\
> Дата: сентябрь 2025

## Задача 1: Обеспечить разработку

**Выбранное решение:** **GitLab Ultimate** (облачная SaaS-версия или self-managed) в связке с **Kubernetes** и **Docker Registry**.

**Обоснование выбора:**

GitLab представляет собой единую платформу "все-в-одном" (DevSecOps platform), которая полностью покрывает все поставленные требования без необходимости интеграции и поддержки множества разнородных инструментов.

**Соответствие требованиям:**

| Требование | Реализация в GitLab |
| :--- | :--- |
| **Облачная система** | GitLab.com (SaaS) или развертывание GitLab Self-Managed в любом облаке (AWS, GCP, Azure). |
| **Система контроля версий Git** | Встроенная базовая функциональность. |
| **Репозиторий на каждый сервис** | Создание отдельного проекта (репозитория) для каждого микросервиса. |
| **Запуск сборки по событию из СКВ** | Запуск пайплайна по событию `push`, `merge request`, созданию тега и др. |
| **Запуск сборки по кнопке с параметрами** | Manual jobs (ручные задания) с возможностью передачи переменных пайплайна. |
| **Настройки для каждой сборки** | Файл `.gitlab-ci.yml` + переменные CI/CD (на уровне проекта, группы, пайплайна). |
| **Создание шаблонов для сборки** | Ключевые слова `include`, `extends`, а также anchors & aliases в YAML. |
| **Безопасное хранение секретов** | Masked и protected CI/CD Variables. Интеграция с HashiCorp Vault. |
| **Несколько конфигураций из одного репозитория** | Правила (`rules`) и `only`/`except` для запуска разных jobs в зависимости от условий. |
| **Кастомные шаги при сборке** | Любые shell-команды или скрипты в секции `script` job'а. |
| **Собственные докер-образы для сборки** | Ключевое слово `image` в конфигурации job'а. Использование GitLab Container Registry. |
| **Развертывание агентов на своих серверах** | Регистрация собственных GitLab Runner'ов в Docker, Kubernetes, на виртуальных машинах. |
| **Параллельный запуск сборок** | Запуск нескольких пайплайнов на разных раннерах. Ключевое слово `parallel` для job'ов. |
| **Параллельный запуск тестов** | Разделение тестов на группы и их параллельный запуск через `parallel:matrix`. |

**Альтернативы:** Jenkins + GitHub/GitLab, GitHub Actions + Self-Hosted Runners, CircleCI.

---

## Задача 2: Логи

**Выбранное решение:** Стек **Elasticsearch + Vector + Kibana**.

**Обоснование выбора:**

Данный стек является современным, высокопроизводительным и отраслевым стандартом для построения централизованной системы логирования. Vector, в отличие от более старого Fluentd, предлагает лучшую производительность и более простую конфигурацию при сохранении всех ключевых возможностей.

**Соответствие требованиям:**

| Требование | Реализация в Elasticsearch + Vector + Kibana |
| :--- | :--- |
| **Сбор логов со всех хостов** | Агент Vector развертывается на каждом хосте как DaemonSet (K8s) или системный сервис. |
| **Сбор логов из stdout** | Vector читает логи контейнеров Docker из `/var/lib/docker/containers/.../*.log`. |
| **Гарантированная доставка логов** | Disk-buffering в Vector: логи буферизируются на диске при недоступности Elasticsearch. |
| **Поиск и фильтрация по логам** | Мощный движок полнотекстового поиска и анализа в Elasticsearch. |
| **UI для разработчиков** | Веб-интерфейс Kibana (раздел Discover) с возможностью сохранения поисковых запросов. |
| **Ссылка на сохраненный поиск** | Любой поиск в Kibana имеет уникальный URL, которым можно поделиться. |

**Альтернативы:** Grafana Loki + Promtail, Splunk, Datadog, New Relic.

---

## Задача 3: Мониторинг

**Выбранное решение:** Стек **Prometheus + Grafana**.

**Обоснование выбора:**

Prometheus и Grafana стали отраслевым стандартом для мониторинга микросервисных архитектур. Prometheus обеспечивает сбор и хранение метрик, а Grafana — мощную визуализацию и анализ. Данный стек полностью соответствует всем требованиям и обладает широкими возможностями для расширения и интеграции.

**Соответствие требованиям:**

| Требование | Реализация в Prometheus + Grafana |
| :--- | :--- |
| **Сбор метрик со всех хостов** | Prometheus использует механизм pull-модели, периодически опрашивая настроенные цели (targets). Для автоматического обнаружения сервисов в Kubernetes используется service discovery. |
| **Сбор метрик состояния ресурсов хостов** | **Node Exporter** устанавливается на каждом хосте и предоставляет метрики CPU, RAM, Disk, Network. |
| **Сбор метрик потребляемых ресурсов для каждого сервиса** | В Kubernetes **cAdvisor** (часть kubelet) собирает метрики контейнеров. **kube-state-metrics** предоставляет метрики о состоянии объектов Kubernetes (поды, деплойменты и т.д.). |
| **Сбор специфичных метрик для каждого сервиса** | Разработчики используют клиентские библиотеки Prometheus для добавления кастомных метрик в свои сервисы. |
| **UI с запросами и агрегацией** | Grafana позволяет создавать дашборды с запросами на языке PromQL, который поддерживает агрегацию, математические операции и т.д. |
| **UI с настраиваемыми панелями** | Grafana предоставляет интуитивный интерфейс для создания и настройки дашбордов с различными типами панелей (графики, таблицы, heatmaps и др.). |

**Альтернативы:** VictoriaMetrics, Thanos (для масштабирования Prometheus), коммерческие решения (Datadog, New Relic).

---

## Задача 4: Логи (Практическая реализация)

**Реализация:** Docker Compose файл для развертывания стека Elasticsearch + Vector + Kibana.

**Результат:** 
- Kibana доступна по адресу: http://localhost:8081
- Логин: `elastic` (или через анонимный доступ при отключенной безопасности)
- Пароль: `qwerty123456`

**Особенности реализации:**
- Использованы официальные образы Elasticsearch 8.9.0 и Kibana 8.9.0
- Vector настроен для сбора логов Docker контейнеров
- Отключена security для упрощения тестирования

---

## Задача 5: Мониторинг (Практическая реализация)

**Реализация:** Docker Compose файл для развертывания стека Prometheus + Grafana с тестовыми сервисами.

**Результат:**
- Grafana доступна по адресу: http://localhost:8082
- Логин: `admin`
- Пароль: `qwerty123456`
- Prometheus доступен по адресу: http://localhost:9090

**Особенности реализации:**
- Использованы образы prom/node-exporter для тестовых сервисов
- Настроены два тестовых сервиса: security-service и uploader-service
- Создан дашборд в Grafana для отображения метрик
- Автоматическая настройка источников данных через provisioning

---

## YAML

<details>
<summary>docker-compose.yml</summary>

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=qwerty123456
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana-dashboards:/var/lib/grafana/dashboards
      - ./grafana-provisioning:/etc/grafana/provisioning
    ports:
      - "8082:3000"
    depends_on:
      - prometheus
    networks:
      - monitoring
    restart: unless-stopped

  security-service:
    image: prom/node-exporter:latest
    ports:
      - "9999:9100"
    networks:
      - monitoring
    restart: unless-stopped

  uploader-service:
    image: prom/node-exporter:latest
    ports:
      - "8888:9100"
    networks:
      - monitoring
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:

networks:
  monitoring:
    driver: bridge
```
</details>

---
<details>
<summary>prometheus.yml</summary>

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'security-service'
    static_configs:
      - targets: ['security-service:9100']
    metrics_path: /metrics
    scrape_interval: 10s

  - job_name: 'uploader-service'
    static_configs:
      - targets: ['uploader-service:9100']
    metrics_path: /metrics
    scrape_interval: 10s
```
</details>

---

## Скриншоты

### Задача 4: Kibana

![Kibana Interface](https://github.com/asad-bekov/hw-33/blob/main/img/1.PNG)

### Задача 5: Grafana Dashboard

![Grafana Dashboard](https://github.com/asad-bekov/hw-33/blob/main/img/2.PNG)

![Prometheus](https://github.com/asad-bekov/hw-33/blob/main/img/3.PNG)

---

## Вывод

Все поставленные задачи успешно выполнены. Развернуты и настроены:
1. **Стек логирования** (Elasticsearch + Vector + Kibana) для сбора и анализа логов
2. **Стек мониторинга** (Prometheus + Grafana) для сбора метрик и визуализации
3. **Тестовая инфраструктура** с демонстрационными сервисами

Решение полностью соответствует требованиям задания и готово к использованию в production-среде после дополнительной настройки безопасности и отказоустойчивости.
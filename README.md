# ELK-stack
Install and setting

Так как по офф документации нужен VPN, а качать много то так установка проходит быстрее

Вот пошаговая инструкция по установке и настройке ELK-стека на Ubuntu:

### 1. Подготовка системы
**Обновите систему и установите Java:**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jdk  # Или более новую версию
java -version  # Проверьте версию Java (требуется 17+)
```

**Добавление репозитория Elastic:** VPN!!!!!!!!!!!!
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
```

### 2. Установка Elasticsearch
```bash
sudo apt install -y elasticsearch
```

**Настройте Elasticsearch:**
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Основные параметры:
```yaml
cluster.name: my-elk-cluster
node.name: elk-node-1
network.host: localhost
http.port: 9200
discovery.type: single-node  # Для одноподной установки
```

**Запустите Elasticsearch:**
```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
sudo systemctl status elasticsearch  # Проверка статуса
```

**Проверка работы:**
```bash
curl -X GET "localhost:9200"
```

### 3. Установка Kibana VPN!!!!!!!!!!!!
```bash
sudo apt install -y kibana
```

**Настройте Kibana:**
```bash
sudo nano /etc/kibana/kibana.yml
```
```yaml
server.port: 5601
server.host: "localhost"
elasticsearch.hosts: ["http://localhost:9200"]
```

**Запустите Kibana:**
```bash
sudo systemctl enable kibana
sudo systemctl start kibana
sudo systemctl status kibana
```

**Доступ к Kibana:**
Откройте в браузере: `http://localhost:5601`

### 4. Установка Logstash
```bash
sudo apt install -y logstash
```

**Пример конфигурации Logstash:** VPN!!!!!!!!!!!!
Создайте конфиг для приема логов:
```bash
sudo nano /etc/logstash/conf.d/my-pipeline.conf
```
```conf
input {
  file {
    path => "/var/log/*.log"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:loglevel} %{GREEDYDATA:message}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "my-logs-%{+YYYY.MM.dd}"
  }
}
```

**Запустите Logstash:**
```bash
sudo systemctl enable logstash
sudo systemctl start logstash
```

### 5. Настройка Filebeat (опционально)
Для сбора логов с других серверов:
```bash
sudo apt install -y filebeat
sudo nano /etc/filebeat/filebeat.yml
```
Настройте вывод в Logstash или напрямую в Elasticsearch.

Далее нам нужно настроить аутентификацию в elasticsearch.
Для этого создадим пароль для встроенного
пользователя kibana_system:
#/usr/share/elasticsearch/bin/elasticsearch-reset-
password -u kibana_system

### Kibana verification code:

    Navigate to the Kibana directory:
        cd /usr/share/kibana/
    Run the verification code script:
        For Linux/macOS: sudo ./bin/kibana-verification-code
        For Windows: sudo .bin/kibana-verification-code.bat --allow-root -c /etc/kibana/kibana.yml
    Copy the output: The command will output a verification code, such as Your verification code is: ******.

### 6. Безопасность
**Включите базовую аутентификацию:**
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

### 7. Работа с Kibana
1. Откройте http://localhost:5601
2. Перейдите в "Stack Management" → "Index Patterns"
3. Создайте индекс-паттерн (например, `my-logs-*`)
4. Используйте вкладку "Discover" для просмотра логов


```

**Настройте HTTPS:**
Используйте сертификаты из `elasticsearch-certutil` или Let's Encrypt.

### 8. Мониторинг и диагностика
- Логи Elasticsearch: `/var/log/elasticsearch/`
- Логи Kibana: `/var/log/kibana/`
- Логи Logstash: `/var/log/logstash/`

### 9. Примеры использования
**Для системных логов:**
```bash
sudo nano /etc/logstash/conf.d/syslog.conf
```
```conf
input {
  syslog {
    port => 514
  }
}
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "syslog-%{+YYYY.MM.dd}"
  }
}
```

### Советы:
1. Начните с тестового стенда
2. Используйте `journalctl -u service_name` для просмотра логов служб
3. Регулярно обновляйте компоненты стека
4. Настройте ротацию индексов в Elasticsearch через Curator
5. Для продакшена используйте отдельные серверы для каждого компонента

Для глубокого изучения ознакомьтесь с официальной документацией:
- [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Kibana Guide](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Logstash Guide](https://www.elastic.co/guide/en/logstash/current/index.html)

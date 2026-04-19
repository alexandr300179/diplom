
#  Дипломная работа по профессии «Системный администратор» - Александров Александр

---------

## Задача
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в [Yandex Cloud](https://cloud.yandex.com/) и отвечать минимальным стандартам безопасности: запрещается выкладывать токен от облака в git. Используйте [инструкцию](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#get-credentials).


### 1.	Структура проекта

```
terraform/
 ├── main.tf
 ├── variables.tf
 ├── terraform.tfvars
```

**Переменные (токен не в git!)**

**variables.tf**

```
variable "yc_token" {}
variable "cloud_id" {}
variable "folder_id" {}
```

**terraform.tfvars**

```
yc_token  = "TOKEN"
cloud_id  = "CLOUD_ID"
folder_id = "FOLDER_ID"
```

[main.tf](terraform/main.tf)
[variables.tf](terraform/variables.tf)
[terraform.tfvars](terraform/terraform.tfvars)


### Яндекс консоль

**Консоль Yandex Cloud**

![Консоль Yandex Cloud](img/1%20console-yandex-cloud.png)

**Виртуальные машины**

![Виртуальные машины](img/2%20VM.PNG)

**VPS**

![VPS](img/3%20VPS.PNG)

**Подсети**

![Подсети](img/4%20Подсети.PNG)

**Целевые группы**

![Целевые группы](img/5%20целевые%20группы.PNG)

**Балансировщик**

![Балансировщик](img/6%20Балансировщик.PNG)

**Роутер**

![Роутер](img/7%20роутер.PNG)

**Backend**

![Backend](img/backend.PNG)



### 2. Сеть (VPC, подсети, NAT)

Создана единая VPC сеть с разделением на публичную и приватные подсети (terraform, публичная подсеть, приватные подсети, nat gateway, bastion host).

Bastion host 
ssh ubuntu@ 89.169.131.105 (подключение через публичный IP)

![Bastion](img/8%20bastion.PNG)

WEB сервера
Созданы две ВМ в разных зонах без публичного IP

![web1](img/9%20web1.PNG)

![web2](img/10%20web2.PNG)

**Ansible (nginx)**

**inventory.ini**

```
[web]
web1.ru-central1.internal
web2.ru-central1.internal
```

**nginx.yml**

```
- hosts: web
  become: yes

  tasks:
    - name: install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

ansible-playbook -i inventory.ini nginx.yml

![web1 nginx](img/11%20nginx%20web1.PNG)

![web2 nginx](img/12%20nginx%20web2.PNG)


### 3. LOAD BALANCER (YC CLI)

[Балансировщик](http://158.160.236.105/)

### 4. Zabbix

![Zabbix](img/13%20zabbix.PNG)

**Доступ к панели** 

[Zabbix](http://93.77.182.155/zabbix)

логин - Admin, пароль - zabbix

Настроен дашборд Инфраструктура Диплома, добавлены триггеры

### 5. ELK (Docker)

**Elastic**

![Elastic](img/14%20elastic.PNG)

**Kibana**

![Kibana](img/15%20kibana.PNG)

[Elastic](http://130.193.49.192:5601/app/discover) 



### 6. Автоматизация (Ansible)

**Структура**

```
ansible/
├── inventory.ini
├── nginx.yml
├── deploy.yml
├── files/
│   ├── nginx.conf
│   └── index.html
├── docker/
│   ├── elastic/
│   ├── kibana/
│   └── filebeat/
```
![Ansible](img/ansible%20на%20bastion.PNG) 

**inventory.ini**

[inventory.ini](ansible/inventory.ini) 

**docker/elastic/docker-compose.yml**

[elastic](ansible/docker/elastic/docker-compose.yml) 

**docker/kibana/docker-compose.yml**

[kibana](ansible/docker/kibana/docker-compose.yml) 

**docker/filebeat/filebeat.yml**

[filebeat](ansible/docker/filebeat/filebeat.yml) 

**docker/filebeat/docker-compose.yml**

[filebeat](ansible/docker/filebeat/docker-compose.yml) 

**deploy.yml**

[deploy.yml](ansible/deploy.yml) 

**nginx.yml**

[nginx.yml](ansible/nginx.yml) 

**Запускаем - ansible-playbook -i inventory.ini nginx.yml**


### 7. SECURITY GROUPS

Ограничен доступ только к необходимым портам
•	Bastion → 22 
•	Web → 80 + SSH только с bastion 
•	Zabbix → 80 
•	Kibana → 5601 
•	Elastic → 9200 (внутри сети)

### 8. Настроен BACKUP, снимки дисков

![Снимки дисков](img/снимки%20дисков.PNG)


### 9. ИТОГ

Разработана отказоустойчивая инфраструктура для веб-приложения с использованием:
•	Terraform — создание инфраструктуры 
•	Ansible — конфигурация серверов 
•	Docker Compose — деплой сервисов
•	Zabbix — мониторинг
•	ELK (Elasticsearch + Kibana + Filebeat) — централизованный сбор логов

**Архитектура**

•	1 VPC сеть
•	Публичная подсеть:
•	Bastion host (SSH доступ)
•	Zabbix server
•	Kibana
•	Load Balancer
•	Приватные подсети:
•	Web1 (nginx)
•	Web2 (nginx)
•	Elasticsearch

**Сайт**

•	2 веб-сервера (web1, web2) в разных зонах
•	Nginx + статический контент
•	Нет внешних IP
•	Доступ только через Application Load Balancer

**Настроен Yandex Application Load Balancer**

**Сеть и безопасность**

•	Bastion host — единственная точка входа по SSH
•	Доступ к приватным ВМ только через bastion
•	Security Groups:
•	web: 80 (LB), 22 (bastion), 10050 (Zabbix)
•	elastic: 9200 (internal)
•	kibana: 5601 (public)
•	zabbix: 80, 10051
•	NAT Gateway — доступ в интернет для приватных ВМ

**Установлен Zabbix Server**

**Логи (ELK)**

Развернут Elasticsearch
Kibana подключена к Elasticsearch для анализа логов
Filebeat установлен на web1 и web2 для отправк логов в Elasticsearch

**Автоматизация (Ansible)**

•	Установка Docker и docker-compose
•	Доставка docker-compose.yml
•	Запуск контейнеров:
•	Elasticsearch
•	Kibana
•	Filebeat

**Инфраструктура (Terraform)**

Создаёт:
•	VPC
•	Подсети (public/private)
•	NAT Gateway
•	ВМ:
•	bastion
•	web1, web2
•	elastic
•	kibana
•	zabbix

**Резервное копирование**

•	Настроены snapshots дисков ВМ
•	Периодичность: ежедневно
•	Хранение: 7 дней
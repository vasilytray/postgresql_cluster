Для утановки загрузи или клонируй с репозитория
>git clone https://github.com/vasilytray/postgresql_cluster.git

Перейди в директорию плейбук
>cd postgresql_cluster/

# Меняем переменные:
Перейдите в каталог playbook
> cd postgresql_cluster/

Отредактируйте файл inventory
Укажите (не общедоступные) IP-адреса и параметры подключения 
> nano inventory

Отредактируйте файл /group_vars/all.yml
ansible_user, 
ansible_ssh_pass или 
ansible_ssh_private_key_file для вашей среды
> nano /group_vars/all.yml

Отредактируйте файл /group_vars/pgbackrest.yml
ansible_user, 
ansible_ssh_pass или 
ansible_ssh_private_key_file для вашей среды
> nano /group_vars/all.yml

Отредактируйте файл переменных vars/ main.yml
> nano vars/main.yml

Минимальный набор переменных:
proxy_env# если требуется ( для скачивания пакетов )
cluster_vip# виртуальный IP для клиентского доступа к базам данных в кластере (необязательно)
patroni_cluster_name
postgresql_version
postgresql_data_dir
with_haproxy_load_balancing 'true'(Тип A) или 'false'/default (Тип B)
dcs_type# "etcd" (по умолчанию) или "consul" (тип C)

изменим пароль оутентификации keepalived:
> nano roles/keepalived/templates/keepalived.conf.j2
    переменную: 
    auth_pass 1ce24b6e

Изменим настройки кластера PostgreSQL
> nano vars/system.yml
etc_hosts: []
- удалим сервер из
# - "10.128.64.143 pgbackrest.minio.local minio.local s3.eu-west-3.amazonaws.com"  # example (MinIO)
- удалим IP адреса из
ntp_servers: []
#  - "10.128.64.44"
#  - "10.128.64.45"
- подключим если надо для 1С баз локаль ru_RU
locale_gen:
  - { language_country: "en_US", encoding: "UTF-8" }
-> #  - { language_country: "ru_RU", encoding: "UTF-8" }
- заменим в 
# sudo
sudo_users:
  - name: "postgres"
    nopasswd: "yes"  # or "no" to require a password
->  nopasswd: "no"  # or "no" to require a password
# закончили менять переменные!!!

Для безпроблемного доступа с 1 сервера на 2-й и 3-й создадим ssh-key
> ssh-keygen
оставим все по умолчанию
и после создания ключей перейдем в директорию где они лежат
> cd /root/.ssh/
копируем публичный ключ на наши сервера 2 и 3
> ssh-copy-id root@<ip server2> # потребуется пароль от ssh server2
> ssh-copy-id root@<ip server3> # потребуется пароль от ssh server3
и проверяем подключение к серверам:
> ssh root@<ip server2>
> ssh root@<ip server3>
Если все окей проверим как видит сервера 
вернемся обратно в деррикторию /postgresql_cluster
и проверим подключение ansible
> ansible all -m ping
Запустим плэйбук:
> ansible-playbook deploy_pgcluster.yml

# Проверим установку
# ПРоверим работоспособность etcd
> etcd --version
> etcdctl version
> systemctl status etcd
> etcdctl endpoint health --cluster
> etcdctl member list

Укажем во конфигах etcd на каждом узле кластера, сто это уже работающий узел
> nano /etc/etcd/etcd.conf
ETCD_INITIAL_CLUSTER_STATE="existing"
сохраним, выйдем и рестартуем etcd
> systemctl restart etcd
проверим  работу etcd
> etcdctl member list
> patronictl list

> etcdctl member list
18b6878843860580, started, pgnode01, http://192.168.171.1:2380, http://192.168.171.1:2379, false
44ad459c595c961a, started, pgnode02, http://192.168.171.2:2380, http://192.168.171.2:2379, false
d968bf8a52211a84, started, pgnode03, http://192.168.171.3:2380, http://192.168.171.3:2379, false

# Проверим работоспособность keepalived
> systemctl status keepalived

# Проверим работоспособность HAproxy
> systemctl status haproxy
Посмотрим на каком сервере находится наш VIP
на каждом терминале посмотрим
> ip a
> ss -tunapl

# Проверим работоспособность Patroni
запустим службу patroni
> sudo systemctl start patroni.service
> patronictl list
если после установки отвечает
> root@z34096:~/postgresql_cluster# patronictl list
2023-03-09 11:37:21,548 - WARNING - Listing members: No cluster names were provided

то запускаем 
> patronictl -c /etc/patroni/patroni.yml list

или выходим и заново автризуемся на сервере

root@pgnode02:~# patronictl list
+ Cluster: postgres-cluster --------------+---------+----+-----------+
| Member   | Host          | Role         | State   | TL | Lag in MB |
+----------+---------------+--------------+---------+----+-----------+
| pgnode01 | 192.168.171.1 | Leader       | running |  1 |           |
| pgnode02 | 192.168.171.2 | Replica      | running |  1 |         0 |
| pgnode03 | 192.168.171.3 | Sync Standby | running |  1 |         0 |
+----------+---------------+--------------+---------+----+-----------+

# проверим как работает patroni Сменим Лидера в кластере:
> patronictl switchover

root@pgnode02:~# patronictl switchover
Current cluster topology
+ Cluster: postgres-cluster --------------+---------+----+-----------+
| Member   | Host          | Role         | State   | TL | Lag in MB |
+----------+---------------+--------------+---------+----+-----------+
| pgnode01 | 192.168.171.1 | Leader       | running |  1 |           |
| pgnode02 | 192.168.171.2 | Replica      | running |  1 |         0 |
| pgnode03 | 192.168.171.3 | Sync Standby | running |  1 |         0 |
+----------+---------------+--------------+---------+----+-----------+
Primary [pgnode01]: 
Candidate ['pgnode02', 'pgnode03'] []: pgnode03
When should the switchover take place (e.g. 2023-03-09T07:17 )  [now]: now
Are you sure you want to switchover cluster postgres-cluster, demoting current leader pgnode01? [y/N]: y
2023-03-09 06:18:03.59482 Successfully switched over to "pgnode03"
+ Cluster: postgres-cluster ---------+---------+----+-----------+
| Member   | Host          | Role    | State   | TL | Lag in MB |
+----------+---------------+---------+---------+----+-----------+
| pgnode01 | 192.168.171.1 | Replica | stopped |    |   unknown |
| pgnode02 | 192.168.171.2 | Replica | running |  1 |         0 |
| pgnode03 | 192.168.171.3 | Leader  | running |  1 |           |
+----------+---------------+---------+---------+----+-----------+

# Проверим REST API
1. установим http
> apt  install httpie
Отправим запрос на 8008 порт нашей ноды
> http http://192.168.171.3:8008

# Все работает кластер стабилен

# Еще несколько команд на всякий случай под руку:
# Изменение конфигупрации PostgreSQL
Посмотреть конфигурацию можно командой
> patronictl show-config
а изменить параметры, так чтобы они вступили в силу на других нодах кластера
> patronictl edit-config
кроме того Параметры постгрес можем менять в patroni.yml или в etcd в confg

# Перезагрузка postgresql
> systemctl restart postgresql

# _____ Добавляем в кластер ноды____
# __Подготовим ноды к расширению кластера__
Нам надо добавить новый узел или подсеть в pg_hba.conf на всех нодах кластера
1. зададим переменные /vars/add_newnode.yml
new_ip_address: "192.168.171.4"
pg_hba_file: "/etc/postgresql/{{postgresql_version}}/main/pg_hba.conf"
запустим плейбук
Плейбук предназначен для создания инстансов mysql8 на серверах 
server1
server2

Для создания инстанса необходимо
1. Заполнить vars.yaml необходимыми значениями 
2. Запустить плейбук создания инстанса на первом хосте

ansible-playbook -i hosts --limit server1 create_instances.yaml

3. Запустить плейбук создания инстанса на втором хосте

ansible-playbook -i hosts --limit server2 create_instances.yaml

Пункты 2-3 можно объединить, не указывая --limit, но тогда в случае проблем будет сложнее разбираться
в каше вывода. Так как ансибла будет выполнять их параллельно. 

4. Запустить настройку репликации

ansible-playbook -i hosts mysql_replication.yaml


Проверки и защиты от дурака. 
Для исключения влияния на существующие инстансы данный плейбук отработает только для заведения нового инстанса. 
При его создании:

- должна быть vg с именем, указанным в vars
- на ней должно хватать места для разделов под БД и логи, с запасом в 1Гб (для снапшотов)

НЕ должно быть
- lvm c именем mysql_{{ instance }} (в любой VG)
- lvm c именем mysql_{{ instance }}_log (в любой VG)
- смонтированных маунтов в /var/lib/mysql_{{ instance }} /var/log/mysql_{{ instance }}
- файла сервиса systemd /etc/systemd/system/{{ instance }}-mysql.service
- скрипта запуска контейнера /usr/local/bin/{{ instance }}-mysql 
- должен быть свободен выбранный порт (порт должен отсутствовать в выводе docker ps, и в файлах /usr/local/bin/*-mysql)

В случае, если плейбук создал инстанс на хосту (создал все lvm, точки монтирования, службу и запустил контейнер), 
но выпал с ошибкой подключения к базе, можно запустить плейбук 
mysql_init_pb.yaml 
отдельно для инициализации БД. Тут нужна осторожность, так как защит в этом плейбуке уже нет. 
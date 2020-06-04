1. В /etc/systemd/system/kafka.service была ссылка на env, которого в системе не существует EnvironmentFile=/etc/default/kafka
Решение - закомментировал.

2. В systemd используется переменная "%i" а сервис создан без учета этого.
Решение - переименовал название сервиса kafka.service -> kafka@.service

3. Доступ на порт zookeeper был закрыт правилами firewall:
# iptables -L -nv
Chain INPUT (policy ACCEPT 14626 packets, 6960K bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:9092
    0     0 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:9096
  204 12240 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:2181
Решение - сброс правил "iptables -F"

4. В конфигурации путь к логам был указан как /var/lib/kafka100[123], но таких каталогов не было.
Решение - изменил путь на /var/log/kafka100[123]

После чего все запустилось:

kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic MyTopic
Created topic MyTopic.

echo "Hello, world" | kafka-console-producer.sh --broker-list 10.236.30.222:9092 --topic MyTopic

kafka-console-consumer.sh --bootstrap-server 10.236.30.222:9092 --topic MyTopic --from-beginning
Hello, world


------------------------------------------
Запуск роли:

!!!Внимание версия на которой запускать:
ansible-playbook 2.9.2

ansible-playbook site.yml -u <USERNAME> -i <IP>, -e ansible_ssh_port=<PORT> --private-key=~/.ssh/<SSH_KEY>.key

Например:

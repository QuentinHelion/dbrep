- Chaque docker-compose est à mettre dans un serveur (ce sont tous les même globalement donc peu importe lequel va dans lequel), renommer les fichiers en docker-compose.yml sur chaque serveur pour simplifier les choses.


Sur chaque serveur:

- Dans chaque docker compose, remplacer les valeurs dans "loose-group-replication-group-seeds" par les bons hostnames (ceux des serveurs dans lesquels se trouvent les bdd)

- docker-compose up


Sur le serveur maître:

docker exec -t node1 mysql -uroot -proot \
  -e "SET @@GLOBAL.group_replication_bootstrap_group=1;" \
  -e "create user 'repl'@'%';" \
  -e "GRANT REPLICATION SLAVE ON *.* TO repl@'%';" \
  -e "flush privileges;" \
  -e "change master to master_user='root' for channel 'group_replication_recovery';" \
  -e "START GROUP_REPLICATION;" \
  -e "SET @@GLOBAL.group_replication_bootstrap_group=0;" \
  -e "SELECT * FROM performance_schema.replication_group_members;"


Sur chaque serveur esclave (en remplaçant nodeX par la bonne valeur):


docker exec -t nodeX mysql -uroot -proot \
  -e "change master to master_user='repl' for channel 'group_replication_recovery';" \
  -e "START GROUP_REPLICATION;"


- Pour checker si tout s'est bien passé (sur chaque serveur en remplaçant nodeX par la bonne valeur):

docker exec -t nodeX mysql -uroot -proot \
  -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
  -e "SELECT * FROM performance_schema.replication_group_members;"

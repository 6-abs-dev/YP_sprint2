Заполняем mongodb данными

```shell
./scripts/mongo-init.sh
```

## Инструкция по запуску и проверке
docker compose up -d



Подключитесь к серверу конфигурации и сделайте инициализацию:
docker exec -it configSrv mongosh --port 27017

> rs.initiate(
{
_id : "config_server",
configsvr: true,
members: [
{ _id : 0, host : "configSrv:27017" }
]
}
);
> exit();

Инициализируйте шарды:
docker exec -it shard1 mongosh --port 27018

> rs.initiate(
{
_id : "shard1",
members: [
{ _id : 0, host : "shard1:27018" },
{_id: 1, host: "mongodb1repl1:27021"},
{_id: 2, host: "mongodb1repl2:27022"},
{_id: 3, host: "mongodb1repl3:27023"}
]
}
);
> exit();

docker exec -it shard2 mongosh --port 27019

> rs.initiate(
{
_id : "shard2",
members: [
{ _id : 0, host : "shard2:27019" },
{_id: 1, host: "mongodb2repl1:27024"},
{_id: 2, host: "mongodb2repl2:27025"},
{_id: 3, host: "mongodb2repl3:27026"}
]
}
);
> exit();

Инцициализируйте роутер и наполните его тестовыми данными:
docker exec -it mongos_router mongosh --port 27020

> sh.addShard( "shard1/shard1:27018");
> sh.addShard( "shard2/shard2:27019");

> sh.enableSharding("somedb");
> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

> use somedb

> for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

> db.helloDoc.countDocuments()
> exit(); 

Сделайть проверку на шардах:
docker exec -it shard1 mongosh --port 27018
> use somedb;
> db.helloDoc.countDocuments();
> exit();

Сделайть проверку на втором шарде:
docker exec -it shard2 mongosh --port 27019
> use somedb;
> db.helloDoc.countDocuments();
> exit(); 


Отразить статус инфраструктуры:
http://localhost:8080

Проверить кэширование:
http://localhost:8080/helloDoc/users
(кэш работает и возвращает вторичные запросы меньше чем за 100мс)
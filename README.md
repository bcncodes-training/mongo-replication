# mongo-replication

C:\MongoDB\Server\bin>mongod -replSet rs0 --dbpath C:\MongoDB\Replicas\repl_1 --port 27017
C:\MongoDB\Server\bin>mongod -replSet rs0 --dbpath C:\MongoDB\Replicas\repl_2 --port 27018
C:\MongoDB\Server\bin>mongod -replSet rs0 --dbpath C:\MongoDB\Replicas\repl_3 --port 27019


# 1. Primero definiremos las instancias

##	a. Para ello deberemos crear 3 carpetas:

		mkdir repl_1 repl_2 repl_3

##	b. Lanzamos las instancias. Para ello abre tres ventanas de consola y escribe

	#ventana 1
		mongod –-replSet rs0 –-dbpath /data/replica/repl1 –-port 27017

	#ventana 2
		mongod –-replSet rs0 –-dbpath /data/replica/repl2 –-port 27018

	#ventana 3
		mongod –-replSet rs0 –-dbpath /data/replica/repl3 –-port 27019

##	c. Nos conectamos con 3 clientes. Para ello abre otras 3 ventanas de consola:

	#cliente 1
		mongo
	#cliente 2
		mongo –port 27018
	#cliente 3
		mongo –port 27019

## 2. Generaremos la configuración de replica
	a. En el cliente 1 escribe:

	var rsconf = {_id: "rs0", members: [{_id: 0,host: 'localhost:27017'},
					    {_id: 1,host: 'localhost:27018'},
					    {_id: 3, host: 'localhost:27019'}
			]};

 

	b. Luego inicializamos el conjunto de réplicas:

	rs.initiate(rsconf);
 

c. Verificaremos el estado de las réplicas:

rs.status();
 

3. Habilitaremos los secundarios para que acepten replicas
a. En cada secundario y acepta el estado de secundario

#cliente 2
rs.slaveOk();

#cliente 3

rs.slaveOk();

Probando el conjunto
Ahora probaremos el conjunto:

Crea una base de datos, una colección e insertaremos datos en el primario:
use shipsdb;
db.ships.insert({name:'USS Enterprise-D', operator:'Starfleet', type:'Explorer', class:'Galaxy', crew:750, codes:[10, 11, 12]});

db.ships.insert({name:'USS Prometheus', operator:'Starfleet', class:'Prometheus', crew:4, codes:[1, 14, 17]});

 

db.ships.find();

Comprueba la réplica en los secundarios:
show collections;
 

use shipsdb;

 

db.ships.find();

 

Ahora deshabilitaremos el primario para validar que los secundarios se reorganizan para que uno se haga primario

Tira el nodo primario: en la consola #1 haz CTRL-C
Verifica los secundarios
rs.status();
 

Uno de ellos se debe marcar como nuevo primario
Inserta datos en el nuevo primario
use shipsdb;
 

db.ships.insert({name:'USS Defiant', operator:'Starfleet', class:'Defiant', crew:50, codes:[10, 17, 19]}); db.ships.insert({name:'IKS Buruk', operator:' Klingon Empire', class:'Warship', crew:40, codes:[100, 110, 120]}) ;

db.ships.insert({name:'IKS Somraw', operator:' Klingon Empire', class:'Raptor', crew:50, codes:[101, 111, 120]});

 

db.ships.find();

 

Comprueba en el secundario
 

use shipsdb;
db.ships.find();

 

Levantaremos nuevamente el primario para verificar que el conjunto se reorganiza:

Vuelve a levantar el primario: para ello inicialízalo:
#ventana 1
mongod -replSet rs0 --dbpath C:\MongoDB\Replicas\repl_1 --port 27017

Verifica su situación y si no es primario acepta como secundario
#cliente 1
mongo

rs.status();

rs.slaveOk();

 

use shipsdb;
db.ships.find();

Verifica que los datos se han actualizado en el nodo recientemente levantado (los datos insertados cuando estaba caído deben aparecer)

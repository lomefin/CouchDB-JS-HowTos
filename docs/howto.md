Usando CouchDB CouchApps y Markdown
=================================
Con la aparición de bases de datos NoSql, la decisión de qué tipo de base de datos usar se está poniendo compleja. Entonces, decidimos aportar con ejemplos. Estaremos desarrollando mini aplicaciones usando las distintas tecnologías de bases de datos, tratando de darle un uso apropiado a las mismas y dando una conclusión final.

BroTip: El proyecto completo se puede [descargar en GitHub](https://github.com/lomefin/CouchDB-JS-HowTos)

Acerca de CouchDB
-----------------

CouchDB es un servidor de bases de datos documentales creado por Apache Foundation, que se puede acceder vía una API RESTful en JSON.
Es schema-free (o sea no hay una definición de los elementos que debe contener), sin embargo se le pueden realizar queries y puede indexar, con un sistema de reportes que usa a javascript como lenguaje de consulta.

Hey, javascript? Si, como leíste anteriormente, se consulta mediante un API RESTful en JSON, eso grita javascript. Por lo que ya tenemos una noción de qué tipo de cosas podriamos hacer fácilmente con ella.

***Que NO es CouchDB***

Esto lo extraigo directamente de la página de couchDB, pero es muy importante que lo tengan en cuenta.

- CouchDB NO es una base de datos relacional
- CouchDB NO es un reemplazo a las bases de datos relacionales
- CouchDB NO es una base de datos orientada a objetos.

Si quieren adentrarse más en las definiciones de couchDB, deberían visitar la [página de su documentación](http://couchdb.apache.org/docs/intro.html).

Enough said, como parto?
------------------------

Si ya quieres comenzar, debes primero instalar CouchDB, puedes bajar el usar [build-couchdb](https://github.com/iriscouch/build-couchdb) para construirlo o instalar el código fuente. Yo soy más flojo y le pido el favor a mi querido amigo apt.

	sudo apt-get install couchdb
	sudo apt-get install couchapp

CouchDB está escrito en ERLang un lenguaje pensado para la telefonía y comunicaciones asi que aparte de los binarios de couch hay que bajar un monton de librerías de erlang. Lo otro que descargamos es couchapp que es para el desarrollo de aplicaciones independientes en javascript, lo dejaré instalado en caso de requerirlo en un futuro próximo.

Mi base de datos
----------------

Cuando termines este proceso, anda a http://localhost:5984/_utils/ y tu base de datos ya estará ahí esperandote!

Primer dato en la base
----------------------

Para este ejemplo, haré una base de datos que almacene los howto's que genero, tal como este, asi que mi ejemplo más simple de Hello World será el siguiente:

1.	Anda a tu base de datos por la dirección anterior y crea una base de datos llamada howtos.
2.	Consigue un cliente de HTTP para este ejemplo usaré Telnet
3.  Ingresa al servidor: <code>telnet localhost 5984</code>
4.	Copia y pega esto: (Si, puede ser un poco marciano)

		PUT /howtos/some_doc_id HTTP/1.0
		Content-Length: 61
		Content-Type: application/json

		{"Titulo":"Hola Mundo!","Mensaje":"Este es mi primer texto!"}
	Hernán comentó que hablará sobre los verbos de HTTP, quizás eso te pueda ayudar a entender que acaba de pasar.
5.	La respuesta que nos dio fue la siguiente:

		HTTP/1.0 201 Created
		Server: CouchDB/1.0.1 (Erlang OTP/R14B)
		Location: http://127.0.0.1:5984/howtos/some_doc_id
		Etag: "1-e8b1eadf3f1be10c563852796b791e7c"
		Date: Sat, 10 Dec 2011 21:09:36 GMT
		Content-Type: text/plain;charset=utf-8
		Content-Length: 74
		Cache-Control: must-revalidate

		{"ok":true,"id":"hola_mundo","rev":"1-e8b1eadf3f1be10c563852796b791e7c"}		
		Connection closed by foreign host.
	Sigue el chino, bueno, lo que quiere decirnos es básicamente lo siguiente:

		*	201 Created: Hicimos un PUT y fue válido, por lo tanto el elemento fue creado.
		*	Los otros tags no son importantes <em>para este ejemplo</em>.
		*	Finalmente, viene un string en JSON diciendonos que el documento fue creado con el id hola_mundo y su <em>número de revisión</em>.

6.	Revisen su base de datos, el documento debería estar [listado](http://localhost:5984/_utils/database.html?howtos).

Ya con eso tenemos lo básico para partir, ya puedes suponer maneras en que podemos hacer funcionar este sistema.
Ahora veremos algún ejemplo más poderoso de cómo trabajar con CouchDB.

Falta una cosa, configurar el servidor para darle un nombre de usuario y contraseña. Si entran al administrador, en la esquina inferior derecha está el link para configurar la seguridad, para terminos practicos usaré de usuario a USER y de password PASS por si los vuelven a ver más adelante.

Aplicación de HowTo's en Markdown
--------------------------------

Vamos a hacer una aplicación muy simple en Javascript, solo será un set de páginas en donde cada una de ellas podrá realizar una actividad, estas deberían ser:

-	Crear un Howto (con título, autor y contenido)
-	Editar el HowTo
-	Ver una lista de HowTo's
-	Ver los detalles de un HowTo (su historial)
-	Ver un HowTo en particular (una revisión en particular)

Para editar el Markdown usaremos [MarkitUp!](http://markitup.jaysalvat.com/home/) que es el editor que usamos en Ataxic.

Por primera vez en la historia, usaré solamente JQuery, aunque ustedes deben saber que no considero que JQuery sea suficiente para hacer aplicaciones decentes.

El servidor
------------
Una cosa que me había olvidado de contarles, es que CouchDB al ser 100% HTTP, tiene la capacidad de saltarse el paradigma de las tres capas (Browser, Servidor de Aplicaciones y Datos) para transformarse en uno de dos capas: El Browser y el resto. La gran ventaja de esto es la capacidad de escalar horizontalmente, asi que si tu aplicación funciona bajo el paradigma de CouchDB, quizás te convenga hacerlo asi, sino deberías revisar un tutorial de Couch con otro lenguaje.

Entonces, mis páginas donde van a estar? Fácil: Las páginas serán servidas directamente desde el servidor CouchDB, y para eso usaremos CouchApp.

Primero creamos la aplicación

	couchapp generate howtos

Dentro tendremos una aplicación de Couch lleno de cosas lindas que pronto sabremos usar.
	
Obviamente primero haremos el Hello World de un CouchApp, pero muy rapidamente

	cd howtos
	couchapp generate show hello
	vim shows/hello.js

Dentro de vi escribimos

	function(doc, req)
	{
		return "Hello World";
	}

Listo, ya estás seco en CouchApp, vamos a mandar la aplicación al servidor ahora.

Ahora el PUSH! Vamos a mandar la aplicación al servidor para eso le decimos a couchapp que envie el directorio arriba.

	couchapp push http://USER:PASS@localhost:5894/howtos

Listo! Tenemos una aplicación arriba, si vamos a http://localhost:5984/howtos/_design/howtos/index.html veremos una linda pagina donde puedes crear un usuario y mandar mensajes!

Estructura del CouchApp
-----------------------

Para entender que se hizo dentro de la aplicación veamos algunos de los directorios creados:

-	evently: Librería para hacer algunas consultas complejas
-	shows:	Vistas de la base de datos (como la que vimos hace un rato)
-	vendor: Para colocar librerías externas
-	_attachments: La carpeta donde colocaremos los archivos que nos interesa subir

Ahora haremos nuestra página que muestre los documentos de nuestra base de datos, puedes ver el código completo en [el repositorio en github](https://github.com/lomefin/CouchDB-JS-HowTos/blob/master/howtos/_attachments/doc_list.html), lo importante de esta página es lo siguiente:

	<ul id="result"></ul>
	<script>
	db = $.couch.db('howtos');
	db.allDocs(
	{
		success: function(docs)
		{
			$.each(docs.rows,
				function(idx,obj)
				{
					$("#result").append('<li><a href="/_utils/document.html?howtos/'+obj.id+'">'+obj.id+'</a></li>');
				}
			)
		}
	})
	</script>

Aquí simplemente le digo que saque todos los documentos de la base de datos howtos.

Para enviarlo al servidor


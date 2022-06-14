 - Crear un fichero README con: 
	- Instrucciones para arrancar la aplicacion
	- Breve documentación sobre el diseño, estructura del código y cualquier anotación que quieras añadir sobre
extensibilidad, mantenimiento, seguridad, rendimiento, etc., que no haya tenido tiempo de implementar.
	- Breve descripción de supuestos o requisitos no implementados.
	
# Solución Inventario

## Arrancar la solución

Tras abrir la solución, seleccionaremos como proyectos de inicio múltiples los siguientes: 
 - **Inventory.API.Host** - Ejecuta la Api con Swagger
 - **Inventory.Web.Mvc** - Ejecutamos la web creada para el Frontend, las llamadas las realiza a través de la integración con la Api
 
## Diseño, estructura y funcionalidad

La solución se divide en 4 grupos: 

1. Api
   Compuesta de la WebApi (Inventory.API.Host), y la librería donde está toda la logica (Inventory.API), dividido para poder implementar de forma facil en otro tipo de WebApi (Minimal Api por ejemplo). Implementado el patron **CQRS** (Mediator), dividiendo los controladores en el de Commands y otro para las Queries. Añadida tambien **seguridad** a través de un Middleware, para que los metodos solo sean accesibles mediante scheme basic. Tanto para llamadas Swagger, como si se quiere realizar desde otro entorno, como Postman, el usuario y contraseña para acceder es user: **test** y password: **test**. Utilizado **inyección de dependencias** (Services) para referenciar a los repositorios. 
   
2. Backend (Capas principales)
	Incluimos las **capas de Infrastructure y Domain**, teniendo en la de Domain la logica como las **entidades, validaciones con FluentValidation, Eventos, Settings, Interfaces, Requests and Responses objects**, y en la de Infrastructure la utilizamos para **Repositorio y DbContext**, donde guardamos los datos del inventario. 
	También hay otras 2 capas añadidas adicionalmente, una llamada **Services.Utils**, donde está destinada a poner utilidades con logica, para utilizar en toda la solución, en este caso, tengo un **ApiService**, para gestionar las llamadas a la api desde los proyectos web. 
	Por último, he añadido otra librería llamada **Services.Notifications**, donde simulamos un servicio de envío de mail, incluyendo en esta capa, y en el DbContext, la funcionalidad **ILogger**, para tener seguimiento.
 
3. Frontend (Presentation)
	He agregado un **proyecto Mvc**, donde al ejecutar, se puede ir añadiendo items en el inventario, y debajo se sitúa un grid, en el que se va actualizando los registros introducidos, borrados, así como los expirados. 
	La lógica de los items es que se ponga un **Name, Cantidad, y Días de expiración**, guardandose en memoria. **NOTA: Si ponemos dias de expiración 0, por defecto se guarda con la Datetime.Utc.Now, sumando los días indicados, y cada vez que se realiza un cambio en el DbContext, se revisa los items, y si está caducado, se muestra en amarillo.** 
	Por defecto está que debe pasar un minuto, despues de la fecha de expiración, para que el item se considere expirado, y se muestre como tal. A la derecha de los items, se encuentra un boton de borrado, que ejecuta la funcion de eliminar el item del inventario.
	Las gestión de los items, está referenciado directamente en la Api, añadiendo en los appsettings.json, la configuración con la url, endpoints, así como la autenticación a la hora de realizar las gestiones con los items. 
4. Test
	Añadidos unit test con **xUnit**, para validar las requests con exceptions, y las logicas de insercción y borrado de items, además de otras funciones como el que compruebe el duplicado.

## Supuestos

1. **Añadir Item**: Se crea con los campos Name, Canditidad y días de expiración, con la logica anterioramente comentada. 
2. **Borrar Item**: Se borra el item, y ya no está disponible para revisar. 
3. **Evento ItemRemoved**: Cuando se borra el item, se añade el evento al item. Antes de hacer el guardado en BD, se recupera todos los eventos sin publicar (Tiene un IsPublished el  DomainEvent), y cuando se realiza el SaveChanges, se mandan los eventos. 
4. **Evento ItemExpired**: Cuando se realiza cualquier modificación en el DbContext, se revisa si la fecha de expiración ya ha caducado (Tiene 1 minuto de margen, pero configurable por field del objeto), y si es asi, se añade el evento de caducidad, y se cambia la propiedad IsCaducated, y al mandar el evento, se modifica tambien el CaducatedNotificationSended, para que no se envie hasta que no se quiera eliminar por parte del usuario. 
	
	Comentar que la logica del guardado consiste en recuperar los eventos sin enviar, guardar los datos, y checkear eventos y publicar. Además, todos los eventos añadidos, tanto el de Expiracion, como el de Borrado, he puesto una iteracción con el ILogger, escribiendo el tipo de evento, así como hace un EmailSenderSimulator, para poner algo de lógica. 
	
## Resumen Requisitos

He podido añadir todo lo necesario que se pedía, y lo opcional, solo me ha faltado el punto de añadir Vue o React.
No he tocado esa tecnología, y los conocimientos que tenía eran basicos, he intentado revisar, con varios tutoriales y formación, pero ya se echaba el tiempo encima. Con mas tiempo estoy convencido de que lo podría implementar sin mayores problemas. 


	
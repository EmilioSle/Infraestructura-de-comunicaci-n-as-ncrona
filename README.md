# Práctica individual 3.1

Exportación del carrito de NexoCommerce con Laravel. La API publica el mensaje en la cola `exportaciones-carrito`, almacenada en la tabla `jobs` de SQLite. Un worker independiente genera el CSV y conserva el identificador de correlación.

## Entrega

- [Informe en Word](Informe_Practica_3_1_IS803.docx)
- [Diagrama de comunicación](Diagrama_Practica_3_1.png)
- [Registro de pruebas HTTP](Evidencia_Practica_3_1.txt)
- [Colección de Postman](Practica_3_1.postman_collection.json)

## Ejecución local

Requiere PHP con SQLite y Composer. Desde PowerShell:

```powershell
composer install
Copy-Item .env.example .env
php artisan key:generate
New-Item -ItemType File database/database.sqlite
php artisan migrate
php artisan demo:preparar-api
php artisan serve --host=127.0.0.1 --port=8000
```

En un entorno local ya configurado, conserva tu archivo `.env` y usa una base SQLite de pruebas. Importa la colección y configura el token de demostración como valor local de la variable `token`.

1. Envía las tres solicitudes asíncronas con el worker detenido. Deben devolver HTTP 202.
2. Consulta la primera exportación y verifica el estado `pendiente`. Envía la solicitud incompleta y verifica HTTP 422.
3. Comprueba que hay tres trabajos en la cola. Arranca el worker en otra terminal.

```powershell
php artisan tinker --execute="echo DB::table('jobs')->where('queue', 'exportaciones-carrito')->count();"
php artisan queue:work exportaciones --queue=exportaciones-carrito --stop-when-empty
```

4. Consulta de nuevo las tres exportaciones y verifica el estado `completada`, el CSV y el mismo `correlation_id` del HTTP 202. Repite el conteo para verificar que la cola quedó vacía.
5. Envía la solicitud síncrona y compara sus tiempos de respuesta. Debe devolver HTTP 201 con la exportación completada.

Para ejecutar las pruebas automatizadas, usa `php artisan test`.
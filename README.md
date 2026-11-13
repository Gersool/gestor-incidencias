# Gestor de Incidencias - Docker Compose

Aplicación web para gestionar incidencias informáticas desplegada con Docker Compose.
Consta de tres servicios: frontend (Nginx), backend (API) y PostgreSQL.

## Arquitectura

- **Frontend**: solo en la red `frontend`. Sirve la interfaz web.
- **Backend**: está en las dos redes (`frontend` y `datos`). Actúa de intermediario.
- **PostgreSQL**: solo en la red `datos`.

### Respuestas a las preguntas

1. **¿Por qué el frontend no necesita conectarse a PostgreSQL?**  
   Porque el frontend solo muestra la interfaz. Todas las operaciones de datos las hace el backend a través de la API. El navegador nunca habla directamente con la base de datos.

2. **¿Por qué el backend pertenece a dos redes?**  
   Porque necesita hablar con el frontend (para recibir las peticiones `/api/`) y también con PostgreSQL (para guardar y consultar los datos). Al estar en ambas redes puede comunicarse con los dos.

3. **¿Qué servicio necesita publicar un puerto en el anfitrión?**  
   Solo el frontend. El usuario accede desde el navegador a `localhost:8080`. El backend y PostgreSQL se comunican internamente por las redes de Docker.

4. **¿Por qué publicar PostgreSQL hacia fuera no es la solución adecuada para un fallo de comunicación interno?**  
   Porque expone la base de datos a Internet/red local de forma innecesaria y peligrosa. Los problemas de comunicación entre contenedores se resuelven con las redes internas y los nombres de servicio, no abriendo puertos al exterior.

5. **¿Qué ventaja ofrece Docker Compose respecto a ejecutar los tres contenedores manualmente?**  
   Con un solo archivo se definen todos los servicios, redes, volúmenes, variables y dependencias. Se arranca, para o recrea todo con pocos comandos (`up`, `down`, `ps`, `logs`) en lugar de tener que recordar y ejecutar muchos `docker run`, `docker network create`, etc.

6. **¿Por qué hemos dividido la arquitectura en dos redes?**  
   Para aplicar el principio de mínimo privilegio. El frontend no necesita llegar a la base de datos, y PostgreSQL no necesita ser visible desde la red del frontend. Solo el backend (el intermediario) está en las dos.

7. **¿Por qué el backend utiliza `db` como dirección de PostgreSQL?**  
   Porque Docker Compose crea un DNS interno. El nombre del servicio (`db`) se resuelve a la IP del contenedor de PostgreSQL dentro de la red. No hace falta usar IPs ni `localhost`.

8. **¿Qué diferencia existe entre detener un proyecto y eliminar sus volúmenes?**  
   - `docker compose down` → elimina contenedores y redes, pero **conserva** el volumen con los datos.  
   - `docker compose down -v` → también elimina el volumen y se pierden todos los datos de PostgreSQL.

9. **¿Qué consecuencias tendría para la empresa perder el volumen de PostgreSQL?**  
   Se perderían todas las incidencias registradas. Sería una pérdida de información crítica del negocio, por eso es importante tener copias de seguridad del volumen o de la base de datos.

## Cómo arrancar

1. Copia `.env.sample` a `.env` y ajusta si es necesario.
2. Ejecuta:

```bash
docker compose up -d

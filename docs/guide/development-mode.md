# Modo Desarrollo

>Asumiendo que ya tenemos lista la instalaci贸n de un proyecto Vue, como mencionamos en le secci贸n anterior, vamos a preparar nuestro entorno de desarrollo.

Echemos un vistazo a la siguiente plantilla.

馃搩`docker-compose.dev.yml`
```sh
version: "3.9"
services:
  web:    
    image: node
    ports:
      - "5173:80"
    volumes:
      - ".:/app"
    environment:
      NODE_ENV: development
    working_dir: /app
    container_name: vue_dev_env
    command: sh -c "cd /app; npm install; npm run dev"
```

Tenga en cuenta lo siguiente:

1. Declaramos el servicio `web`.
    - Seleccione la imagen que se compilar谩, en este caso `node`. **Es mejor detallar las versiones utilizadas**, vale la pena mantenerlas exactamente igual que la compilaci贸n de producci贸n. (Ejemplo: `image: node:16.10-alpine3`).
2. Seleccionamos los puertos que reflejar谩n los puertos del contenedor en ejecuci贸n en nuestro sistema `host`.
3. Montamos todo, desde el directorio actual hasta el contenedor. Esto es necesario para que los cambios locales llamen inmediatamente a reconstruir.
4. `environment` le permite establecer variables de entorno que sean de inter茅s en su caso particular.
5. `working_dir` especifica el directorio de trabajo dentro del contenedor en el que se ejecutar谩n los comandos posteriores.
6. `container_name` permite espec铆ficar el nombre del contenedor.
7. `command` ejecuta comandos, en este caso, instalar dependencias y correr el desarrollo.

**Levante el `modo development` con el comando:**

```sh
docker-compose -f docker-compose.dev.yml up
```

Entonces el terminal arrojar谩 los siguientes mensajes:

```
Starting vue_dev_env ... done
Attaching to vue_dev_env
vue_dev_env | 
vue_dev_env | up to date, audited 203 packages in 863ms
vue_dev_env | 
vue_dev_env | 43 packages are looking for funding
vue_dev_env |   run `npm fund` for details
vue_dev_env | 
vue_dev_env | found 0 vulnerabilities
vue_dev_env | 
vue_dev_env | > frontend-code@0.0.0 dev
vue_dev_env | > vite
vue_dev_env | 
vue_dev_env | 
vue_dev_env |   VITE v3.0.9  ready in 361 ms
vue_dev_env | 
vue_dev_env |   鉃?  Local:   http://localhost:5173/
vue_dev_env |   鉃?  Network: use --host to expose
```

El contenedor levant贸, pero si intent谩ramos abrir el navegador con la ruta `http://localhost:5173/` nos aparecer谩 la siguiente imagen.

![mode-develop](./img/mode-develop1.jpg)

Realmente la aplicaci贸n no se ejecuta en nuestra direcci贸n anfitriona `localhost` o `172.0.0.1`, sino en alguna otra direcci贸n **IP** otorgada por _Docker_. Pero Vite a煤n no lo sabe.

Tomando en cuenta la 煤ltima l铆nea del mensaje que arroja el terminal, `Network: use --host to expose`, esto significa que no hay una direcci贸n **IP** para la red ya que en ning煤n momento hemos [declarado el `host` a Vite](https://vitejs.dev/config/server-options.html).

As铆 que modifiquemos el archivo de configuraci贸n de Vite agregando las siguiente l铆neas.

馃搩`vite.config.ts`
```ts{8,9,10}
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  server: { 
    host: '0.0.0.0'
  }, 
  plugins: [vue()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

Luego, volvamos a ejecutar `docker-compose -f docker-compose.dev.yml up` y ahora s铆 se mostrar谩 alguna nueva **IP** del contenedor.

```sh
vue_dev_env |   鉃?  Local:   http://localhost:5173/
vue_dev_env |   鉃?  Network: http://192.168.32.2:5173/
```

Por lo que con esta **IP** ahora si deber铆a levantar la aplicaci贸n en el navegador.

![mode-develop](./img/mode-develop2.jpg)

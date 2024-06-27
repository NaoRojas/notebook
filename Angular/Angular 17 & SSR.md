



# Angular 17 & SSR



```js
// Crea un servidor Express y levanta la aplicación Angular por cada peticion que se realiza.
export function app(): express.Express {
  const server = express();
  const serverDistFolder = dirname(fileURLToPath(import.meta.url));
  // Despues del ng-build se crea una carpeta browser en la carpeta dist, la cual contiene el codigo compilado de Angular.
  const browserDistFolder = resolve(serverDistFolder, '../browser');
  const indexHtml = join(serverDistFolder, 'index.server.html');
```





Main.ts

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [provideZoneChangeDetection({ eventCoalescing: true }), provideRouter(routes), provideClientHydration()]
};

// Tiene la configuracion del parte del cliente, con providers que se encargan de la detección de cambios, el enrutamiento y la hidratación del cliente.
// El enrutamiento se hace del lado del cliente, la detección de cambios se hace del lado del cliente y la hidratación del cliente se hace del lado del cliente.

```



**provideClientHydration**: Es prioritario, digamos que hay una ruta como 

url -> google/about : Carga un componente, ese component pide informacion, con esa informacion va y carga la vista, que pasa si el cliente por alguna razon cambia dicha informacion, si no esta **provideClientHydration**, cada vez que esta informacion cambie  se renderiza entero. Entonces **provideClientHydration** lo que nos permite es que solamente  se va renderizar aquellos nodos que se vean afectados por el cambio, esto hace que que tengo un mayor perfomance.





En localhost file se encuentra el codigo de Angular

![image-20240626000920258](/Users/naorojas/Library/Application Support/typora-user-images/image-20240626000920258.png)
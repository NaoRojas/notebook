**Angular 18: Novedades y Mejoras**

Angular 18 es la mayor actualización desde Angular 17, trayendo muchas características interesantes. Aunque no soy un experto en Angular, aquí tienes un resumen de las novedades más destacadas:

1. **Soporte experimental a 'zoneless'**:
   - Angular finalmente permite eliminar la dependencia de Zone.js, que ha sido costosa y complicada durante mucho tiempo.
2. **RxJS Opcional**:
   - En su hoja de ruta, Angular planea hacer RxJS totalmente opcional, simplificando el desarrollo sin necesidad de aprender observables.
   - Esto fue anunciado en Google I/O y parece una mejora significativa.
3. **Signals**:
   - La mejor forma de utilizar componentes sin zonas es mediante Signals. Angular ha comenzado a usar Signals en sus aplicaciones, como YouTube.
   - Hay una API de Signals en vista previa para Signal inputs en Angular versión 17.1 y 17.2.
4. **Defers**:
   - Ahora, Angular permite cargar vistas solo cuando aparecen en el viewport, mejorando el rendimiento al evitar cargar contenido innecesario.
   - Esta característica está integrada de manera nativa en Angular.
5. **Material 3**:
   - El catálogo de componentes de Material Design para Angular es ahora estable, con una página mejorada y muchos componentes listos para usar.
6. **Partial Hydration**:
   - Similar a Quick, Angular implementa Partial Hydration, lo que permite hidratar la página de forma incremental conforme se necesite, mejorando el rendimiento.
   - Esta técnica permite renderizar en el servidor y solo inicializar eventos y ejecutar JavaScript en el cliente cuando sea necesario.
7. **Server-Side Rendering (SSR)**:
   - Mejora en la renderización en el servidor, permitiendo hacer replay de eventos y una experiencia de desarrollo más fluida.
8. **Mejoras en Developer Experience**:
   - Se han introducido mejoras en la experiencia de desarrollo y se están planeando más para partial hydration.

En cuanto a experiencias personales, recuerdo una vez que fui a una conferencia en Rusia en 2020. A raíz de eso, siempre que viajo a Estados Unidos, me preguntan sobre mi visa rusa, lo cual puede ser algo incómodo. La primera vez que me preguntaron, me metieron en una sala y me hicieron muchas preguntas, aunque al final todo salió bien. Ahora, cuando viajo, simplemente digo que fui a hacer turismo y eso facilita las cosas.

Espero que esta información te sea útil y te dé una mejor idea de las novedades en Angular 18.
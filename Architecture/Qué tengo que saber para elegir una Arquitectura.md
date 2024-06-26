# Qué tengo que saber para elegir una Arquitectura ? 

Tenemos una aplicacion bancaria con las siguientes características:

- Solmanete se ejecuta en el browser, no tenemos una aplicacion Nativa
- Es privada (**A public app can be accessed by anyone in both edit and run mode, while a private app is protected and the editor is only accessible to you and your collaborators**.)
- Buena Latencia (Rapida)
- Se puede consultar/ejecutar offline

Q: **Tecnologias que se pueden utilizar?**

1. NextJS(SSG)

2. Zustand, 

3. SCSS

R/ Primero, SSG quiere decir que toda la app va a hacer un pre-fetching del lado del servidor, y el user lo que hace es traerse la app entera y usarla. Para cosas que son muy dinámicas como por ejemplo, quiero consultar el estado de mi cuenta, eso ya hago un llamdo a un API y se trae nueva información, en resumen va a haber intercaccion en esa pagina, mucha interaccion con el user, entonces SSG no nos sirve. Ademas de que no nos sirve, el tema de Zustand es una tecnología que es para manejo de estada de la aplicacion y donde corre en el lado del cliente, ose en el Client Side Renderin (CSR), en resumen no se puede utilizar con SSG.

Siguiente que tenemos que ver antes de elegir una arquitectura?

- Quality Goals, en resumen los requerimientos
- Requiriementos Funcionales y Requerimientos No Funcionales
- Performance/ Eficiencia

## Google Web Vitals

Metricas que se utilizan para guiar a los developers en la peromance, la eficiencia de la pagina y SEO

- **TTFB: Time To First Byte**: the time taken to receive the first byte
- **FID: First Contentful Paint:** the time taken before the browser renders the first content
- **LCP Largest Contentful Paint**:  reports the render time of the largest image or block of text visible within the browser viewport; typically a banner, hero image, or heading. LCP selects one of the following elements:
- **FID First Input Delay**:  How quickly does the page respond to user actions such as clicking, tapping, scrolling, etc?
- **TTI Time To Interactive**: The time at which the page is capable of responding to user input. TTI is presumed when no long-running tasks are active and fewer than three HTTP requests are yet to complete.
- **CLS Cumulative Layout Shift**:  measures visual stability. Does content move unexpectedly on the page?.The metric calculates a score when elements move without warning or interaction. This often occurs when you’re reading an article on a mobile device and the content jumps off-screen. It can be caused when an image or advertisement loads above the current scroll position. In extreme cases, it can make you click the wrong link or control.


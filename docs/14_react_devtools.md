---
title: React DevTools
layout: default
nav_order: 14
---

## ¿Qué es?

React devtools es una extensión oficial que nos brinda React como una serie de herramientas para apoyar al desarrollo y con el proceso de debug.

Esta extensión se utiliza a partir de la instalación en el navegador. Al añadirla vamos a tener disponibles las ventanas de _Components_ y _Profiler_ en las DevTools del navegador, que nos permiten explorar el **árbol de componentes** de la aplicación de react en la que estemos trabajando, ver los props y el estado actual de cada uno de los componentes. Incluso nos permite editar valores en tiempo real para probar distintos comportamientos.

![react-devtools-change-props.gif](/assets/img/14/react-devtools-change-props.gif)

Además, brinda herramientas avanzadas para **medir** y **optimizar el rendimiento** a partir de su panel _Profiler_ en donde se pueden ver gráficos de tiempo de renderizado de cada componente, ayudando a identificar cuellos de botella.

React DevTools está diseñada para facilitar la detección de errores complejos sin depender únicamente de `console.log`. Puede ser de gran ayuda para depurar más rápido y comprender mejor lo que ocurre en una aplicación.

## ¿Por qué React DevTools?

- **Depuración más rápida**: Al Inspeccionar directamente los componentes activos del árbol de componentes es posible evitar tener que agregar varios logs manuales.
- **Visión completa del estado de la UI**: Con DevTools puedes ver de un vistazo las props, estado local y contexto de cualquier componente del árbol.
- **Detección de renders innecesarios:** Otra de las vistas que brindan gran apoyo al momento de evaluar cómo se está comportando nuestra aplicación es posible ver el proceso de re-renderizados de nuestros componentes a través de la opción _Highlight updates when components render_. Al activar esta opción se muestran los límites de los componentes que se dibujan en cada actualización del estado.
- **Perfilado de rendimiento**: El panel **Profiler** registra cada renderizado de la pp. En él aparecen commits con barras que muestra cuánto tardó cada render y un _flame chart_ detallado de cada componente. De esta manera es posible identificar los componentes que consumen más tiempo, los componentes lentos o intensivos de procesamiento.
- **Facilita debugging de hooks y contexto:** Además de props/estado muestra el estado de los Hooks como `useState` o `useContext` lo que

## Instalación y configuración

Para usar React DevTools, instala primero la extensión en tu navegador de desarrollo:

**Chrome/Firefox/Edge:** Busca **“React Developer Tools”** en la tienda de extensiones. Una vez instalada, al abrir DevTools verás dos pestañas nuevas: **Components** y **Profiler**.

**Safari y otros navegadores:** Instala la versión global para correrlo localmente:

```bash
npm install -g react-devtools
```

Luego lanza la aplicación de DevTools con el comando `react-devtools`. Esto abrirá una ventana independiente que se conectará a tu app React. Para ello, agrega en tu HTML (al inicio del `<head>`) la etiqueta:

```html
<script src="http://localhost:8097"></script>
```

y recarga la página. De este modo DevTools se vincula con tu aplicación aunque no sea un navegador Chrome/Firefox.

> **Configuración adicional:** Asegúrate de ejecutar tu app en modo desarrollo (`NODE_ENV=development`), ya que en producción el profiling está desactivado por defecto. Actualiza React DevTools regularmente para aprovechar las mejoras. Opcionalmente, puedes instalar la extensión **React Developer Tools** del Chrome Web Store

Tras la instalación, abre las DevTools (F12) mientras navegas tu aplicación React. Verás las pestañas **Components** y **Profiler** junto a las pestañas habituales del navegador.

## Uso práctico

### 1- Inspeccionar componentes, props y estado

![react-devtools-components-tab.png](/assets/img/14/react-devtools-components-tab.png)
_En la izquierda se muestra el árbol de componentes de la aplicación, y a la derecha los detalles del componente seleccionado (props, hooks, etc.)._

Para inspeccionar un componente, abre DevTools y elige la pestaña **Components**. Allí aparece la jerarquía React de tu página. Puedes expandir nodos para navegar por el árbol (app ➔ hijos ➔ etc.). Selecciona cualquier componente y en el panel de la derecha verás sus **props** y **state** actuales. También verás una sección de _hooks_ donde se listan los estados de cada hook de ese componente. Por ejemplo:

```javascript
function ExampleComponent({ name }) {
	const [message, setMessage] = useState('Hello react-devtools');
	// ...
	return (
		<p>
			{message}, {name}!
		</p>
	);
}
```

Si seleccionamos `<ExampleComponent>` en React DevTools, veremos `props: { name: "Carlos" }` y `state: { message: "Hello react-devtools" }`. Incluso podemos editar estos valores directamente en la pestaña para probar cómo responde la UI.

Además, la pestaña **Components** permite buscar por nombre de componente (barra de búsqueda superior) y resalta en el DOM el elemento asociado al componente (al estilo “Inspect element”). Así, React DevTools actúa como inspector de elementos orientado a React. Esta capacidad de ver y modificar **en tiempo real** el estado y props facilita muchísimo el debugging, por ejemplo para entender por qué falla un render.

### 2- Detectar renders innecesarios

Es común que componentes se actualicen cuando no deberían, provocando renderizados extra innecesarios. React DevTools ofrece ayudas visuales para detectar esto como:

- **Resaltar re-renders:** En la configuración de DevTools (icono de engranaje en la pestaña **Components**), activa _“Highlight updates when components render”_. Al hacer cambios en tu app, DevTools resaltará con colores los componentes que se re-renderizan. Si un componente parpadea al cambiar otro, sabrás que se re-renderizó innecesariamente.

![react-devtools-highlight-screenshot.png](/assets/img/14/react-devtools-highlight-screenshot.png)
_Con esta opción React DevTools nos mostrará los elementos que se renderizan en cada momento que se realiza algún cambio de manera visual agregando un contorno en los elementos que se re-renderizan_

### 3- Perfilado de rendimiento (Profiler)

![react-devtooks-profiling-screenshot.png](/assets/img/14/react-devtooks-profiling-screenshot.png)

- El panel **Profiler** mide tiempos de renderizado. Para usarlo, debemos ir a la pestaña **Profiler** y hacer clic en el botón de grabar (círculo azul).
- Luego, al interactuar con la aplicación (por ejemplo, escribiendo en inputs, clicks, etc) para generar renders.
- Al terminar hacer click en Stop.

Al terminar se mostrará en el panel lateral un gráfico de barras con cada render registrado. Cada barra representa un ciclo completo de renderizado. Su altura y color indican cuánto tardó ese render en particular (bandas amarillas y altas son las más costosas).

Al insepccionar en cada uno de los renders es posible examinar los detalles de cada commit. En la parte inferior tendremos una vista de **flame chart** donde cada barra corresponde a un componente y su subárbol. En este caso el ancho de la barra indica el tiempo de render de ese componente (y sus hijos) en ese commit.

En la imagen siguiente, por ejemplo, vemos que el componente _Router_ tardó 18.4 ms en un commit concreto. Este nivel de detalle te permite identificar precisamente qué componentes consumen más tiempo: “El flame chart representa el estado de la aplicación en un commit; cada barra corresponde a un componente, y su ancho muestra cuánto tardó en renderizarse”

![react-devtools-flame-chart.png](/assets/img/14/react-devtools-flame-chart.png)

Por más detalle dirigirse a [Herramientas de Desarrollo de React – React](https://es.react.dev/learn/react-developer-tools)

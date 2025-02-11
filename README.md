# POC - Cerrar Ventanas Hijas en JavaScript

A continuación se presenta la documentación técnica de la solución implementada para gestionar ventanas secundarias desde una página principal, utilizando el mecanismo de **postMessage()** para la comunicación entre ventanas.

---

# Documentación Técnica: Gestión de Ventanas Secundarias con postMessage

## 1. Resumen

La solución consiste en un sistema en el que la **página principal** es responsable de generar (abrir) ventanas secundarias, registrar dichas ventanas mediante un identificador único (`windowId`) y, posteriormente, enviarles órdenes de cierre. La comunicación entre la página principal y las ventanas secundarias se realiza mediante el método `postMessage()`, garantizando la seguridad a través de la verificación del origen (`location.origin`). Además, se implementa la lógica para remover el listener de mensajes cuando no existan ventanas secundarias abiertas, evitando recursos o listeners innecesarios.

---

## 2. Estructura de la Solución

La solución se divide en dos archivos principales:

- **main.html**  
  Página principal que:
  - Genera (abre) ventanas secundarias.
  - Registra las ventanas secundarias que se abren, mediante la recepción de un mensaje con un `windowId`.
  - Permite enviar órdenes de cierre a ventanas secundarias específicas, identificadas por su `windowId`.
  - Gestiona el registro y la eliminación del listener de mensajes para evitar listeners activos innecesarios cuando no hay ventanas abiertas.

- **secondary.html**  
  Página secundaria que:
  - Al cargarse, genera un identificador único (`windowId`).
  - Muestra su `windowId` en pantalla.
  - Notifica a la página principal su apertura mediante `postMessage()`, enviando su `windowId`.
  - Escucha el mensaje de cierre (`close-window`) y se cierra a sí misma al recibir dicho mensaje.

---

## 3. Flujo de Ejecución

### 3.1. Registro de la Ventana Secundaria

1. **Generación del `windowId`:**  
   Al cargarse, la página secundaria genera un identificador único (por ejemplo, utilizando `Date.now().toString()`).

2. **Notificación a la Página Principal:**  
   La secundaria, utilizando `window.opener.postMessage()`, envía un mensaje con la siguiente estructura:
   ```js
   { type: 'secondary-open', windowId: windowId }
   ```
   Esto permite a la página principal registrar la ventana secundaria.

3. **Registro y Actualización del DOM:**  
   La página principal, a través del listener de mensajes, verifica el origen del mensaje y, si coincide, almacena la referencia de la ventana (obtenida en `event.source`) en un objeto (`ventanasSecundarias`) utilizando el `windowId` como clave. Además, actualiza la interfaz (una lista) para mostrar los `windowId` de las ventanas abiertas.

### 3.2. Envío de Orden de Cierre

1. **Entrada del `windowId` a Cerrar:**  
   La página principal cuenta con un campo de texto en el que se ingresa el `windowId` de la ventana secundaria que se desea cerrar.

2. **Envió del Mensaje de Cierre:**  
   Al pulsar el botón de “Cerrar Ventana”, se envía un mensaje a la ventana secundaria con el siguiente objeto:
   ```js
   { type: 'close-window' }
   ```
   Utilizando la referencia almacenada en `ventanasSecundarias` y verificando el mismo origen.

3. **Cierre de la Ventana Secundaria:**  
   La página secundaria, al recibir el mensaje de cierre, verifica el origen y ejecuta `window.close()`, cerrándose a sí misma.

### 3.3. Gestión del Listener de Mensajes

- **Registro del Listener:**  
  Al abrir una ventana secundaria, se verifica si el listener de mensajes ya está registrado; si no lo está, se registra.

- **Eliminación del Listener:**  
  Cuando se cierra una ventana secundaria y, tras eliminar su referencia, se comprueba que no queden ventanas registradas, se remueve el listener de mensajes. Esto evita que se mantenga un listener activo innecesariamente cuando no hay ventanas abiertas.

---

## 4. Requisitos y Consideraciones

- **Mismo Origen:**  
  La prueba de concepto asume que ambos archivos se sirven desde el mismo origen, por lo que se utiliza `location.origin` en la verificación del origen de los mensajes. En entornos de producción, es necesario ajustar los orígenes según la configuración real.

- **Uso de postMessage:**  
  Se utiliza `postMessage()` para la comunicación entre ventanas. Es fundamental especificar un `targetOrigin` correcto para evitar vulnerabilidades de seguridad.

- **Limpieza de Listeners:**  
  La solución implementa la lógica para remover el listener de mensajes cuando no hay ventanas secundarias abiertas, liberando recursos y evitando posibles efectos colaterales.

- **Generación de `windowId`:**  
  Se utiliza una técnica sencilla (por ejemplo, `Date.now().toString()`) para generar un identificador único. En aplicaciones reales, se puede optar por métodos más robustos si es necesario.

---

## 5. Código de Referencia

### 5.1. main.html

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Página Principal</title>
  <style>
    body { font-family: Arial, sans-serif; }
    .registered-windows { margin-top: 20px; }
    .close-window { margin-top: 20px; }
  </style>
</head>
<body>
  <h1>Página Principal</h1>
  
  <!-- Botón para generar (abrir) una ventana secundaria -->
  <button id="abrirSecundaria">Abrir Ventana Secundaria</button>
  
  <!-- Sección para listar las ventanas secundarias registradas -->
  <div class="registered-windows">
    <h3>Ventanas secundarias registradas:</h3>
    <ul id="listaVentanas"></ul>
  </div>
  
  <!-- Sección para enviar la orden de cierre -->
  <div class="close-window">
    <h3>Cerrar ventana secundaria</h3>
    <input type="text" id="windowIdToClose" placeholder="Ingrese windowId">
    <button id="cerrarVentana">Cerrar Ventana</button>
  </div>

  <script>
    // Objeto para almacenar las referencias a las ventanas secundarias registradas
    const ventanasSecundarias = {};

    // Bandera para saber si el listener está registrado
    let listenerRegistrado = false;

    // Función que maneja los mensajes entrantes
    function handleMessage(event) {
      // Verificar el origen; en esta prueba usamos location.origin
      if (event.origin !== location.origin) return;
      
      console.log("Mensaje recibido en la página principal:", event.data);
      
      // Se espera recibir: { type: 'secondary-open', windowId: 'valor_unico' }
      if (event.data && event.data.type === 'secondary-open' && event.data.windowId) {
        // Registrar la ventana secundaria usando event.source
        ventanasSecundarias[event.data.windowId] = event.source;
        
        // Actualizar la lista en el DOM para visualizar el windowId registrado
        const li = document.createElement('li');
        li.textContent = `windowId: ${event.data.windowId}`;
        li.id = 'ventana-' + event.data.windowId;
        document.getElementById('listaVentanas').appendChild(li);
      }
    }

    // Función para registrar el listener de mensajes si aún no está agregado
    function registrarListener() {
      if (!listenerRegistrado) {
        window.addEventListener('message', handleMessage);
        listenerRegistrado = true;
        console.log('Listener de mensajes registrado.');
      }
    }

    // Función para remover el listener si no hay ventanas secundarias abiertas
    function removerListenerSiNoHayVentanas() {
      if (Object.keys(ventanasSecundarias).length === 0 && listenerRegistrado) {
        window.removeEventListener('message', handleMessage);
        listenerRegistrado = false;
        console.log('No hay ventanas abiertas, se removió el listener de mensajes.');
      }
    }

    // Al hacer clic en el botón "Abrir Ventana Secundaria"
    document.getElementById('abrirSecundaria').addEventListener('click', () => {
      // Asegurarse de que el listener esté registrado
      registrarListener();

      // La página principal genera la ventana secundaria.
      // Al no especificar dimensiones, se abrirá en una nueva pestaña o en el marco normal del navegador.
      window.open('secondary.html', '_blank');
      // La ventana secundaria se encargará de notificarse a sí misma.
    });

    // Al hacer clic en el botón "Cerrar Ventana"
    document.getElementById('cerrarVentana').addEventListener('click', () => {
      const windowId = document.getElementById('windowIdToClose').value.trim();
      if (ventanasSecundarias[windowId]) {
        // Enviar mensaje a la ventana secundaria para que se cierre
        ventanasSecundarias[windowId].postMessage({ type: 'close-window' }, location.origin);
        
        // Eliminar la referencia y actualizar la lista en el DOM
        delete ventanasSecundarias[windowId];
        const li = document.getElementById('ventana-' + windowId);
        if (li) li.remove();
        console.log(`Se ha enviado la orden de cierre a la ventana con ID: ${windowId}`);

        // Remover el listener si ya no quedan ventanas registradas
        removerListenerSiNoHayVentanas();
      } else {
        console.log(`No se encontró una ventana registrada con windowId: ${windowId}`);
      }
    });
  </script>
</body>
</html>
```

### 5.2. secondary.html

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Página Secundaria</title>
  <style>
    body { font-family: Arial, sans-serif; }
  </style>
</head>
<body>
  <h1>Página Secundaria</h1>
  <!-- Se muestra el windowId generado -->
  <p id="info"></p>
  
  <script>
    // Generar un identificador único para esta ventana secundaria
    const windowId = Date.now().toString();
    // Mostrar el windowId en pantalla
    document.getElementById('info').textContent = 'Mi windowId es: ' + windowId;

    // Notificar a la página principal (window.opener) que esta ventana está abierta,
    // enviando el windowId para su registro.
    if (window.opener) {
      window.opener.postMessage({ type: 'secondary-open', windowId: windowId }, location.origin);
      console.log('Notificación enviada a la página principal con windowId:', windowId);
    }

    // Escuchar mensajes entrantes para detectar la orden de cierre
    window.addEventListener('message', (event) => {
      // Verificar el origen; usamos location.origin en esta prueba
      if (event.origin !== location.origin) return;
      
      console.log("Mensaje recibido en la secundaria:", event.data);
      if (event.data && event.data.type === 'close-window') {
        console.log('Orden de cierre recibida, cerrando la ventana...');
        window.close();
      }
    });
  </script>
</body>
</html>
```

---

## 6. Instrucciones de Uso

1. **Preparación del Entorno:**  
   - Inicia un servidor local (por ejemplo, utilizando `http-server`, Live Server de VSCode u otro método) para servir ambos archivos desde el mismo origen.

2. **Acceso a la Página Principal:**  
   - Abre `main.html` en tu navegador.

3. **Generación de Ventanas Secundarias:**  
   - Haz clic en el botón **"Abrir Ventana Secundaria"**.  
   - Se abrirá una nueva pestaña con `secondary.html`, la cual mostrará su `windowId` y enviará un mensaje a la página principal para su registro.

4. **Visualización del Registro:**  
   - En la página principal, se mostrará en una lista el `windowId` de cada ventana secundaria abierta.

5. **Cierre de una Ventana Secundaria:**  
   - Ingresa en el campo de texto el `windowId` de la ventana secundaria que deseas cerrar.  
   - Haz clic en el botón **"Cerrar Ventana"**.  
   - La página principal enviará la orden de cierre a la ventana secundaria, que se cerrará al recibir el mensaje.  
   - Si ya no quedan ventanas abiertas, se removerá el listener de mensajes.

---

## 7. Consideraciones Finales

- **Seguridad:**  
  Es fundamental verificar el origen (`location.origin`) en ambos extremos para asegurar que la comunicación se realice únicamente entre fuentes confiables.

- **Limpieza de Recursos:**  
  La implementación se asegura de eliminar el listener de mensajes cuando no hay ventanas secundarias abiertas, evitando posibles fugas de memoria o comportamientos inesperados.

- **Escalabilidad:**  
  Aunque la prueba de concepto utiliza un identificador generado mediante `Date.now()`, en aplicaciones de mayor envergadura se recomienda utilizar métodos de generación de IDs más robustos para garantizar la unicidad y la trazabilidad.

Esta documentación proporciona una visión completa de la solución implementada, sus componentes y el flujo de trabajo, permitiendo su integración y adaptación en proyectos reales según las necesidades del sistema.

---

**Nota**: Este proyecto no requiere instalación ni dependencias adicionales. Solo abre el archivo `index.html` en tu navegador para probarlo.

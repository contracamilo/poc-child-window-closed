# README: Prueba de Concepto - Abrir y Cerrar Ventanas Hijas en JavaScript

Este proyecto es una prueba de concepto que demuestra cómo abrir múltiples ventanas hijas desde una página padre en un navegador y cómo cerrar cada ventana hija utilizando su título (`document.title`) como criterio. A continuación, se explica el funcionamiento del código y cómo probarlo.

---

## Funcionalidad Principal

1. **Abrir Ventanas Hijas**:
   - Desde la página padre, puedes abrir múltiples ventanas hijas haciendo clic en un botón.
   - Cada ventana hija se abre sin contenido adicional, solo con un título único generado automáticamente.

2. **Cerrar Ventanas Hijas**:
   - Por cada ventana hija abierta, se genera un botón en la página padre.
   - Al hacer clic en el botón, la ventana hija correspondiente se cierra.
   - El cierre se basa en el título de la ventana hija (`document.title`).

---

## Estructura del Proyecto

El proyecto consta de un único archivo HTML que contiene tanto la interfaz de usuario como el código JavaScript necesario para implementar la funcionalidad.

## Instrucciones para Probar el Proyecto

1. **Abrir la Página Padre**:
   - Abre el archivo `index.html` en un navegador web compatible (por ejemplo, Chrome, Firefox o Edge).

2. **Abrir Ventanas Hijas**:
   - Haz clic en el botón **Abrir Ventana Hija** en la página padre.
   - Se abrirá una nueva ventana hija vacía con un título único (por ejemplo, "Ventana Hija abc123").
   - Puedes abrir tantas ventanas hijas como desees.

3. **Cerrar Ventanas Hijas**:
   - Por cada ventana hija abierta, aparecerá un botón en la página padre con el texto "Cerrar [Título de la ventana hija]".
   - Haz clic en el botón correspondiente para cerrar la ventana hija.

---

## Explicación del Código

### 1. **Abrir Ventanas Hijas**:
   - Se utiliza `window.open()` para abrir una ventana hija vacía.
   - Se asigna un título único a la ventana hija usando `ventanaHija.document.title`.

### 2. **Generar Botones de Cierre**:
   - Por cada ventana hija abierta, se crea un botón en la página padre.
   - El botón tiene un evento `onclick` que cierra la ventana hija correspondiente usando `ventanaHija.close()`.
   - Después de cerrar la ventana, el botón se elimina del DOM.

### 3. **Criterio de Cierre**:
   - El cierre de la ventana hija se basa en la referencia a la ventana (`ventanaHija`), que está asociada con su título único.
   - No es necesario acceder directamente al `document.title` de la ventana hija para cerrarla, ya que la referencia a la ventana ya está disponible.

---

## Consideraciones Adicionales

- **Compatibilidad**:
  - Algunos navegadores pueden bloquear ventanas emergentes si no se activan mediante una acción del usuario (como un clic). Asegúrate de que el botón "Abrir Ventana Hija" esté correctamente configurado para evitar bloqueos.

- **Manejo de Ventanas Cerradas Manualmente**:
  - Si el usuario cierra manualmente una ventana hija, el botón de cierre correspondiente en la página padre no se eliminará automáticamente. Podrías agregar una verificación adicional para eliminar botones de cierre asociados a ventanas ya cerradas.

---

## Conclusión

Este proyecto es una demostración sencilla pero efectiva de cómo abrir y cerrar ventanas hijas desde una página padre utilizando JavaScript. Es ideal para entender conceptos básicos de manipulación del DOM y manejo de ventanas en el navegador.

¡Siéntete libre de modificar y expandir este código según tus necesidades! 😊

---

**Nota**: Este proyecto no requiere instalación ni dependencias adicionales. Solo abre el archivo `index.html` en tu navegador para probarlo.

# Tauros Barbería — Sistema de Agendamiento en Línea
### Presentación Técnica del Proyecto

---

## ¿Qué es el sistema?

Tauros Barbería cuenta hoy con un sistema de agendamiento en línea completamente funcional, desarrollado a medida. Los clientes pueden reservar su cita desde cualquier dispositivo, eligiendo barbero, fecha y horario disponible — sin llamadas, sin mensajes de WhatsApp, sin cruces de citas. Paralelamente, existe un panel de administración protegido desde donde se gestionan los barberos, se visualizan todas las citas y se controla la operación del negocio en tiempo real.

---

## Stack tecnológico utilizado

El proyecto se construyó deliberadamente con tecnologías ligeras y de alto rendimiento, evitando frameworks pesados o infraestructura costosa:

**Frontend — HTML, CSS y JavaScript puro**
No se usó ningún framework como React, Vue o Angular. Todo el sistema vive en dos archivos HTML autosuficientes: `index.html` (la página de reservas para clientes) y `tauros-admin.html` (el panel de administración). Esta decisión reduce la complejidad al mínimo, elimina dependencias de terceros y garantiza que el sitio cargue rápido en cualquier conexión.

**Base de datos — Supabase (PostgreSQL)**
Supabase actúa como backend completo. Provee una base de datos PostgreSQL con una API REST lista para usar, sin necesidad de escribir un servidor propio. Las tablas `barberos` y `citas` almacenan toda la información del negocio. Las políticas de Row Level Security (RLS) controlan exactamente qué operaciones puede hacer un cliente anónimo versus un administrador.

**Despliegue — Vercel**
El sitio está publicado en Vercel, conectado directamente al repositorio de GitHub. Cada vez que se hace un `git push`, Vercel detecta el cambio y despliega la nueva versión automáticamente en cuestión de segundos. No hay servidor que mantener, no hay costos de hosting significativos, y el sitio tiene disponibilidad global con CDN incluido.

**Seguridad — Web Crypto API (SHA-256)**
Para la autenticación del panel admin se usa la API nativa del navegador (`crypto.subtle`) para hashear contraseñas con SHA-256. Nunca se almacena ni se compara texto plano — el hash se guarda en `localStorage` y se compara con el hash de lo que el usuario ingresa. Incluye bloqueo tras 5 intentos fallidos y expiración automática de sesión tras 2 horas de inactividad.

---

## Cómo se desarrolló

El desarrollo se realizó de forma iterativa, construyendo primero la estructura base y luego refinando funcionalidad a funcionalidad. El flujo de trabajo fue:

1. Se definió el flujo de reserva en 4 pasos: selección de silla → fecha → horario → confirmación.
2. Se configuró Supabase con las tablas necesarias y las políticas de acceso.
3. Se conectó el frontend con la API de Supabase usando `fetch` nativo, sin librerías adicionales.
4. Se integró Vercel con el repositorio de GitHub para despliegue continuo.
5. Se iteraron mejoras de UX, validaciones y seguridad hasta el estado actual.

El desarrollo completo, incluyendo depuración y mejoras, se realizó en colaboración con inteligencia artificial (Claude de Anthropic), lo que aceleró significativamente el proceso al poder identificar errores, proponer soluciones y escribir código en tiempo real.

---

## Problemas que se presentaron

El desarrollo no estuvo exento de obstáculos. Los más relevantes desde el punto de vista técnico:

**1. Sintaxis inválida en JavaScript (el bug más crítico)**
El mayor problema del proyecto fue un error sutil pero devastador: una función del frontend usaba *template literals anidados* (backticks dentro de backticks) para construir HTML dinámico. En JavaScript esto es sintaxis inválida, y el resultado fue que el motor del navegador rechazaba el bloque `<script>` completo sin mostrar ningún error visible. La consecuencia: la página cargaba pero se quedaba congelada en el loader indefinidamente, sin que los barberos o las sillas aparecieran jamás. El diagnóstico requirió revisar el código línea por línea hasta encontrar la causa raíz. La solución fue reescribir esas funciones usando concatenación de strings con un bucle `for`, eliminando los templates anidados por completo.

**2. Políticas de Row Level Security en Supabase**
Supabase protege las tablas con RLS por defecto, lo que significa que ninguna operación es permitida a menos que exista una política explícita que la autorice. Faltaban políticas para el rol `anon` (usuario no autenticado) en operaciones de INSERT y UPDATE, lo que hacía que las reservas de clientes fallaran silenciosamente. Hubo que diagnosticar qué políticas faltaban y crearlas con precisión para no abrir permisos innecesarios.

**3. Conflictos en Git durante el despliegue**
En varias ocasiones al intentar hacer `git push`, el repositorio remoto tenía cambios que no estaban en el local, generando conflictos. Esto requirió manejar merges con estrategia explícita (`git pull origin main -X ours`) para preservar los cambios locales. En un momento el sistema abrió el editor Vim para confirmar el merge y el usuario no sabía cómo salir — se resolvió con `:wq`.

**4. Manejo de valores nulos en la base de datos**
El campo `disponible` de la tabla `barberos` podía contener `null` (no establecido), `true` o `false`. El código original trataba `null` como `false` usando `!b.disponible`, lo que marcaba como "no disponible" a barberos recién creados que nunca habían tenido ese campo definido. Se corrigió a una comparación estricta `b.disponible === false`.

---

## ¿Qué tan fácil fue el proceso?

Técnicamente el proyecto tiene un nivel de complejidad medio. No es trivial — involucra integración con una base de datos externa, manejo de autenticación, lógica de disponibilidad de horarios, despliegue en la nube y consideraciones de seguridad. Sin embargo, la elección del stack simplificó enormemente la ejecución: no hay servidor propio que configurar, no hay framework que aprender, no hay proceso de build complicado.

Lo que más consumió tiempo no fue escribir código nuevo, sino diagnosticar el bug de los backticks anidados — un error de una sola línea que bloqueó toda la funcionalidad visible. En desarrollo de software esto es común: los errores más difíciles de encontrar no son los que lanzan mensajes de error obvios, sino los que fallan silenciosamente.

Una vez identificado el problema raíz, las correcciones y mejoras posteriores fluyeron con rapidez. La integración entre Supabase, Vercel y GitHub funciona de forma muy fluida una vez configurada — un `git push` es todo lo que se necesita para publicar un cambio al mundo.

---

## Resultado final

El sistema hoy en producción permite:

- Reservas en línea en menos de 2 minutos, desde cualquier dispositivo
- Selección visual de silla y barbero con estado de disponibilidad en tiempo real
- Bloqueo automático de domingos y fechas pasadas para evitar errores del cliente
- Teléfono de contacto obligatorio en cada reserva
- Panel admin con visualización completa de citas, gestión de barberos y estadísticas del día
- Contraseña del admin protegida con hashing criptográfico, bloqueo por intentos fallidos y expiración de sesión
- Despliegue continuo: cualquier mejora futura se publica con un solo comando

---

*Desarrollado con HTML, CSS, JavaScript · Supabase · Vercel · GitHub*
*Asistido por Claude (Anthropic)*

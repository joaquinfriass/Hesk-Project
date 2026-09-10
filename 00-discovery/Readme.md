# 00 — Discovery

## 1. Definir el contexto del producto

### **Problema y necesidad que resuelve HESK**

Un servicio de mesa de ayuda (*help desk*) como HESK resuelve la **desorganización, dispersión y falta de seguimiento** de las solicitudes de atención o soporte que de otro modo llegarían por canales no estructurados (como correos individuales o mensajes verbales).
En términos de comportamientos observables, HESK **centraliza las peticiones entrantes convirtiéndolas en tickets** con un identificador único de seguimiento (**`trackid`**), les asigna una categoría, prioridad y estado inicial, registra todo el historial de la conversación con sus adjuntos y ofrece un portal de autoservicio para resolver dudas recurrentes.

### **Actores, comportamientos observables y resultados esperados**

**1. Cliente / Solicitante (con o sin cuenta registrada)**

- **Comportamientos observables:** Navega por la Base de Conocimiento (KB) pública buscando artículos; completa el formulario web de alta de ticket (o envía un correo) aportando datos, asunto, mensaje y adjuntos; consulta el estado del ticket ingresando su identificador de seguimiento (*tracking ID*) y correo (o autenticándose en su cuenta); y añade respuestas en texto plano.
- **Resultado esperado:** Obtener la confirmación del registro del ticket, recibir respuestas claras a su problema dentro de un plazo predecible y poder verificar el progreso de su solicitud de forma transparente

**2. Agente de Soporte (*Staff*)**

- **Comportamientos observables:** Inicia sesión en el panel administrativo, filtra las bandejas por categoría o estado, toma o se asigna tickets, reclasifica prioridad o vencimiento, redacta respuestas usando texto enriquecido o plantillas predefinidas, agrega notas internas no visibles para el usuario y cambia el estado del ticket a resuelto o cerrado.
- **Resultado esperado:** Disponer de una interfaz ordenada que le permita gestionar y priorizar su carga de trabajo diaria, comunicarse eficientemente con el cliente.

**3. Administrador del Sistema**

- **Comportamientos observables:** Define la estructura operativa configurando categorías, estados, prioridades, campos personalizados, reglas de autoasignación y límites de adjuntos; administra usuarios *staff*, grupos de permisos y cuentas de clientes; y consulta o exporta reportes analíticos de rendimiento.
- **Resultado esperado:** Garantizar la gobernanza de la plataforma, la correcta aplicación de las políticas de acceso/seguridad y contar con visibilidad operativa sobre el volumen de atención y desempeño del equipo.

**4. Procesos del Sistema / Tareas Programadas (*Cron* y Correo)**

- **Comportamientos observables:** Procesan el envío de notificaciones automáticas por SMTP, reciben o parsean correos entrantes (IMAP/POP3/piping) y ejecutan scripts periódicos para identificar e informar sobre tickets vencidos.
- **Resultado esperado:** Ejecutar las tareas de integración y temporización en segundo plano de manera limpia, sin generar bucles, registros duplicados ni pérdidas de información.

### **Información que atraviesa todo el ciclo de vida del ticket**

Para que la solicitud mantenga su trazabilidad desde la apertura hasta el cierre, los siguientes datos deben persistir y acompañar al ticket en todo momento:

- **Identificación y propiedad:** Identificador de seguimiento público (**`trackid`**), ID interno secuencial, e identidad/contacto del cliente (nombre y correo electrónico).
- **Clasificación y estado:** Categoría asignada, prioridad (Baja, Media, Alta, Crítica) y estado actual (Nuevo, En progreso, Esperando respuesta, Respondido, En espera, Resuelto).
- **Responsabilidad:** Agente propietario (*owner*), colaboradores asignados y seguidores.
- **Contenido funcional:** Asunto, mensaje inicial, valores de campos personalizados (**`customN`**) y archivos adjuntos vinculados.
- **Historial conversacional:** Conjunto cronológico de respuestas enviadas tanto por el cliente como por el *staff* (**`replies`**).
- **Trazabilidad temporal y auditoría:** Fecha de creación (**`dt`**), fecha de última modificación (**`lastchange`**), fecha de cierre (**`closedat`**), fecha límite de vencimiento (**`due_date`**), usuario de apertura/cierre, tiempo trabajado y registro de cambios (*history*)

### **Componentes observables vs. componentes ocultos del producto**

| **Dimensión** | **Componentes Observables (Interfaz / Frontend)** | **Componentes Ocultos (Backend / Persistencia / Sistema)** |
| --- | --- | --- |
| **Portal del Cliente** | Formulario público de alta de tickets, pantalla de consulta por *tracking ID* + correo, portal de inicio de sesión de clientes y visor de artículos de la Base de Conocimiento pública. | Sanitización del contenido ingresado (HTMLPurifier), validación de tokens anti-CSRF, verificación de captcha/antispam y lógica PHP de manejo del formulario (**`submit_ticket.php`**). |
| **Atención de Agentes** | Consola administrativa (**`/admin`**), bandejas de entrada, vistas de detalle de tickets, editores WYSIWYG (TinyMCE) y pantallas de reportes. | **Notas internas:** Comentarios e información técnica añadidos por el *staff* que están restringidos por lógica de negocio y jamás se muestran en la interfaz del cliente. |
| **Base de Datos y Datos** | Listados tabulares de registros, conteo de respuestas y visualización de adjuntos descargables. | **Modelo MySQL/MariaDB:** Estructura subyacente de 40 tablas relacionales (**`hesk_tickets`**, **`hesk_replies`**, **`hesk_users`**, etc.), consultas SQL dinámicas y almacenamiento físico de archivos en el servidor (**`/attachments`**). |
| **Procesos y Configuración** | Formularios de ajustes generales, menús de gestión de permisos y botones de ejecución. | Scripts ejecutados por *Cron* (**`/cron/email_overdue_tickets.php`**), manejo de conexiones de correo (SMTP/IMAP/POP3), claves secretas en **`hesk_settings.inc.php`** y archivos de caché generados en **`/cache`**. |

## 2. Identificar actores y permisos

**1. Actores y Roles en HESK (Objetivos, Acciones Visibles y Restricciones)**

- **Visitante (Anónimo / Público)**
    - **Objetivos:** Resolver dudas mediante autoservicio (**Base de Conocimiento**) o tomar contacto con soporte sin registrarse.
    - **Acciones visibles:** Consultar artículos y categorías de la KB pública, y crear un ticket web (si la política de la instancia lo habilita).
    - **Restricciones conocidas:** No puede acceder al área administrativa, ver la KB privada, borradores ni tickets pertenecientes a otros usuarios.
- **Cliente sin cuenta**
    - **Objetivos:** Crear solicitudes puntuales y dar seguimiento a su resolución.
    - **Acciones visibles:** Completar el formulario de alta, acceder al seguimiento de un ticket proporcionando su identificador único (*tracking ID*) y correo electrónico, y agregar respuestas o adjuntos.
    - **Restricciones conocidas:** Su acceso está limitado al ticket vinculado a su combinación de correo y *tracking ID*; no posee un panel centralizado a menos que se habiliten las cuentas de cliente.
- **Cliente registrado**
    - **Objetivos:** Gestionar de forma centralizada sus tickets, consultar su historial y administrar su perfil de usuario.
    - **Acciones visibles:** Iniciar sesión (con MFA si se exige), consultar la lista de tickets propios o donde figura como participante, responder, agregar adjuntos y actualizar sus datos de perfil.
    - **Restricciones conocidas:** Restringido únicamente a sus datos e información donde figura como participante (**`ticket_to_customer`**); no tiene visibilidad de notas internas, acciones del *staff*  ni consola administrativa **`/admin`**.
- **Agente de Soporte (*Staff*)**
    - **Objetivos:** Atender, clasificar, responder, reasignar y cerrar las solicitudes asignadas dentro de su alcance operativo.
    - **Acciones visibles:** Acceder a la consola **`/admin`**, visualizar bandejas filtradas, responder tickets con texto enriquecido, agregar notas internas, cambiar estado/prioridad/vencimiento, tomar/asignar tickets, usar respuestas predefinidas, registrar tiempo trabajado y gestionar la KB (si tiene el permiso).
    - **Restricciones conocidas:** Limitado estrictamente a las categorías autorizadas, las funciones/privilegios concedidos (**`heskprivileges`**) y los grupos de permisos a los que pertenece. No puede responder tickets directamente por correo saliente (debe usar la interfaz web por seguridad).
- **Colaborador (*Staff Colaborador*)**
    - **Objetivos:** Asistir en la resolución de tickets específicos donde fue incluido sin ser el propietario principal (*owner*).
    - **Acciones visibles:** Ver el detalle del ticket asignado como colaborador, agregar respuestas y notas privadas, e interactuar según las funciones permitidas.
    - **Restricciones conocidas:** No puede alterar la propiedad (*ownership*) ni los permisos del ticket sin privilegios explícitos para ello.
- **Administrador del Sistema**
    - **Objetivos:** Gobernar y configurar la plataforma, administrar el personal, velar por la seguridad y analizar métricas globales.
    - **Acciones visibles:** Acceso integral a todas las categorías, funciones administrativas, configuración general, servidor de correo, usuarios *staff*, grupos de permisos, campos personalizados, estados, prioridades, reportes y herramientas.
    - **Restricciones conocidas:** Aunque posee acceso total a la consola, requiere reautenticación para páginas sensibles y sus acciones están sujetas a validaciones de seguridad (como tokens anti-CSRF) y auditoría.
- **Procesos y Roles de Sistema / Infraestructura**
    - **Servicio de correo / Integraciones (SMTP / IMAP / POP3 / Piping):** Transporta notificaciones e ingesta mensajes para convertirlos en tickets o respuestas. *Restricciones:* Protegido mediante clave de acceso URL; no posee interacción directa con la interfaz web.
    - **Cron / Planificador de tareas:** Ejecuta la notificación periódica de tickets vencidos (**`email_overdue_tickets.php`**) y la recolección de correo entrante. *Restricciones:* Ejecución en segundo plano restringida por claves URL o CLI para evitar ejecuciones simultáneas o no autorizadas.
    - **Operador de infraestructura:** Mantiene la disponibilidad del servidor web, PHP, MySQL, backups y certificados SSL. *Restricciones:* No requiere ni debe poseer privilegios funcionales dentro del help desk.

---

**2. Respuestas a las Preguntas de Análisis**

**¿Qué puede ver y modificar cada rol?**

- **Visitante / Cliente:** Ve la KB pública y la pantalla de sus tickets individuales. Modifica únicamente la conversación de su ticket agregando mensajes y adjuntos.
- **Agente (Staff):** Ve las bandejas y tickets de las categorías que se le han asignado. Modifica estado, prioridad, propietario, colaboradores, notas privadas, tiempo trabajado y categorías. Si se le otorga la función, puede modificar artículos de la KB.
- **Administrador:** Ve la totalidad de datos, reportes y herramientas del sistema. Modifica ajustes globales, parámetros de infraestructura, perfiles de *staff*, grupos de permisos, categorías, campos personalizados y catálogos de negocio.

**¿Qué acciones cambian datos?**

- **Operaciones de tickets:** Creación (**`submit_ticket.php`**), respuesta (**`reply_ticket.php`**), adición de notas internas (**`notes`**), reclasificación de categoría/prioridad/vencimiento, asignación de propietario/colaboradores, fusión (**`merged`**), vinculación o eliminación.
- **Gestión de usuarios y accesos:** Creación/edición de cuentas *staff* (**`manage_users.php`**), modificación de grupos de permisos (**`manage_permission_groups.php`**), asignación de categorías y gestión de clientes (**`manage_customers.php`**).
- **Configuración y catálogos:** Alteración de ajustes del sistema (**`hesk_settings.inc.php`**), creación de campos personalizados (**`custom_fields`**), estados/prioridades personalizados y publicación o borrado de artículos en la KB (**`knowledgebase.php`**).

**¿Qué segregación de funciones debería existir?**

- **Separación Agente vs. Administrador:** Los agentes de soporte no deben tener capacidad para administrar usuarios, modificar permisos, editar campos personalizados ni alterar la configuración de correo o infraestructura.
- **Segregación por Categoría:** Un agente perteneciente a una categoría específica no debe ver, buscar, responder ni exportar tickets pertenecientes a categorías fuera de su alcance.
- **Regla de Delegación de Menor Privilegio:** Quien administra usuarios sólo puede otorgar permisos e intereses iguales o más restrictivos que los que posee en su propia cuenta (no puede delegar privilegios superiores).
- **Privacidad entre Cliente y Staff:** Las notas internas y sus archivos adjuntos permanecen ocultos al cliente en todo momento.

**¿Qué permisos todavía no pudieron confirmarse? (Puntos por validar)**

- **Soporte de Integridad en BD:** No fue posible confirmar en el repositorio si las relaciones de asignación y categorías están respaldadas físicamente por **`FOREIGN KEY`** o si la integridad relacional depende de la aplicación PHP.
- **Impacto por Revocación de Permiso en Sesión Activa:** Por validar el comportamiento del sistema si a un agente se le revoca una categoría o un privilegio mientras mantiene una sesión abierta o un ticket cargado.
- **Validación en Endpoints AJAX y Descargas:** Se requiere verificar que cada endpoint auxiliar (**`upload_attachment.php`**, **`download_attachment.php`**, endpoints AJAX) valide de forma estricta los permisos de categoría y funciones antes de entregar o procesar datos.

---

**3. Evidencia Recomendada para Guardar**

**Matriz Actor por Capacidad**

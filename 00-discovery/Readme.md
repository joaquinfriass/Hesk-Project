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
**Matriz Actor por Capacidad**

| **Capacidad / Función** | **Visitante / Cliente** | **Cliente Registrado** | **Agente (Staff)** | **Administrador** |
| --- | --- | --- | --- | --- |
| **Ver KB Pública** | Sí | Sí | Sí | Sí |
| **Ver KB Privada / Borradores** | No | No | Según privilegio | Sí |
| **Crear Ticket Web** | Sí | Sí | Sí (a nombre de cliente) | Sí |
| **Consultar Ticket** | Sí | Sí | Sí | Sí |
| **Cerrar Ticket** | No | No | Sí | Sí |
| **Consultar Tickets Ajenos** | No | No | Solo en sus categorías | Sí |
| **Agregar Notas Internas** | No | No | Sí | Sí |
| **Reasignar Propietario / Mover** | No | No | Sí | Sí |
| **Gestionar Usuarios / Grupos** | No | No | No | Sí |
| **Configurar Sistema / Correo** | No | No | No | Sí |

## 3. Mapear módulos y objetos

El sistema HESK se estructura sobre un modelo relacional de 40 tablas en MySQL/MariaDB12. A continuación se detallan los módulos funcionales y los objetos de datos que los respaldan:

| Módulo Funcional | Propósito y Descripción | Objetos / Tablas de Datos Asociadas |
| --- | --- | --- |
| **Autenticación y Seguridad** | Gestión de sesiones (separadas para *staff* y clientes), MFA, control de intentos fallidos, bloqueos de IP/email y tokens de recuperación. | `users`, `customers`, `auth_tokens`, `logins`, `mfa_verification_tokens`, `mfa_backup_codes`, `reset_password`, `banned_ips`, `banned_emails` |
| **Tickets** | Agregado principal para el registro, clasificación, ciclo de vida, asignación y relaciones entre solicitudes. | `tickets`, `ticket_to_customer`, `ticket_to_collaborator`, `linked_tickets`, `bookmarks`, `attachments`, `temp_attachments` |
| **Respuestas y Conversación** | Hilo conversacional auditable entre el cliente y el *staff*, incluyendo borradores y notas privadas. | `replies`, `notes`, `reply_drafts` |
| **Categorías y Clasificación** | Segmentación operativa del help desk, delimitación del alcance de permisos y configuración de prioridades o vencimientos por defecto. | `categories`, `custom_statuses`, `custom_priorities` |
| **Usuarios y Permisos** | Administración del personal (*staff* y administradores), asignación directa de privilegios (`heskprivileges`) y grupos de permisos. | `users`, `permission_groups`, `permission_group_members`, `permission_group_categories`, `permission_group_features` |
| **Base de Conocimiento (KB)** | Sistema de autoservicio con taxonomía jerárquica de artículos públicos, privados o borradores, valoraciones y sugerencias. | `kb_categories`, `kb_articles`, `kb_attachments` |
| **Búsqueda y Productividad** | Herramientas operativas como respuestas predefinidas, plantillas de alta, marcadores y definición de campos personalizados. | `std_replies`, `ticket_templates`, `custom_fields` |

---

Entidades Principales y Campos Obligatorios

- **tickets (Cabecera del Ticket):**
    - **Campos clave:** `id`, `trackid`, `u_name`, `u_email`, `category`, `priority`, `subject`, `message`/`message_html`, `dt`, `lastchange`, `closedat`, `status`, `owner`, `due_date`, `customN`.
    - **Campos obligatorios:** **Categoría (category)**, **Asunto (subject)**, **Mensaje (message)** y **Correo electrónico (u_email)**. Además, son obligatorios los campos personalizados (`customN`) configurados con el flag `req`.
- **replies (Respuestas Conversacionales):**
    - **Campos clave:** `id`, `replyto` (referencia a `tickets.id`), `staffid`/`customer_id`, `message`/`message_html`, `dt`, `attachments`, `rating`, `read`.
    - **Campos obligatorios:** **replyto**, **message** y el identificador del actor emisor.
- **users (Personal / Staff / Admin):**
    - **Campos clave:** `id`, `user`, `pass`, `isadmin`, `active`, `name`, `email`, `categories`, `autoassign`, `heskprivileges`.
    - **Campos obligatorios:** **user**, **pass**, **name** y **email**.
- **customers (Clientes Registrados):**
    - **Campos clave:** `id`, `email`, `pass`, `name`, `verified`, `language`.
    - **Campos obligatorios:** **email** (único) y **pass.**
- **categories (Categorías de Tickets):**
    - **Campos clave:** `id`, `name`, `orden`, `autoassign`, `prioridad` y `vencimiento` por defecto.
    - **Campos obligatorios:** **name**.
- **custom_fields (Definición de Campos Personalizados):**
    - **Campos clave:** `id`, `use`, `place`, `type`, `req`, `category`, `name`, `value`, `order`. Gobierna la interpretación funcional de las columnas `customN` en la tabla `tickets`.
- **kb_articles / kb_categories (Base de Conocimiento):**
    - **Campos clave:** `id`, `category_id`, `title`, `content`, `type`/visibilidad (público, privado, borrador), `views`, `rating`.

---

Estados y Relaciones Lógicas

**Glosario de Estados y Prioridades Base**

- **Estados de Ticket (status):**
    - **0 - Nuevo:** Ticket creado pendiente de primera atención por el *staff*.
    - **1 - Esperando respuesta:** El cliente ha respondido; requiere acción del agente.
    - **2 - Respondido:** El *staff* respondió; en espera de acción del cliente o cierre.
    - **3 - Resuelto:** Estado terminal funcional (puede reabrirse según reglas).
    - **4 - En progreso:** Trabajo activo de atención.
    - **5 - En espera:** Pausa operativa. *(Se pueden agregar estados adicionales personalizados en la tabla custom_statuses).*
- **Prioridades (priority):**
    - **3 - Baja**
    - **2 - Media**
    - **1 - Alta**
    - **0 - Crítica** (Generalmente no seleccionable directamente por clientes).

**Relaciones Lógicas Clave entre Entidades**

- **tickets.id**  Clave primaria referenciada por `replies.replyto`, `notes.ticket`, `reply_drafts.ticket`, `bookmarks.ticket_id`, `linked_tickets.ticket_id1/ticket_id2`, `ticket_to_collaborator.ticket_id` y `ticket_to_customer.ticket_id`.
- **users.id**  Referenciado por `tickets.owner`/`openedby`/`closedby`, `replies.staffid`, `bookmarks.user_id`, `ticket_to_collaborator.user_id` y `permission_group_members.user_id`.
- **customers.id**  Referenciado por `ticket_to_customer.customer_id` y `replies.customer_id`.
- **categories.id**  Referenciado por `tickets.category` y `permission_group_categories.category_id`.
- **permission_groups.id**  Relaciona miembros (`permission_group_members`), categorías (`permission_group_categories`) y funciones (`permission_group_features`).

---

Dependencias entre Módulos

```
  [ Base de Conocimiento ] (Sugerencias previas al alta)
            │
            ▼
   [ Categorías ] ◄────────── [ Usuarios / Permisos ] (Grupos y Features)
        │                              │
        ▼                              ▼
   [  Tickets  ] ◄─────────────────────┘ (Asignación de Owner / Colaboradores)
     │   │   │
     │   │   ├──────────────► [ Custom Fields ] (Mapeo de customN)
     │   │   │
     │   │   └──────────────► [ Clientes / Personas ] (Solicitantes / Seguidores)
     │   │
     │   └──────────────────► [ Adjuntos / Archivos ]
     │
     └──────────────────────► [ Respuestas / Notas ] (Conversación e Historial)
```

1. **Tickets  Categorías:** Todo ticket requiere obligatoriamente una categoría. La categoría determina las reglas de autoasignación, la prioridad/vencimiento predeterminados y el alcance de permisos del agente.
2. **Respuestas/Notas  Tickets:** Las respuestas (`replies`) y notas internas (`notes`) no pueden existir de forma aislada; dependen estrictamente de un registro cabecera `tickets.id`.
3. **Tickets  Custom Fields:** La estructura de almacenamiento físico utiliza columnas fijas `custom1` a `customN` en `tickets`. La interfaz y validación dependen dinámicamente de las definiciones en `custom_fields`.
4. **Usuarios  Permisos y Categorías:** La visibilidad del *staff* sobre los módulos de tickets y reportes depende de las categorías asignadas y los privilegios otorgados directamente o mediante `permission_groups`.
5. **Tickets  Base de Conocimiento:** Durante la creación pública de un ticket, el módulo de tickets consulta `kb_articles` para mostrar sugerencias automáticas antes de completar el envío.

---

### 4. Modelar flujos críticos

1. Camino del Cliente (*Customer Journey*)

**Diagrama de Flujo del Cliente**

```
[Inicio: Problema/Duda]
       │
       ▼
[¿Consulta KB?] ──────(Sí: Artículo resuelve)──────► [Fin Exitoso: Autoservicio]
       │ (No / No resuelve)
       ▼
[Alta de Ticket] ───(Camino Alternativo: Error validación/adjunto)───► [Corrección Formulario]
       │
       ▼
[Ticket Creado (Estado: 0 Nuevo)] ──► [Notificación Email con Tracking ID]
       │
       ▼ (Transferencia de responsabilidad al Staff)
[Espera de Respuesta del Staff]
       │
       ▼
[Notificación de Respuesta (Estado: 2 Respondido)]
       │
       ├────(¿Problema Resuelto?)────► [Cierre de Ticket (Estado: 3 Resuelto)] ──► [Fin Exitoso]
       │
       └────(¿Sigue con dudas?)─────► [Cliente Responde (Estado: 1 Esperando respuesta)]
                                                  │
                                                  └──► (Vuelve a Espera de Staff)
```

- **Comienzo exitoso:** El cliente ingresa al portal público o canal de correo para buscar una solución. Si halla la respuesta en un artículo de la Base de Conocimiento (KB), el flujo finaliza en **autoservicio** sin generar carga operativa. Si requiere atención, completa el formulario web con categoría, asunto, mensaje, datos de contacto y adjuntos
- **Final exitoso:** La solicitud es atendida por el *staff*, el problema queda solucionado y el ticket pasa al estado **3 - Resuelto** (ya sea confirmado por el cliente o marcado por el agente).
- **Transferencia de responsabilidad:**
    1. **Del Cliente al Agente:** Al enviar el formulario de alta, la responsabilidad operativa pasa al equipo de soporte (el estado queda en `0 - Nuevo` o se notifica al *owner* autoasignado/elegible)
    2. **Del Agente al Cliente:** Cuando el *staff* emite una respuesta, la responsabilidad de verificar o aportar más datos vuelve al cliente (el estado cambia a `2 - Respondido`).
- **Interrupciones y caminos alternativos:**
    - **Fallo de validación:** El usuario omite campos requeridos, el adjunto excede el tamaño/tipo permitido o falla la prueba anti-SPAM/CAPTCHA, requiriendo reintento.
    - **Uso de cuentas:** Si las cuentas de cliente están habilitadas, el seguimiento se realiza desde su panel centralizado; de lo contrario, accede mediante la combinación de `trackid` + correo electrónico.
    - **Reapertura de ticket:** Si un ticket resuelto recibe un nuevo mensaje del cliente dentro de las reglas permitidas, se reabre cambiando su estado a `1 - Esperando respuesta`.

---

2. Camino del Agente (*Staff Journey*)

**Diagrama de Flujo del Agente**

```
[Inicio: Inicio de Turno en /admin]
       │
       ▼
[Filtro y Revisión de Bandeja]
       │
       ▼
[Apertura de Ticket (Estado: 0 Nuevo / 1 Esperando respuesta)]
       │
       ├────(¿Categoría/Asignación incorrecta?)────► [Reclasificar / Mover Categoria / Asignar Owner]
       │
       ├────(¿Requiere consulta interna?)──────────► [Agregar Nota Interna / Colaborador]
       │
       ▼
[Redacción de Respuesta (WYSIWYG / Plantilla)]
       │
       ▼
[Envío de Respuesta] ──► [Estado pasa a 2 Respondido / 4 En progreso / 5 En espera]
       │
       ▼ (Transferencia de responsabilidad al Cliente)
[¿Resolución del Caso?] ──► [Cambio a Estado: 3 Resuelto] ──► [Registrar Tiempo Trabajado] ──► [Fin Exitoso]
```

- **Comienzo exitoso:** El agente inicia sesión en `/admin` (superando autenticación y MFA si aplica), consulta las bandejas filtradas por sus categorías asignadas y selecciona un ticket pendiente (`0 - Nuevo` o `1 - Esperando respuesta`).
- **Final exitoso:** El agente provee la solución en la conversación web, cambia el estado a `3 - Resuelto`, registra opcionalmente el tiempo trabajado y concluye la atención.
- **Transferencia de responsabilidad:**
    - **Entre Agentes / Categorías:** El agente puede mover el ticket a otra categoría o reasignar el propietario (*owner*), transfiriendo el caso a otro agente o grupo.
    - **Hacia Colaboradores:** Puede incluir a un segundo agente como colaborador para recibir asistencia técnica puntual sin ceder la propiedad del ticket.
- **Interrupciones y caminos alternativos:**
    - **Colisión de edición concurrente:** Si otro agente modifica o responde el mismo ticket simultáneamente, el sistema debe alertar para evitar la pérdida silenciosa de datos o duplicación de respuestas.
    - **Requerimiento de información externa:** El ticket se coloca en estado `4 - En progreso` o `5 - En espera` mientras se realizan gestiones de fondo.
    - **Notas internas privadas:** El agente registra análisis técnicos mediante notas privadas que el cliente jamás puede visualizar.

---

3. Tabla de Precondiciones y Postcondiciones

| Flujo / Caso de Uso | Actor | Precondiciones | Postcondiciones (Estado Final) |
| --- | --- | --- | --- |
| **Consultar KB (UC-01)** | Visitante / Cliente | KB activa y artículos públicos publicados. | El usuario visualiza la solución; se incrementa el contador de visitas/valoración si aplica. |
| **Crear Ticket Web (UC-02)** | Visitante / Cliente | Portal accesible, categoría pública activa y datos obligatorios completos | Ticket registrado en BD con `trackid` único; estado `0 - Nuevo`; notificaciones por correo enviadas. |
| **Consultar/Responder Ticket (UC-04/05)** | Cliente | Identidad válida (cuenta o `trackid` + email) y ticket en estado modificable. | Nueva entrada en `replies`; estado del ticket actualizado a `1 - Esperando respuesta`; *owner* notificado. |
| **Atender y Responder Ticket (UC-07)** | Agente (*Staff*) | Agente autenticado con privilegios en la categoría del ticket. | Respuesta registrada; estado cambia a `2 - Respondido` (o `4`/`5`); cliente notificado. |
| **Agregar Nota Interna (UC-08)** | Agente (*Staff*) | Permisos de acceso al ticket específico8. | Nota guardada en `notes`; nunca visible para el cliente; historial actualizado. |
| **Mover / Reclasificar Ticket (UC-09)** | Agente (*Staff*) | Permisos en la categoría de origen y destino. | Categoría actualizada; reglas de autoasignación/vencimiento evaluadas según la nueva categoría. |
| **Resolver / Cerrar Ticket (UC-10)** | Agente / Cliente | Ticket en estado activo y permisos de cierre habilitados. | Estado actualizado a `3 - Resuelto`; marca `closedat` y `closedby` registradas. |

---

4. Lista de Transiciones de Estado del Ticket

Las transiciones principales sobre la columna `status` en la tabla `tickets` son:

- **[Inicio]  0 - Nuevo:** Al crearse un ticket por formulario web o correo, pendiente de primera atención por el *staff*.
- **0 - Nuevo  4 - En progreso / 5 - En espera:** Cuando el agente toma el ticket y comienza la investigación sin responder aún al cliente.
- **0 - Nuevo / 1 - Esperando respuesta  2 - Respondido:** Cuando el agente emite una respuesta oficial al cliente desde la consola administrativa.
- **2 - Respondido  1 - Esperando respuesta:** Cuando el cliente agrega un nuevo mensaje en la conversación.
- **0 / 1 / 2 / 4 / 5  3 - Resuelto:** Cuando el agente (o el cliente, si está autorizado) marca la solicitud como resuelta.
- **3 - Resuelto  1 - Esperando respuesta:** Cuando el cliente responde a un ticket resuelto, provocando su reapertura.

*(Nota: La instalación admite estados personalizados que extienden estos ID base mediante la tabla custom_statuses).*

---

5. Datos Persistentes entre Pasos del Ciclo de Vida

Para garantizar la trazabilidad completa de punta a punta, los siguientes atributos deben persistir de forma inmutable o auditable a lo largo de todas las transiciones:

1. **Identificadores de Dominio:** `id` secuencial interno y `trackid` único de cara al cliente.
2. **Datos de Identidad:** Correo del solicitante (`u_email`), nombre (`u_name`) y vínculos en `ticket_to_customer`.
3. **Parámetros de Clasificación:** Categoría (`category`), prioridad (`priority`) y campos personalizados (`customN`).
4. **Asignación y Responsabilidad:** Agente propietario (`owner`), colaboradores (`ticket_to_collaborator`) y usuario de apertura/cierre (`openedby`/`closedby`).
5. **Conversación y Auditoría:** Mensajes iniciales, historial completo de respuestas (`replies`), notas internas (`notes`), adjuntos vinculados (`attachments`), tiempo trabajado (`time_worked`) y marcas temporales de creación, modificación y cierre (`dt`, `lastchange`, `closedat`, `due_date`).

## 5. Separar hechos de inferencias

**Clasificación de Reglas de Negocio**

Para garantizar la precisión de la estrategia de pruebas, clasificamos las reglas del sistema en tres niveles de certeza según la evidencia técnica disponible:

```
                               ┌─ CONFIRMADAS ───► Documentadas en guías / código
                                  │
REGLAS DE NEGOCIO EN HESK ─────┼─ INFERIDAS ─────► Deducidas de la UI / arquitectura
                                  │
                               └─ DESCONOCIDAS ──► Por validar / respuesta externa
```

**1. Reglas Confirmadas (Documentadas u Observadas en Código)**

- **BR-01 (Identificación única):** Todo ticket posee un identificador de seguimiento público único (**`trackid`**), pudiendo coexistir con un ID entero secuencial interno (**`id`**).
- **BR-02 (Visibilidad de categorías):** Las categorías públicas son seleccionables por clientes; las privadas están restringidas al personal autorizado (*staff*).
- **BR-03 (Parámetros por defecto):** La categoría define la prioridad, el plazo de vencimiento (**`due_date`**) y las reglas de autoasignación iniciales.
- **BR-04 (Aislamiento de clientes):** El cliente solo puede ver o consultar sus propios tickets o aquellos donde figura explícitamente como participante.
- **BR-05 (Campos personalizados):** Los campos personalizados (**`customN`**) pueden configurarse como públicos o privados, requeridos u opcionales, y restringirse a categorías específicas.
- **BR-06 (Formato conversacional):** Las entradas enviadas por clientes se almacenan y muestran estrictamente en texto plano, mientras que el *staff* puede utilizar editor HTML enriquecido (TinyMCE) si está activo.
- **BR-07 (Privacidad de notas):** Las notas internas creadas por el *staff* nunca se exponen al cliente.
- **BR-08 (Supresión de auto-notificación):** El sistema no envía notificaciones por correo al mismo agente o usuario que ejecuta la acción.
- **BR-09 (Delegación de permisos):** Un usuario que administra cuentas de personal solo puede otorgar un alcance de permisos e intereses igual o más restrictivo que el que posee su propia cuenta.
- **BR-10 (Respuesta vía Web):** El *staff* no puede responder solicitudes respondiendo directamente a correos salientes; la plataforma exige enviar la respuesta desde la consola **`/admin`** por seguridad.
- **BR-11 (Higiene de instalación):** El directorio **`/install`** debe eliminar de forma obligatoria al concluir la instalación o actualización.

**2. Reglas Inferidas (Deducidas de la Interfaz y Arquitectura PHP)**

- **Integridad referencial en capa de aplicación:** La eliminación o fusión de tickets y clientes se ejecuta mediante consultas SQL secuenciales manuales en código PHP, lo que indica que la integridad relacional no depende de restricciones **`FOREIGN KEY`** en MySQL sino de la lógica del programa.
- **Precarga de formularios web:** Se pueden precargar parámetros en el formulario de alta pública (**`submit_ticket.php`**) mediante GET/POST (**`name`**, **`email`**, **`catid`**, **`priority`**, **`subject`**, **`message`**), pero la aplicación aplica la validación en el servidor antes de guardar los datos.
- **Generación de tokens CSRF:** El token de sesión anti-CSRF se genera derivando hashes SHA-1 sobre valores temporales no CSPRNG y se valida mediante comparación estándar en los scripts procesadores.
- **Aislamiento de sesiones:** Las sesiones de clientes (**`HESKC`**) y de personal (**`HESK`**) se gestionan mediante cookies y prefijos totalmente independientes.

**3. Reglas Desconocidas / Por Validar (Incertidumbres Técnicas o Ambivalentes)**

- **Estado del esquema físico en MySQL:** Debido a que el servicio MySQL no estuvo disponible durante la inspección estática (error 2002), se desconoce si la base de datos real contiene índices físicos, tipos estrictos o claves foráneas creadas.
- **Efecto de la revocación activa de permisos:** No está determinado el comportamiento exacto cuando a un agente se le revoca una categoría o privilegio mientras mantiene una sesión abierta o un formulario cargado.
- **Colisión de concurrencia entre agentes:** Falta validar si la edición o cierre simultáneo de un ticket por dos agentes implementa bloqueos optimistas o si se produce un sobreescribimiento directo.
- **Inexistencia de API REST oficial:** Se confirma la ausencia de una API REST pública nativa en HESK. Cualquier integración automatizada externa depende de una decisión de arquitectura propia o extensión de terceros.
- **Módulos demostrativos Cloud:** Los accesos a Escalamiento, Tickets recurrentes, Satisfacción y Estadísticas avanzadas figuran en la UI, pero corresponden a demostraciones comerciales de HESK Cloud y no ejecutan lógica en la versión autohospedada.

---

**Análisis de Comportamientos y Decisiones Dependientes**

- **¿Qué regla está documentada vs. cuál se infiere?** Estándares como el formato de respuesta del cliente (texto plano) y el aislamiento de notas internas están documentados oficialmente. Por el contrario, el procesamiento de eliminaciones multi-tabla sin transacciones **`BEGIN/COMMIT`** se infiere de la lectura estática del código.
- **¿Qué comportamiento es ambiguo?** La respuesta del sistema ante fallos de conexión parciales (ej. caída de MySQL tras insertar el ticket pero antes de guardar los adjuntos o enviar el correo).
- **¿Qué decisión depende de una respuesta externa?**
    1. Definición de la **zona horaria de negocio** y su conversión respecto a UTC para determinar el vencimiento (**`due_date`**).
    2. La política de activación de **cuentas de cliente** (desactivadas en la configuración local observada).
    3. Estrategia de **integración de correo** (SMTP saliente e IMAP/POP3 entrante estaban inactivos en el entorno evaluado).

---

**Banco de Preguntas a Interesados**

```
                               ┌─── PRODUCT OWNER ────► Retención PII, Cuentas, SLA
                               │
BANCO DE PREGUNTAS (QA) ───────┼─── DESARROLLO ───────► Transacciones, DB, APIs
                               │
                               ├─── SOPORTE ──────────► Cron jobs, Backups, Adjuntos
                               │
                               └─── SEGURIDAD ────────► IDOR, HTTPS, Captcha, CSRF
```

**A Product Owner / Negocio**

1. ¿Las cuentas de clientes permanecerán desactivadas o se exigirá registro previo para abrir tickets?
2. ¿Cuál es el SLA y la zona horaria oficial que gobernará los reportes y las alertas de vencimiento (**`due_date`**)?
3. ¿Cuál es la política de retención y el procedimiento para la anonimización de datos personales (GDPR/Derecho al olvido)?
4. ¿Qué tratamiento se dará a las secciones informativas de HESK Cloud que aparecen en el menú pero no son funcionales en esta versión?

**A Desarrollo / Arquitectura**

1. Dado que no existen transacciones relacionales explícitas en el código procedural, ¿cómo se previenen registros huérfanos si ocurre un error en medio de un proceso compuesto?
2. Ante la ausencia de una API REST nativa, ¿se construirá una capa de servicios propia o las pruebas de integración se realizarán sobre las llamadas HTTP/POST de la UI?
3. ¿Cuál será la versión exacta de PHP y MySQL/MariaDB definida como línea base para el despliegue final?

**A Soporte / Operaciones**

1. ¿Con qué frecuencia se ejecutará la tarea programada (**`cron/email_overdue_tickets.php`**) y en qué entorno?
2. ¿Cuál es el procedimiento y la ventana para realizar respaldos coordinados de la base de datos y la carpeta física **`/attachments`**?
3. ¿Qué límites de tamaño y qué extensiones de archivo se autorizarán para los adjuntos en producción?

**A Seguridad**

1. ¿Se forzará el uso de HTTPS/SSL y banderas **`Secure`** en cookies para todos los ambientes fuera de entorno local?
2. ¿Se mitigará la vulnerabilidad de enumeración/IDOR en la descarga de adjuntos y la consulta de tickets por **`trackid`**?
3. ¿Se requiere implementar CSP (Content Security Policy) o actualizar la generación de tokens CSRF a un estándar criptográficamente seguro?

---

## 6. Delimitar el programa de pruebas

**1. Delimitación del Programa de Pruebas: Demo vs. Instancia Local vs. Fuera de Alcance**

Para estructurar la estrategia de pruebas sin comprometer entornos públicos ni intentar validar capacidades inexistentes, la cobertura se divide según el nivel de acceso y control de datos requerido:

**A. Demo Pública (Exploración Prudente y Lectura)**

- **Alcance:** Exploración funcional manual de la interfaz del cliente (portal de autoservicio), consulta y búsqueda en la Base de Conocimiento (KB) pública, navegación por menús y formularios de alta de tickets con datos sintéticos no sensibles.
- **Justificación:** La demo permite validar usabilidad, diseño responsive y flujos de usuario finales sin requerir credenciales administrativas ni modificar configuraciones.

**B. Instancia Local (Entorno Controlado)**

- **Alcance:** Pruebas destructivas, validación de base de datos MySQL/MariaDB, inspección del sistema de archivos (**`/attachments`**, **`/cache`**), automatización de interfaz (E2E) e integración con servicios de correo (*mail catcher*) y tareas programadas (*Cron*).
- **Justificación:** Permite ejecutar consultas SQL directas, reiniciar el estado mediante scripts de *setup/teardown*, simular fallas técnicas y ejecutar suites automatizadas de alta frecuencia de forma aislada y reproducible.

**C. Fuera de Alcance (Exclusiones Formales)**

- **Funcionalidades de HESK Cloud:** Módulos de escalamiento, tickets recurrentes, encuestas de satisfacción y estadísticas avanzadas que aparecen en la UI pero corresponden a demostraciones comerciales no funcionales en la versión autohospedada.
- **API REST pública nativa:** HESK no expone un contrato o API REST nativa para operaciones CRUD de tickets. *(Cualquier prueba de API requerirá un entorno simulado/mock o la integración de una capa propia).*
- **Capacidades ITSM / CMDB:** Gestión de activos, cambios o problemas (no soportadas nativamente por la herramienta).
- **Respuesta de agentes por correo saliente:** Regla de negocio que exige que el personal responda obligatoriamente desde la consola web **`/admin`** por motivos de seguridad.

---

**2. Matriz de Capacidades por Etapa Futura del Portfolio**

| **Etapa del Portfolio** | **Capacidades y Funcionalidades Asignadas** | **Entorno Requerido** | **Criterio de Selección / Valor QA** |
| --- | --- | --- | --- |
| **Etapa 1: Estrategia y Planificación** | Modelado de la matriz de riesgos, diseño de la matriz RBAC (roles x categorías), trazabilidad y definición del catálogo de regresión. | N/A (Documentación) | Establecer los criterios de entrada, salida y severidad antes de la ejecución técnica. |
| **Etapa 2: Pruebas Funcionales y Manuales** | NAVEGACIÓN KB, alta de ticket web, seguimiento por **`trackid`**, ciclo de vida del ticket (*New*, *Replied*, *Resolved*), filtros de bandeja, notas internas y campos personalizados. | Demo Pública / Instancia Local | Exploración basada en riesgo, charters exploratorios y validación de reglas de negocio en UI. |
| **Etapa 3: Pruebas de Base de Datos / SQL** | Verificación de persistencia, detección de registros huérfanos tras eliminaciones/fusiones, consistencia en **`tickets`**, **`replies`**, **`customers`** e integridad en **`permission_groups`**. | **Instancia Local** | Exige acceso a MySQL/MariaDB para ejecutar consultas directas y verificar la integridad relacional. |
| **Etapa 4: Pruebas de API Manuales (Postman)** | Validación de endpoints auxiliares JSON (**`upload_attachment.php`**, sugerencias KB, gestión de sesiones/autenticación) o colección contra API simulada/mock. | Instancia Local / Mock API | Evaluación de contratos HTTP, parámetros, headers, manejo de errores y seguridad sin interfaz. |
| **Etapa 5: API Automatizada (Playwright + TS)** | Automatización de regresión de endpoints auxiliares, autenticación, creación de fixtures y validación de esquemas JSON. | Instancia Local / Sandbox | Ejecución rápida de contratos estables, preparación y limpieza de datos (*teardown*). |
| **Etapa 6: UI Automatizada (Playwright + TS)** | Flujos E2E críticos: Login de staff, alta pública de ticket, toma de propiedad, respuesta del agente y cambio a estado resuelto. | **Instancia Local** | Cobertura de recorridos clave de usuario sin saturar la suite con escenarios frágiles. |
| **Etapa 7: Integración Continua (GitHub Actions)** | Pipeline automatizado (linter PHP, ejecuciones Smoke/E2E en headless, generación de reportes HTML y gestión de secretos). | CI Runner / Local | Validación continua del estado del código y publicación automática de evidencia de prueba. |

---

**3. Análisis de Preguntas Guía**

- **¿Qué puede probarse de manera segura en la demo?**
Búsquedas en la KB pública, navegación responsive, creación de tickets simples sin adjuntos maliciosos y consulta del estado de solicitudes utilizando credenciales sintéticas de prueba.
- **¿Qué exige control de datos o acceso técnico?**
Las pruebas sobre la base de datos (queries SQL), el análisis de archivos binarios en el servidor (**`/attachments`**), la verificación de permisos en perfiles administrativos, las tareas programadas (*Cron*) y la interceptación de correos entrantes y salientes.
- **¿Qué pruebas serían riesgosas o irreproducibles en la demo?**
Pruebas de seguridad intrusivas (SQLi, XSS persistente, subida de malware), pruebas de carga/estrés, borrado masivo de datos o modificaciones de la configuración global que puedan alterar el entorno a otros usuarios.
- **¿Qué dependencia podría bloquear una etapa?**
La indisponibilidad del servicio MySQL local bloquearía la Etapa 3 de datos; la falta de un servidor SMTP/mail catcher simularía falsos fallos en la Etapa 1 y 6; y la ausencia de una API REST pública nativa obligará a redefinir el alcance de las Etapas 4 y 5 mediante una API simulada o endpoints internos.

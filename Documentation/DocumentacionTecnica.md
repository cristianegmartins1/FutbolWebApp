# FutbolWebApp — Documentación técnica

**Entorno:** DataFlex Studio 25.0  
**Base de datos:** MySQL  
**Tipo de aplicación:** DataFlex Web Application  
**Repositorio:** [FutbolWebApp – GitHub](https://github.com/cristianegmartins1/FutbolWebApp)

---

## 1. Sincronización y conexión con la base de datos

### 1.1. Conexión con MySQL

El proyecto utiliza una base de datos MySQL denominada `football`.
La conexión desde DataFlex Studio se realiza mediante ODBC:

- **Driver:** `ODBC_DRV`
- **DSN:** `football`
- **Base de datos:** `football`

La conexión permite que DataFlex acceda a las tablas y a sus estructuras para utilizarlas en la aplicación.

Las credenciales de acceso y demás parámetros sensibles deben mantenerse fuera del repositorio.

### 1.2. Sincronización de las estructuras

Se utiliza DataFlex Database Builder para trabajar con las estructuras de las tablas de la base de datos.
Durante el desarrollo se han utilizado, entre otras, las siguientes tablas:

| Tabla                | Función                          | 
|----------------------|----------------------------------| 
| `liga`               | Almacena las ligas               | 
| `equipo`             | Almacena los equipos             | 
| `entrenador`         | Almacena los entrenadores        | 
| `jugador`            | Almacena los jugadores           | 
| `posicion`           | Almacena las posiciones          | 
| `jugadorposicion`    | Relaciona jugadores y posiciones | 

Las definiciones de archivos (`.fd`) permiten que DataFlex identifique las tablas y sus campos.

**Buena práctica:** mantener sincronizadas las estructuras de la base de datos y sus definiciones en DataFlex. 
Cualquier modificación de campos o índices debe comprobarse en ambos entornos.

---

## 2. Organización del proyecto

El proyecto mantiene la estructura habitual de una aplicación DataFlex:

| Directorio  | Responsabilidad                                | 
|-------------|------------------------------------------------| 
| `AppHtml`   | Recursos HTML de la aplicación                 | 
| `AppSrc`    | Vistas y código fuente de la aplicación        | 
| `DDSrc`     | Clases Data Dictionary                         | 
| `Data`      | Archivos y recursos relacionados con los datos | 
| `IdeSrc`    | Configuración del entorno de desarrollo        | 
| `Programs`  | Archivos generados y programas compilados      | 

### 2.1. Archivos principales

- `.wo`: implementación de vistas y objetos web.
- `.dd`: clases Data Dictionary.
- `.fd`: definiciones de archivos y campos de las tablas.
- `Navigation.json`: configuración de navegación de la aplicación.
- `WebApp.src`: punto de entrada del programa WebApp.

### 2.2. Archivos excluidos de Git

El archivo `.gitignore` evita incorporar archivos temporales, cachés y configuraciones locales.

Entre ellos:
- DataFlex - archivos de caché: `*.x64.cch`
- Archivos temporales de Windows: `Thumbs.db`, `ehthumbs.db`, `Desktop.ini`, `.DS_Store`
- Archivos temporales y copias de seguridad: `*.tmp`, `*.temp`, `*.bak`, `*.bak`, `*.old`, `*.swp`, `*~`
- Archivos temporales de editores: `.vscode/`, `.idea/`
- ARchivos de registro: `*.log`
- Archivos generados por DataFlex: `*.dep` , `*.prn`, `*.dbg`, `*.exe`
- Configuración personal DataFlex Studio:  `IdeSrc/cristiane_Studio.dsk`, `AppSrc/WebApp.cfg`

Se conservan en Git los archivos fuente necesarios para mantener y compilar el proyecto.

---
## 3. Web Properties

### 3.1. Concepto

Las Web Properties permiten sincronizar información entre el servidor DataFlex y el navegador.
En una aplicación web, las propiedades pueden tener distintos ámbitos:

| Tipo            | Característica                                           | 
|-----------------|----------------------------------------------------------| 
| Client          | Se sincroniza entre cliente y servidor                   | 
| ClientProtected | Se sincroniza y mantiene también un valor en el servidor | 
| Server          | Permanece en el servidor durante la vida de la página    | 
| ServerSession   | Permanece en el servidor durante la sesión               | 

No todas las propiedades de un objeto web son Web Properties.

### 3.2. WebGet y WebSet

DataFlex proporciona dos comandos para trabajar con propiedades web:

```dataflex
WebGet pbRender of oDetailButton to bRender
WebSet pbRender of oDetailButton to False
```

- `WebGet`: obtiene el valor sincronizado de una Web Property.
- `WebSet`: modifica el valor sincronizado y comunica el cambio al cliente.

No se debe utilizar `Set` indiscriminadamente para modificar propiedades normales durante las peticiones de los clientes.
Las propiedades normales deben tratarse como configuración estática. Para valores que cambian durante la ejecución y pertenecen al estado de un cliente, se deben utilizar Web Properties.

### 3.3. Aplicación en FutbolWebApp

En nuestras vistas utilizamos, por ejemplo:

```dataflex
WebSet pbRender of oDetailButton to False
```

Este código permite controlar la visibilidad de los botones de detalle según el contexto de navegación.
También utilizamos propiedades de configuración de los controles web, como:

```dataflex
Set piColumnCount to 12
WebSetResponsive piColumnCount rmMobile to 4
```

De esta manera, el número de columnas se adapta a pantallas de menor tamaño.

**Buena práctica:** utilizar Web Properties cuando el estado debe sincronizarse con el navegador y mantener las propiedades de configuración como valores estáticos.

---

## 4. Navegación entre vistas

### 4.1. Tipos de navegación
En las aplicaciones DataFlex Web, el comportamiento de una vista depende de su tipo y del contexto desde el que se abre.
Se han trabajado los cuatro contextos principales:

| Tipo                     | Identificador | Uso                                                  |
|--------------------------|---------------|------------------------------------------------------|
| Desde la vista principal | `nfFromMain`  | Navegación entre Select y Zoom de una misma entidad  |
| Desde una vista padre    | `nfFromParent`| Abrir una vista hija relacionada y restringida       |
| Desde una vista hija     | `nfFromChild` | Abrir una selección de una entidad relacionada padre |
| Sin relación definida    | `nfUndefined` | Navegar entre vistas sin una relación implícita      |

### 4.2. Propiedades de las vistas
En las vistas de selección y detalle se utilizan:

```dataflex
Set peWebViewStyle to wvsDrilldown
Set peViewType to vtSelect
```

Para una vista de detalle:

```dataflex
Set peViewType to vtZoom
```

- `wvsDrilldown` establece el estilo de navegación por niveles.
- `vtSelect` identifica una vista de selección.
- `vtZoom` identifica una vista de detalle.

### 4.3. Registro de rutas
Para registrar una navegación hacia otra vista utilizamos:

```dataflex
WebRegisterPath ntNavigateForward oZoomJugador
```

Y para ejecutar la navegación:

```dataflex
Procedure OnClick
    Send NavigatePath
End_Procedure
```

El framework determina el contexto de navegación teniendo en cuenta la vista y el objeto que la inicia.

También se utilizan:

- `NavigateForward`: abre una vista siguiente.
- `NavigateClose`: cierra la vista actual y regresa a la anterior.
- `GetNavigateData`: obtiene la información del contexto de navegación.
- `OnNavigateForward`: permite adaptar el comportamiento de la vista de destino.

### 4.4. Aplicación en FutbolWebApp

**A. SelectJugador → ZoomJugador**
Al seleccionar un jugador se abre su vista de detalle.
Se utiliza `nfFromMain` o `nfUndefined`, según el contexto desde el que se haya iniciado la navegación.

**B. ZoomEquipo → SelectLiga**
Se incorpora un botón para abrir la selección de ligas desde el detalle del equipo.
La selección permite trabajar con la entidad padre relacionada.

**C. ZoomLiga → Equipos de la Liga**
La vista `ZoomLiga` muestra los equipos relacionados con la liga seleccionada.
Se utiliza una estructura DDO relacionada y una restricción mediante `Constrain_File`.

**D. ZoomJugador → SelectPosicion**
Se incorpora el botón:

`Seleccionar Posición`
Este botón abre `SelectPosicion` mediante una ruta de navegación.

Al seleccionar una posición, se abre `ZoomPosicion` cuando el contexto es `nfUndefined`, siguiendo la lógica implementada en `OnRowClick`.

### 4.5. InvokingObject
En la navegación, DataFlex identifica tanto la vista que inicia la acción como el objeto que la solicita.
El parámetro `hoInvokingObject` permite identificar el objeto que inició la navegación.
Esta información resulta importante cuando una misma vista puede abrirse desde distintos controles o contextos.

**Buena práctica:** registrar explícitamente las rutas, identificar correctamente el contexto de navegación y evitar ciclos de navegación entre vistas.

---

## 5. Control de versiones con Git
El proyecto utiliza Git y GitHub para controlar los cambios del código fuente.

### 5.1. Ramas
Se ha establecido una estructura de trabajo basada en:

- `main`: versión integrada del proyecto.
- `develop`: rama de integración del desarrollo.
- `feature/*`: ramas específicas para implementar funcionalidades.

Entre las ramas utilizadas están:
- `feature/dashboard-tarjetas`
- `feature/vista-entrenador`
- `feature/vista-jugador`
- `feature/vista-posicion`
- `feature/vista-equipo`
- `feature/vista-liga`

### 5.2. Flujo de trabajo
El procedimiento seguido es:

1. Crear o seleccionar una rama de funcionalidad.
2. Implementar los cambios.
3. Compilar y probar la funcionalidad.
4. Crear un commit con un mensaje descriptivo.
5. Hacer Push a GitHub.
6. Integrar las funcionalidades en `develop`.
7. Revisar los cambios mediante un Pull Request de `develop` hacia `main`.

### 5.3. Buenas prácticas
- Utilizar commits pequeños y descriptivos.
- No incorporar archivos generados o configuraciones locales.
- Comprobar la compilación antes de integrar cambios.
- Revisar los archivos incluidos en cada commit.
- Utilizar Pull Requests para revisar la integración entre ramas.

---

## 6. Estado del desarrollo
Hasta el Sprint 2 se han implementado y probado:

- La conexión de la aplicación con MySQL.
- La estructura inicial de los Data Dictionaries.
- Las vistas de selección y detalle de las entidades principales.
- La navegación entre vistas principales, padres e hijas.
- La visualización de equipos relacionados con una liga.
- La navegación desde un jugador hacia la selección de posiciones.
- La estructura de relación N:M entre jugadores y posiciones.
- El control de versiones mediante Git y GitHub.

Los cambios desarrollados se han trabajado en `develop` y se han preparado para su integración en `main`.

--- 

## 7. Data Dictionaries (DD)

### 7.1. Concepto
Los Data Dictionaries centralizan las reglas de negocio y la gestión de datos.

Sus responsabilidades principales son:

- Coordinar las operaciones de acceso a la base de datos.
- Validar los datos antes de guardarlos.
- Gestionar las operaciones de inserción, modificación y eliminación.
- Mantener las relaciones entre tablas.
- Compartir las reglas de datos entre las distintas vistas.

En DataFlex, las clases DD se definen mediante clases derivadas de `DataDictionary`.
En las vistas se instancian objetos Data Dictionary (DDO), que forman una estructura coordinada para acceder a las tablas relacionadas.

### 7.2. Clases implementadas
El proyecto utiliza clases específicas para las entidades principales:

- `cligaDataDictionary.dd`
- `cequipoDataDictionary.dd`
- `cjugadorDataDictionary.dd`
- `cposicionDataDictionary.dd`

Cada clase establece su tabla principal mediante `Set Main_File`.

También se definen relaciones con otras tablas utilizando:

- `Set Add_Server_File`
- `Set Add_Client_File`

Estas declaraciones permiten construir la estructura de relaciones que utilizarán los DDO.

### 7.3. Ejemplo: Liga y Equipo
En `cligaDataDictionary.dd`:

```dataflex
Class cligaDataDictionary is a DataDictionary
    Procedure Construct_Object
        Forward Send Construct_Object
        Set Main_File to liga.File_Number
        Set Add_Client_File to equipo.File_Number
    End_Procedure
End_Class
```

La clase identifica `liga` como tabla principal y `equipo` como tabla relacionada.

En `ZoomLiga.wo` se establece la estructura DDO:

```dataflex
Object oliga_DD is a cligaDataDictionary
End_Object

Object oequipo_DD is a cequipoDataDictionary
    Set DDO_Server to oliga_DD
    Set Constrain_File to liga.File_Number
End_Object

Set Main_DD to oliga_DD
Set Server to oliga_DD
```

El resultado es una lista de equipos restringida a la liga seleccionada.

Esta solución permite que el filtrado se gestione mediante la estructura de datos, evitando implementar manualmente la búsqueda de equipos en cada vista.

### 7.4. Relación Jugador y Posición
La relación entre `jugador` y `posicion` es de muchos a muchos (N:M).

Para implementarla se utiliza la tabla intermedia `jugadorposicion`:

```text
jugador
   |
   | 1:N
   |
jugadorposicion
   |
   | N:1
   |
posicion
```

La tabla intermedia contiene:

- `ID`
- `JugadorID`
- `PosicionID`

Se comprobaron los índices:

- `PRIMARY`: `ID`.
- `uqJugadorPosicion`: `JugadorID`, `PosicionID`.
- `uqPosicionJugador`: `PosicionID`, `JugadorID`.

Los dos últimos índices son únicos y evitan duplicar una misma combinación de jugador y posición.

**Buena práctica:** definir las relaciones y restricciones en la estructura de datos y centralizar las reglas de negocio en los Data Dictionaries, evitando duplicar lógica en las vistas.

---
---

## 8. Referencias oficiales
Documentación de DataFlex:

- [Data Dictionaries](https://docs.dataflex.dev/Welcome/Data_Dictionaries/)
- [Introduction to Data Dictionaries](https://docs.dataflex.dev/DevelopmentGuide/Introduction_to_Data_Dictionaries/)
- [Web Properties](https://docs.dataflex.dev/DevelopmentGuide/Web_Properties/)
- [View Navigation Properties](https://docs.dataflex.dev/DevelopmentGuide/View_Navigation_Properties/)
- [What's New in DataFlex 2025 (25.0)](https://docs.dataflex.dev/Welcome/Whats_New_in_DataFlex_2025/)

Repositorio:
- [FutbolWebApp – GitHub](https://github.com/cristianegmartins1/FutbolWebApp)

---
*Documento técnico de referencia del desarrollo de FutbolWebApp. Debe actualizarse a medida que se incorporen nuevas funcionalidades y reglas de negocio.*


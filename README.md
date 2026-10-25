# Gestión de transporte y paquetería (Java Swing + JDBC + MySQL)

Aplicación de escritorio con CRUD completo para **Camión**, **Camionero**, **Ciudad**, **Paquete** y **Camionero–Camión**
sobre la base `db_paqueteria` (MySQL, InnoDB). Proyecto **Maven**, abre directo en Apache NetBeans.

## 1. Estructura del proyecto

```
paqueteria/
├── pom.xml                         Maven + dependencia del conector MySQL
├── .gitignore                      excluye db.properties, target/, nbproject/private/
├── db.properties.example           plantilla de conexión (se sube a GitHub)
├── db.properties                   TUS credenciales (lo creás vos; NO se sube)
├── db/
│   ├── db_paqueteria.sql           tu script original, sin cambios
│   └── datos_ejemplo.sql           opcional: 5 ciudades de ejemplo
└── src/main/java/paqueteria/
    ├── Main.java                   punto de entrada
    ├── conexion/
    │   ├── ConexionBD.java         abre conexiones leyendo db.properties
    │   └── ConfiguracionBDException.java
    ├── modelo/                     Camion, Camionero, Ciudad, Paquete, ConductorCamion
    ├── dao/                        CamionDAO, CamioneroDAO, CiudadDAO, PaqueteDAO,
    │                               ConductorCamionDAO, DaoUtil, DatosRelacionadosException
    ├── ui/                         FrmMenu, FrmCrud (base), FrmCamion, FrmCamionero,
    │                               FrmCiudad, FrmPaquete, FrmConductorCamion
    └── util/                       Validador, ValidacionException, Mensajes
```

## 2. Dependencias

Una sola, declarada en `pom.xml`: **MySQL Connector/J 8.4.0** (`com.mysql:mysql-connector-j`).
NetBeans la descarga sola la primera vez (hace falta internet). Requiere JDK 8 o superior.

## 3. Preparar la base de datos

1. Ejecutá `db/db_paqueteria.sql` en MySQL (Workbench, DBeaver o consola):
   `mysql -u TU_USUARIO -p < db/db_paqueteria.sql`
2. Opcional: ejecutá `db/datos_ejemplo.sql` para tener ciudades de ejemplo. También podés crearlas desde el módulo **Ciudades**.
   Sin ciudades no se pueden guardar paquetes.

## 4. Configurar la conexión (sin publicar credenciales)

1. En la raíz del proyecto, copiá `db.properties.example` como `db.properties`.
2. Completá `db.usuario` y `db.clave` (y host/puerto si no es `localhost:3306`).
3. `db.properties` está en `.gitignore`, así que Git nunca lo sube.

Si NetBeans no encuentra el archivo, indicá la ruta completa: clic derecho en el proyecto →
**Properties → Run → VM Options** → `-Dpaqueteria.config=C:\ruta\completa\db.properties`.

## 5. Abrir y ejecutar en NetBeans

1. **File → Open Project…** y elegí la carpeta `paqueteria` (el ícono de Maven aparece solo).
2. Esperá a que NetBeans descargue la dependencia (barra de progreso abajo a la derecha).
3. Clic derecho en el proyecto → **Clean and Build** y luego **Run** (o F6).
4. Se abre el menú principal con los módulos.

## 6. Cómo funciona cada módulo

Todos comparten la ventana base `FrmCrud`: formulario arriba, botones **Guardar / Actualizar / Eliminar /
Limpiar / Volver al menú** y una `JTable` con los registros.

- Al seleccionar una fila, sus datos pasan al formulario; se habilitan **Actualizar** y **Eliminar** y se
  deshabilita **Guardar**. **Limpiar** vuelve al modo "registro nuevo".
- Después de cada operación exitosa la tabla se vuelve a leer de MySQL y el formulario se limpia.
- Antes de eliminar se pide confirmación. Los cambios se guardan con auto-commit en cada operación.
- Todos los campos son obligatorios, incluidas las listas desplegables.

| Módulo | Tabla | Campos y validaciones |
|---|---|---|
| Camiones | `camion` | Modelo (máx. 45), Tipo (máx. 45), Potencia (entero 1–99999), Placa (letras, números y guiones, mín. 3; se guarda en mayúsculas) |
| Ciudades | `ciudad` | Nombre (máx. 45, solo letras, espacios, `.`, `'`, `-`) |
| Camioneros | `camionero` | Nombre y Apellidos (máx. 60, solo letras, espacios, `.`, `'`, `-`), Dirección (máx. 100), Salario (positivo, hasta 2 decimales), Población (máx. 45), DPI (13 dígitos), Teléfono (7 a 20 caracteres) |
| Paquetes | `paquete` | Descripción, Destinatario y Dirección (máx. 45), Peso (positivo, hasta 3 decimales), lista de Camioneros y lista de Ciudades |
| Camionero–Camión | `conductorcamion` | Lista de Camioneros, lista de Camiones, Fecha (`aaaa-mm-dd`, fecha real) |

**Eliminación con datos relacionados**
- Un **camionero** con paquetes o con asignaciones de camión no se puede eliminar.
- Un **camión** con asignaciones de camionero no se puede eliminar.
- Una **ciudad** que es destino de algún paquete no se puede eliminar.
- Paquetes y asignaciones se pueden eliminar libremente.
- En esos casos el mensaje indica cuántos registros dependen del que querés borrar.

**Notas de diseño**
- `fecha` (VARCHAR(50)) y `peso` (VARCHAR(45)) son texto en tu base; se guardan como `aaaa-mm-dd` y como número
  sin notación científica. Si una fila vieja tiene un valor que no cumple ese formato, se muestra vacío en el formulario.
- El DPI se valida con 13 dígitos. Si tu regla es otra, cambiá la constante `LONGITUD_DPI` en `Validador.java`.
- Ningún valor se valida como único (decisión tuya): la única clave que no se puede repetir es el ID, que genera
  MySQL con AUTO_INCREMENT. El ID aparece como "(automático)" y no se puede editar.
- Las operaciones usan `PreparedStatement` y `try-with-resources`: conexión, sentencia y resultado se cierran siempre.
- Los errores de conexión (servidor apagado, usuario o clave incorrectos, base inexistente, archivo de configuración
  faltante) muestran mensajes claros y nunca la contraseña.
- Las consultas se ejecutan en el hilo de la interfaz; es adecuado para volúmenes pequeños o medianos.

## 7. Pasos de prueba

**Ciudades**
1. Menú → Ciudades. Guardá con el nombre vacío → aviso; con `Ciudad 2` → se rechaza.
2. Guardá `Santa Rosa` → aparece en la tabla. Seleccionala, cambiá el nombre → **Actualizar**.
3. Eliminá una ciudad sin paquetes → se elimina; una ciudad usada por un paquete → aviso con la cantidad.

**Camiones**
1. Menú → Camiones. Guardá con todo vacío → aviso "El campo Modelo es obligatorio".
2. Probá Potencia `0`, `-5`, `4.5` y `abc` → cada una se rechaza. Placa `P1` o `P#123` → se rechaza.
3. Cargá Modelo `Volvo FH`, Tipo `Carga pesada`, Potencia `450`, Placa `p-123abc` → **Guardar**. Aparece en la tabla con ID nuevo y placa `P-123ABC`.
4. Seleccioná la fila, cambiá la Potencia a `500` → **Actualizar**. La tabla muestra 500.
5. Seleccioná la fila → **Eliminar** → **No**: no pasa nada. Repetí → **Sí**: desaparece.
6. Al volver a abrir el módulo el registro sigue eliminado, lo que confirma que se guardó en MySQL.

**Camioneros**
1. DPI con 12 dígitos o con letras → se rechaza. Salario `1.234` o `0` → se rechaza. Nombre `Juan3` → se rechaza. Teléfono `123` → se rechaza.
2. Guardá un camionero válido (DPI de 13 dígitos, teléfono `5555-1234`) → aparece en la tabla.
3. Actualizalo y comprobá el cambio en la tabla.

**Paquetes**
1. Dejá Camionero o Ciudad sin elegir → "Seleccione un valor en la lista…".
2. Peso `0` o `1E3` → se rechaza.
3. Guardá un paquete eligiendo camionero y ciudad de las listas → aparece con sus nombres en la tabla.
4. Seleccionalo, cambiá la ciudad → **Actualizar**.

**Camionero–Camión**
1. Fecha `2026-02-30` o `24/10/2026` → se rechaza. Fecha `2026-10-24` → se acepta.
2. Guardá una asignación con un camionero, un camión y una fecha válida → aparece en la tabla.

**Eliminación con datos relacionados**
1. Con el camionero del paso anterior ya asignado a un paquete y a un camión, intentá eliminarlo desde Camioneros → aviso con la cantidad de paquetes y asignaciones.
2. Intentá eliminar el camión asignado desde Camiones → aviso de asignaciones existentes.
3. Eliminá primero el paquete y la asignación; ahora sí podés eliminar camionero y camión.

**Errores de conexión**
1. Cambiá `db.clave` por una incorrecta → al abrir un módulo aparece "MySQL rechazó el usuario o la contraseña…" (sin mostrar la clave).
2. Poné `db.nombre=inexistente` → "La base de datos indicada no existe…".
3. Renombrá `db.properties` → mensaje con la ruta donde la busca.

## 8. Subir el proyecto a GitHub sin credenciales

1. Verificá que el archivo esté ignorado (debe mostrar la regla de `.gitignore`):
   `git check-ignore -v db.properties`
2. En la carpeta del proyecto:
   ```
   git init
   git add .
   git status
   ```
   Revisá que **no** aparezca `db.properties` (sí debe aparecer `db.properties.example`).
3. Primer commit y subida (creá antes un repositorio vacío en github.com):
   ```
   git commit -m "Gestión de transporte y paquetería"
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/TU_REPOSITORIO.git
   git push -u origin main
   ```
4. Si subiste `db.properties` por error: `git rm --cached db.properties`, hacé commit y **cambiá la contraseña
   de MySQL**, porque queda en el historial.

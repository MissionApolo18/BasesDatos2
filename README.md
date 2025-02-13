# Base de Datos

## Herramientas

- SQL
- Docker
- Power BI

## Temario

- Respaldo y recuperación
- Concurrencia y bloqueo
- Seguridad en BDD
- Desarrollo de aplicaciones en BDD
- Bases de datos distribuidas
- Gestión de datos masivos
-

## Repaso Bases de Datos 1

> **Base de datos:** Conjunto de datos que puedes ordenar y administrar

El **Sistema de Gestión de Bases de Datos (SGBD)** trabaja en tres niveles:

- Usuario / vista
- Lógico
- Físico

````mindmap 
Modelo de negocio - Semántica -> Modelo de datos -> Estructura formal de síntaxis
````

- **Concurrencia:** Cuando dos o mas usuarios intentan acceder al mismo recurso, al mismo tiempo, para leer o escribir.

- **Campo:** Es el *grano* más fino de la concurrencia, posteriormente el **registro**, seguido de la **tabla** y 
finalmente la **base de datos** es el grano más grueso.

- **Consistencia:** Es cuando todos los usuarios pueden ver el mismo estado, la integridad es parte para que los datos
  se guarden de manera correcta en el *modelo de datos*.

- **Integridad:** Tenemos dos tipos:
  - _La integridad de tabla:_ Todos los renglones deben de tener una llave primaria _(primarykey)_
  - Y _la integridad referencial:_ Garantiza que las relaciones entre tablas sean válidas y consistentes gracias a
 las llaves foráneas _(foreignkey)_

- **Cascada:** permite eliminar o modificar toda la información de un registro, sólo es visible en el join.
- **Restringir:** permite crear reglas y condiciones para modificar o eliminar registros
- **Bloqueo:** es lo que permite mantener un orden, alentando el sistema para garantizar la consistencia.
- **Transacción:** es el conjunto de operaciones que no se consideran terminadas hasta que la última no ha sido 
 concluida, la transacción empieza con start y termina con commit o rollback.
- **Buffer:** espacio temporal no consistente hasta no hacer commit.
- **Estado:** registro en la Base de datos.
- **Esquema:** Estructura de la base de datos.
- **Diccionario de datos:** guarda los metadatos
- **Data definition Language (DDL):** sirve para crear esquemas, create, alter y drop 
- **Data Management Language (DML):** sirve para manipular los registros, insert, update, delete & select 
- **View Management Language & CML**

## Respaldo y recuperación de una Base de Datos
- **Copia de seguridad:** El proceso de crear una copia de seguridad [sustantivo] mediante la copia de registros de datos de
  una BD, o registros de log de su log de transacciones.
- **Respaldo:** Una copia de datos que se puede usar para restaurar y recuperar los datos después de una falla.
  - Las copias de seguridad de una BD también se pueden usar para restaurar una copia de la BD en una nueva ubicación.

Descargar la bdd de classroom

```
DROP DATABASE IF EXISTS bdconcurrencia;
CREATE DATABASE bdconcurrencia CHARACTER SET utf8mb4;
USE bdconcurrencia;
CREATE TABLE cuentas ( id INTEGER UNSIGNED PRIMARY KEY, saldo DECIMAL(11,2) CHECK (saldo >= 0) );
INSERT INTO cuentas VALUES (1, 1000);
INSERT INTO cuentas VALUES (2, 2000);
INSERT INTO cuentas VALUES (3, 0);
 
 
-- Crear backup con timestamp en Windows
mysqldump.exe --defaults-file="S:\Backup\MySQL\parametros.cnf" -h localhost prueba > V:\prueba_%date:/=%_%time:~0,2%-%time:~3,2%-%time:~6,2%.sql
 
mysqldump --defaults-file="parametros_root.cnf" -h localhost colegio2807 > "COLEGIO2807_%DATE:~-4%%DATE:~3,2%%DATE:~0,2%%TIME:~0,2%%TIME:~3,2%%TIME:~6,2%.sql"
 
for /f "tokens=2 delims==" %I in ('wmic os get localdatetime /value') do set datetime=%I && set timestamp=%datetime:~0,4%%datetime:~4,2%%datetime:~6,2%_%datetime:~8,2%%datetime:~10,2%%datetime:~12,2% && mysqldump.exe --defaults-file="S:\Backup\MySQL\parametros.cnf" -h localhost prueba > V:\prueba_%timestamp%.sql
 
 
-- Respaldo incremental
insert into consumo values 
('00023450279', sysdate(),'7','V','925'),
('00023450287', sysdate(),'8','V','121'),
('00023450163', sysdate(),'9','V','210'),
('00023450260', sysdate(), '2', 'V', '75'),
('00023450279', sysdate(), '3', 'V', '780');
```

Meter todo este todo, entrar a cliente

```````
select YEAR (fecha) as anio, count(*) as noreg
from consumo
group by YEAR(fecha)
order by 1;
```````

En consola:

**mysqldump --deafults-file="C:\\ruta\parametroslocal.cnf" --no-create-info credito consumo --where="fecha" > =
CURRENT_DATE()"> C:\\ruta\20250212_credito_current.sql** y esto básicamente hace un respaldo de la base de datos.

**mysqldump --deafults-file="C:\\ruta\parametroslocal.cnf" --no-create-info credito consumo --where="fecha" > =
DATE_SUB(NOW(), INTERVAL 1 MONTH)" > C:\\ruta\20250212_credito_current.sql** otro respaldo, pero el formato de fecha es
diferente

````
 #!/bin/bash
 
# Directorio donde se guardarán los respaldos
BACKUP_DIR="/Users/omarmendoza/Documents/materias/BDII/Scripts/respaldo"
 
# Archivo de configuración de MySQL con credenciales
PARAM_FILE="/Users/omarmendoza/Documents/materias/BDII/Scripts/parametros_root.cnf"
 
# Obtener lista de bases de datos, excluyendo las del sistema
for database in $(/usr/local/mysql/bin/mysql --defaults-file="$PARAM_FILE" -e "SHOW DATABASES" -s --skip-column-names | grep -Ev "^(information_schema|performance_schema|sys|mysql)$"); 
do
    # Generar timestamp para el respaldo
    TIMESTAMP=$(date +%Y%m%d%H%M%S)
 
    # Ruta del archivo de respaldo
    BACKUP_FILE="$BACKUP_DIR/${TIMESTAMP}_${database}.sql"
 
    # Crear respaldo
    /usr/local/mysql/bin/mysqldump --defaults-file="$PARAM_FILE" \
    -h localhost --skip-lock-tables --routines --events --triggers "$database" > "$BACKUP_FILE"
 
    echo "Respaldo de $database completado: $BACKUP_FILE"
done
 
echo "Todos los respaldos han sido generados correctamente."
 
 
 
-- hacer un .bat CMD
@echo off
setlocal enabledelayedexpansion
set BACKUP_DIR=S:\Backup\MySQL
set TIMESTAMP=%date:~6,4%%date:~3,2%%date:~0,2%%time:~0,2%%time:~3,2%%time:~6,2%
 
for /F "tokens=*" %%D in ('mysql.exe --defaults-file="S:\Backup\MySQL\parametros.cnf" -s -N -e "SHOW DATABASES" ^| findstr /V "information_schema performance_schema mysql sys"') do (
    echo Haciendo respaldo de %%D...
    mysqldump.exe --defaults-file="S:\Backup\MySQL\parametros.cnf" %%D > "!BACKUP_DIR!\!TIMESTAMP!_%%D.sql"
)
 
echo Respaldo completado.
endlocal
 
 
-- PowerShell
# Configurar variables
$BackupDir = "S:\Backup\MySQL"
$ParamFile = "S:\Backup\MySQL\parametros.cnf"
$Timestamp = Get-Date -Format "yyyyMMddHHmmss"
 
# Obtener lista de bases de datos excluyendo las de sistema
$Databases = mysql.exe --defaults-file="$ParamFile" -s -N -e "SHOW DATABASES" | Where-Object {$_ -notmatch "^(information_schema|performance_schema|mysql|sys)$"}
 
# Respaldar cada base de datos
foreach ($Db in $Databases) {
    $BackupFile = "$BackupDir\$Timestamp`_$Db.sql"
    Write-Host "Respaldando $Db en $BackupFile..."
    mysqldump.exe --defaults-file="$ParamFile" $Db | Out-File -Encoding utf8 "$BackupFile"
}
 
Write-Host "Respaldo completado."
````

En linux respaldo.sh, en windows respaldo.bat y en powershell .ps1

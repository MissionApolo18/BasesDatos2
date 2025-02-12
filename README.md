# Base de Datos


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

**mysqldump --deafults-file="C:\\ruta\parametroslocal.cnf" --no-create-info credito consumo --where="fecha" > = CURRENT_DATE()"> C:\\ruta\20250212_credito_current.sql** y esto básicamente hace un respaldo de la base de datos.

**mysqldump --deafults-file="C:\\ruta\parametroslocal.cnf" --no-create-info credito consumo --where="fecha" > = DATE_SUB(NOW(), INTERVAL 1 MONTH)" > C:\\ruta\20250212_credito_current.sql** otro respaldo, pero el formato de fecha es diferente


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

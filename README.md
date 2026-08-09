clima-lock
Herramienta para Cinnamon (Linux Mint) que muestra la información meteorológica actual y la previsión de los próximos 4 días directamente en el mensaje de la pantalla de bloqueo.

La actualización se realiza mediante un temporizador systemd --user y los datos meteorológicos se obtienen de Open-Meteo, sin necesidad de una API key.

Ejemplo de salida
         25°C ☁️

Villanueva de la Serena
★ ★ ★ ★ ★ ★ ★ ★ ★ ★ 
09 dom ago 24 / 31 °C 🌤️
10 lun ago 24 / 29 °C ☁️
11 mar ago 24 / 29 °C 🌤️
12 mié ago 24 / 29 °C ☁️
Instalación
Instala el paquete Debian con:

sudo dpkg -i clima-lock_<version>.deb
Si faltan dependencias, puedes corregirlas mediante:

sudo apt --fix-broken install # solo si faltan dependencias
Si ya tenías instalado el temporizador y actualizas a una versión nueva, recarga la configuración de systemd y reinicia el servicio para aplicar inmediatamente los cambios:

systemctl --user daemon-reload
systemctl --user restart clima-lock.service
Configuración
1. Configurar la ciudad
Especifica únicamente el nombre de la ciudad, sin provincia:

clima-lock --ciudad "Villanueva de la Serena"
Si el nombre introducido coincide con varias localidades (por ejemplo, topónimos duplicados en distintas provincias o países), el comando presenta un listado numerado con la provincia y el país de cada resultado devuelto por la API de geocodificación, permitiendo seleccionar la coincidencia correcta. La selección se persiste como coordenadas (latitud/longitud) en el archivo de configuración, en lugar del nombre de texto libre: las actualizaciones automáticas posteriores consultan directamente esas coordenadas, eliminando la resolución por nombre en cada ejecución y, con ella, cualquier ambigüedad entre localidades homónimas.

2. Configurar la actualización automática
Instala el temporizador con el intervalo predeterminado de 30 minutos:

clima-lock --install # cada 30 min (por defecto)
Para establecer un intervalo diferente, especifica los minutos. Por ejemplo, cada 15 minutos:

clima-lock --install 15 # cada 15 min
Para eliminar el temporizador, consulta la sección Desinstalación.

3. Ejecutar manualmente
Para generar el mensaje, imprimirlo y aplicarlo inmediatamente:

clima-lock
Desinstalación
Eliminar únicamente el temporizador
Mantiene el programa instalado, pero desactiva las actualizaciones automáticas:

clima-lock --uninstall
Eliminar el paquete completo
sudo apt remove clima-lock
Eliminar el paquete y la configuración
Esta opción también elimina la configuración almacenada, incluida la ciudad guardada en:

~/.config/mint-weather-lock/
Ejecuta:

sudo apt purge clima-lock
rm -rf ~/.config/mint-weather-lock
Antes de desinstalar el paquete, se recomienda ejecutar:

clima-lock --uninstall
Esto evita que el temporizador quede instalado de forma independiente o huérfana.

Tipografía y alineación
Importante: el centrado y la disposición de las columnas se calculan contando caracteres. Por este motivo, el mensaje debe mostrarse utilizando una fuente monoespaciada. Con una fuente proporcional, las columnas pueden quedar desalineadas.

Instala la fuente Hack:

sudo apt install fonts-hack
Configura Cinnamon para utilizarla en el mensaje de la pantalla de bloqueo:

gsettings set org.cinnamon.desktop.screensaver font-message "Hack 14"
También funcionan correctamente:

Ubuntu Mono
DejaVu Sans Mono
JetBrains Mono
Para comprobar las fuentes monoespaciadas disponibles en el sistema:

fc-list :spacing=100 family
Notas
Si el campo «Mostrar este mensaje...» de las Preferencias del sistema aparece vacío, es normal. El mensaje comienza con un carácter invisible de relleno.
Para comprobar el contenido real configurado en Cinnamon:
gsettings get org.cinnamon.desktop.screensaver default-message
Si no existe conexión a Internet, el script conserva el mensaje anterior y no modifica el contenido actualmente mostrado.

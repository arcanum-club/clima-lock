# clima-lock

Muestra el tiempo actual y la previsión de 4 días en el mensaje de la pantalla de bloqueo de **Cinnamon** (Linux Mint), actualizado solo mediante un temporizador de `systemd --user`. Datos de [Open-Meteo](https://open-meteo.com/) (sin API key).

```
              25°C ☁️

     Villanueva de la Serena
★  ★  ★  ★  ★  ★  ★  ★  ★  ★  ★  ★
09    dom    ago    24 / 31 °C 🌤️
10    lun    ago    24 / 29 °C ☁️
11    mar    ago    24 / 29 °C 🌤️
12    mié    ago    24 / 29 °C ☁️
```

## Instalación

```bash
sudo dpkg -i clima-lock_<version>.deb
sudo apt --fix-broken install   # solo si faltan dependencias
```

Si ya tenías el temporizador instalado y actualizas a una versión nueva, recárgalo para que el cambio se aplique ya:
```bash
systemctl --user daemon-reload
systemctl --user restart clima-lock.service
```

## Configuración

**1. Tu ciudad** (solo el nombre, sin provincia):
```bash
clima-lock --ciudad "Villanueva de la Serena"
```

**2. Actualización automática:**
```bash
clima-lock --install          # cada 30 min (por defecto)
clima-lock --install 15       # cada 15 min
```

**Desinstalar el temporizador:** ver la sección [Desinstalación](#desinstalación) más abajo.

**Probar manualmente** (imprime el mensaje y lo aplica ya):
```bash
clima-lock
```

## Desinstalación

**Quitar solo el temporizador** (deja el programa instalado, deja de actualizar el mensaje automáticamente):
```bash
clima-lock --uninstall
```

**Quitar el paquete completo:**
```bash
sudo apt remove clima-lock
```

**Quitar el paquete y también tu configuración** (ciudad guardada en `~/.config/mint-weather-lock/`):
```bash
sudo apt purge clima-lock
rm -rf ~/.config/mint-weather-lock
```

> Antes de desinstalar el paquete conviene ejecutar `clima-lock --uninstall` para que el temporizador no quede huérfano.

## Tipografía — importante

El centrado y las columnas se calculan contando caracteres, así que **solo se ven bien con una fuente monoespaciada**. Con una fuente normal el texto queda descuadrado.

```bash
sudo apt install fonts-hack
gsettings set org.cinnamon.desktop.screensaver font-message "Hack 14"
```

También funcionan bien `Ubuntu Mono`, `DejaVu Sans Mono` o `JetBrains Mono` (probablemente ya tienes alguna instalada — compruébalo con `fc-list :spacing=100 family`).

## Notas

- Si el campo "Mostrar este mensaje..." de Preferencias del sistema aparece vacío, es normal: el mensaje empieza con un carácter invisible de relleno. Comprueba el contenido real con `gsettings get org.cinnamon.desktop.screensaver default-message`.
- Sin conexión a internet, el script deja el mensaje anterior sin tocar.

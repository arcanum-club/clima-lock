#!/usr/bin/env python3
"""
Mint Weather Lock (versión sencilla)
Uso:
  python3 clima_lock.py --ciudad "Madrid"
  python3 clima_lock.py
  python3 clima_lock.py --install            # instala servicio+timer (cada 30 min)
  python3 clima_lock.py --install 15         # instala con intervalo de 15 min
  python3 clima_lock.py --uninstall          # desinstala servicio+timer
"""
import configparser, os, subprocess, sys, datetime
from pathlib import Path
import requests

CFGDIR=Path.home()/".config"/"mint-weather-lock"
CFGDIR.mkdir(parents=True, exist_ok=True)
CFG=CFGDIR/"config.ini"

SYSTEMD_USER_DIR=Path.home()/".config"/"systemd"/"user"
SERVICE_NAME="clima-lock.service"
TIMER_NAME="clima-lock.timer"

ICONOS_DIA={0:"☀️",1:"🌤️",2:"⛅️",3:"☁️",45:"🌫️",48:"🌫️",51:"🌦️",53:"🌦️",55:"🌧️",61:"🌧️",63:"🌧️",65:"🌧️",71:"❄️",73:"❄️",75:"❄️",80:"🌧️",81:"🌧️",82:"🌧️",95:"⛈️"}
ICONOS_NOCHE={0:"🌙",1:"🌙",2:"🌥️",3:"☁️",45:"🌫️",48:"🌫️",51:"🌦️",53:"🌦️",55:"🌧️",61:"🌧️",63:"🌧️",65:"🌧️",71:"❄️",73:"❄️",75:"❄️",80:"🌧️",81:"🌧️",82:"🌧️",95:"⛈️"}
ICONOS=ICONOS_DIA  # se usa para el pronóstico de días futuros

def hay_internet():
    try:
        requests.head("https://duckduckgo.com", timeout=5)
        return True
    except requests.RequestException:
        return False

def guardar(ciudad):
    c=configparser.ConfigParser()
    c["General"]={"ciudad":ciudad}
    with open(CFG,"w") as f:c.write(f)

def leer():
    if not CFG.exists(): return None
    c=configparser.ConfigParser(); c.read(CFG)
    return c.get("General","ciudad",fallback=None)

def geocode(nombre):
    u="https://geocoding-api.open-meteo.com/v1/search"
    params={"name":nombre,"count":5,"language":"es","format":"json"}
    j=requests.get(u,params=params,timeout=10).json()
    resultados=j.get("results")
    if not resultados and "," in nombre:
        # Reintenta solo con la parte antes de la primera coma
        # (ej: "Sant Feliu de Guíxols, Girona" -> "Sant Feliu de Guíxols")
        params["name"]=nombre.split(",")[0].strip()
        j=requests.get(u,params=params,timeout=10).json()
        resultados=j.get("results")
    if not resultados:
        print(f'No se encontró "{nombre}". Prueba solo con el nombre del pueblo/ciudad, sin provincia.')
        sys.exit(1)
    r=resultados[0]
    return r["latitude"],r["longitude"],r["name"]

def clima(lat,lon):
    u=("https://api.open-meteo.com/v1/forecast"
       f"?latitude={lat}&longitude={lon}"
       "&current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code,is_day,apparent_temperature,pressure_msl"
       "&hourly=temperature_2m,weather_code"
       "&forecast_days=5&timezone=auto")
    return requests.get(u,timeout=10).json()

DIAS_ES=["Lunes","Martes","Miércoles","Jueves","Viernes","Sábado","Domingo"]
DIAS_ES_ABR=["lun","mar","mié","jue","vie","sáb","dom"]
MESES_ES_ABR=["ene","feb","mar","abr","may","jun","jul","ago","sep","oct","nov","dic"]
MESES_ES=["enero","febrero","marzo","abril","mayo","junio","julio","agosto","septiembre","octubre","noviembre","diciembre"]

def mensaje(ciudad,j):
    c=j["current"]
    icono_actual = ICONOS_DIA if c.get("is_day",1) else ICONOS_NOCHE
    if c.get("time"):
        hoy=datetime.date.fromisoformat(c["time"][:10])
    else:
        hoy=datetime.date.today()

    BLANCO = "\u00A0"

    # Diseño para la pantalla de bloqueo: temperatura centrada encima
    # de la ciudad, sin espacio entre la ciudad y las estrellas.
    # El icono se excluye del cálculo de centrado (su ancho real en pantalla
    # no coincide con el número de caracteres que ocupa en la cadena).
    temp_valor = f"{round(c['temperature_2m'])}°C"
    icono_temp = icono_actual.get(c['weather_code'],'?')
    ciudad_linea = ciudad

    horas=[datetime.datetime.fromisoformat(t) for t in j["hourly"]["time"]]
    temps=j["hourly"]["temperature_2m"]
    codigos=j["hourly"]["weather_code"]
    lineas_pronostico=[]
    for i in range(4):
        dia=hoy+datetime.timedelta(days=i)
        inicio=datetime.datetime.combine(dia,datetime.time(12,30))
        fin=inicio+datetime.timedelta(days=1)
        idxs=[k for k,h in enumerate(horas) if inicio<=h<fin]
        if not idxs:
            continue
        tmax=round(max(temps[k] for k in idxs))
        tmin=round(min(temps[k] for k in idxs))
        if i == 0:
            # Hoy: usa el mismo código que la línea de "condiciones actuales"
            # de arriba, para que ambos iconos coincidan.
            icono_cod = c['weather_code']
        else:
            icono_cod=codigos[idxs[0]]
        dia_abr=DIAS_ES_ABR[dia.weekday()]
        mes_abr=MESES_ES_ABR[dia.month-1]
        sep_tras_dia = "\u00A0"*4
        lineas_pronostico.append(f"{dia.day:02d}{'\u00A0'*4}{dia_abr}{sep_tras_dia}{mes_abr}{'\u00A0'*4}{tmin:>2} / {tmax:>2} °C {ICONOS_DIA.get(icono_cod,'?')}")

    # Las estrellas quedan inmediatamente debajo de la ciudad, como en el diseño anterior.
    ancho_max = max((len(l) for l in lineas_pronostico), default=20)
    ESTRELLA = "★  "
    n_estrellas = max(ancho_max // len(ESTRELLA), 1)
    linea_estrellas = (ESTRELLA * n_estrellas).rstrip()
    relleno_e = max(ancho_max - len(linea_estrellas), 0)

    # Mantener el bloque compacto y centrado visualmente en Cinnamon.
    ancho = max(ancho_max, len(ciudad_linea), len(temp_valor))
    def centrar(texto):
        total = max(ancho - len(texto), 0)
        return BLANCO*(total//2) + texto + BLANCO*(total-total//2)

    margen_izq_temp = max(ancho - len(temp_valor), 0) // 2
    linea_temp = BLANCO*margen_izq_temp + temp_valor + BLANCO + icono_temp

    out=[
        BLANCO,
        linea_temp,
        BLANCO,
        centrar(ciudad_linea),
        linea_estrellas + BLANCO*relleno_e,
    ]
    out.extend(lineas_pronostico)
    return "\n".join(out)

def entorno_grafico():
    """Detecta DISPLAY y DBUS_SESSION_BUS_ADDRESS en tiempo de ejecución.
    Necesario cuando el script corre desde una unidad systemd --user
    activada globalmente (sin variables capturadas al instalar)."""
    env = os.environ.copy()
    if not env.get("DISPLAY"):
        # Busca un socket X11 activo (típicamente :0 o :1)
        x11dir = "/tmp/.X11-unix"
        if os.path.isdir(x11dir):
            socks = sorted(os.listdir(x11dir))
            if socks:
                env["DISPLAY"] = ":" + socks[0].lstrip("X")
        env.setdefault("DISPLAY", ":0")
    if not env.get("DBUS_SESSION_BUS_ADDRESS"):
        bus_path = f"/run/user/{os.getuid()}/bus"
        if os.path.exists(bus_path):
            env["DBUS_SESSION_BUS_ADDRESS"] = f"unix:path={bus_path}"
    return env

def lock(text):
    env = entorno_grafico()
    r = subprocess.run(
        ["gsettings","set","org.cinnamon.desktop.screensaver","default-message",text],
        env=env, check=False, capture_output=True, text=True
    )
    if r.returncode != 0:
        print(f"Aviso: no se pudo actualizar el mensaje de bloqueo ({r.stderr.strip()})", file=sys.stderr)

# ---------------- Instalación systemd ----------------

def _uid():
    return os.getuid()

def _script_path():
    return str(Path(__file__).resolve())

def _display_env():
    return os.environ.get("DISPLAY", ":0")

def instalar(intervalo_min):
    SYSTEMD_USER_DIR.mkdir(parents=True, exist_ok=True)
    script = _script_path()

    service_content = f"""[Unit]
Description=Actualiza mensaje de clima en pantalla de bloqueo
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 {script}
"""

    timer_content = f"""[Unit]
Description=Ejecuta clima-lock cada {intervalo_min} minutos

[Timer]
OnBootSec=1min
OnUnitActiveSec={intervalo_min}min
Persistent=true

[Install]
WantedBy=timers.target
"""

    (SYSTEMD_USER_DIR/SERVICE_NAME).write_text(service_content)
    (SYSTEMD_USER_DIR/TIMER_NAME).write_text(timer_content)

    print(f"Escritos:\n  {SYSTEMD_USER_DIR/SERVICE_NAME}\n  {SYSTEMD_USER_DIR/TIMER_NAME}")

    cmds = [
        ["systemctl","--user","daemon-reload"],
        ["systemctl","--user","enable","--now",TIMER_NAME],
    ]
    for cmd in cmds:
        r = subprocess.run(cmd, capture_output=True, text=True)
        if r.returncode != 0:
            print(f"Error ejecutando {' '.join(cmd)}:\n{r.stderr}")
            sys.exit(1)

    print(f"Instalado y activado. Se ejecutará cada {intervalo_min} minutos.")
    print("Verifica con: systemctl --user status clima-lock.timer")
    print("Logs en vivo: journalctl --user -u clima-lock.service -f")

def desinstalar():
    subprocess.run(["systemctl","--user","disable","--now",TIMER_NAME], capture_output=True)
    for f in (SYSTEMD_USER_DIR/SERVICE_NAME, SYSTEMD_USER_DIR/TIMER_NAME):
        if f.exists():
            f.unlink()
            print(f"Eliminado: {f}")
    subprocess.run(["systemctl","--user","daemon-reload"], capture_output=True)
    print("Servicio y timer desinstalados.")

# ---------------- Main ----------------

if __name__=="__main__":
    args = sys.argv[1:]

    if args and args[0] == "--install":
        intervalo = 30
        if len(args) >= 2:
            try:
                intervalo = int(args[1])
            except ValueError:
                print("El intervalo debe ser un número de minutos, ej: --install 15")
                sys.exit(1)
        if not leer():
            print('Aviso: no has configurado ciudad todavía. Ejecuta primero:')
            print('  python3 clima_lock.py --ciudad "Tu Ciudad"')
        instalar(intervalo)
        sys.exit(0)

    if args and args[0] == "--uninstall":
        desinstalar()
        sys.exit(0)

    if len(args)>=2 and args[0]=="--ciudad":
        guardar(" ".join(args[1:]))
        print("Ciudad guardada.")
        sys.exit(0)

    ciudad=leer()
    if not ciudad:
        print('Primero ejecuta: python3 clima_lock.py --ciudad "Tu Ciudad"')
        sys.exit(1)
    if not hay_internet():
        print("Sin conexión a internet, se deja el mensaje actual sin cambios.")
        sys.exit(0)
    lat,lon,nombre=geocode(ciudad)
    txt=mensaje(nombre,clima(lat,lon))
    print(txt)
    lock(txt)
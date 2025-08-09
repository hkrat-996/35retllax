#!/bin/bash

# Colores
RED='\033[1;31m'
GREEN='\033[1;32m'
YELLOW='\033[1;33m'
BLUE='\033[1;34m'
CYAN='\033[1;36m'
NC='\033[0m' # Sin color

# Configuración
LOG_FILE="$HOME/.35retllax.log"
KNOWN_DEVICES="$HOME/.35retllax_devices.log"

# Función para notificaciones
notify() {
    if [ -f "/data/data/com.termux/files/usr/bin/termux-notification" ]; then
        termux-notification -t "35retllax" -c "$1"
    else
        echo -e "${GREEN}[!] $1${NC}"
    fi
}

# Función para escanear dispositivos
scan_devices() {
    router_ip=$(ip route | grep "default via" | awk '{print $3}')
    network=$(echo $router_ip | cut -d '.' -f1-3)
    
    echo -e "\n${GREEN}[+] Escaneando red (${network}.0/24)...${NC}"
    echo -e "${YELLOW}Router: ${router_ip}${NC}\n"
    
    for i in {1..254}; do
        ip="${network}.${i}"
        if ping -c 1 -W 1 "$ip" &>/dev/null; then
            mac=$(arp -n "$ip" | awk '{print $3}' | grep -v "incomplete")
            if [ -n "$mac" ]; then
                if ! grep -q "$mac" "$KNOWN_DEVICES" 2>/dev/null; then
                    vendor=$(curl -s "https://api.macvendors.com/$(echo "$mac" | tr -d ':')")
                    echo -e "${CYAN}[+] Nuevo dispositivo: ${ip} | ${mac} | ${vendor}${NC}"
                    echo "$mac $ip $vendor" >> "$KNOWN_DEVICES"
                    notify "¡Nuevo dispositivo en la red! IP: ${ip} (${vendor})"
                fi
            fi
        fi
    done
}

# Menú principal
main() {
    clear
    echo -e "${GREEN}"
    echo "   _____ _____ _____ _____ _     _ "
    echo "  |  _  |  _  |   __|  |  |_|___| |"
    echo "  |   __|     |__   |  |  | |  .| |"
    echo "  |__|  |__|__|_____|_____|_|___|_|"
    echo -e "${NC}"
    echo -e "${YELLOW}Monitor de Red Local + Notificaciones${NC}"
    echo -e "${BLUE}Sin root | GitHub: tu-usuario/35retllax${NC}"
    echo "----------------------------------------"
    
    echo -e "\n1) Escanear dispositivos ahora"
    echo "2) Modo vigilancia (cada 5 minutos)"
    echo "3) Ver historial de dispositivos"
    echo "4) Actualizar desde GitHub"
    echo "5) Salir"
    echo -n -e "\n${RED}Selecciona una opción: ${NC}"
    read opcion

    case $opcion in
        1) scan_devices ;;
        2) while true; do scan_devices; sleep 300; done ;;
        3) [ -f "$KNOWN_DEVICES" ] && cat "$KNOWN_DEVICES" || echo "No hay datos." ;;
        4) update_tool ;;
        5) exit 0 ;;
        *) echo -e "${RED}Opción inválida.${NC}"; sleep 1; main ;;
    esac
}

# Función para actualizar
update_tool() {
    echo -e "\n${GREEN}[+] Actualizando 35retllax...${NC}"
    cd "$HOME/35retllax" && git pull
    chmod +x 35retllax.sh
    echo -e "${YELLOW}¡Actualización completada!${NC}"
}

# Inicio
mkdir -p "$HOME/35retllax"
main

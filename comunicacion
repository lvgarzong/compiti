# Librerías
from __future__ import print_function
from numpy import *
from can.bus import BusState
import os
import can
import time

os.system('sudo ifconfig can1 down')
os.system('sudo ip link set can1 type can bitrate 100000')
os.system('sudo ifconfig can1 up')

# Configuración específica del bus
bus = can.interface.Bus(bustype = 'socketcan_ctypes',
                channel = 'can1',
                bitrate = 100000)


id_tags = []
data    = []
null    = [None]

# Declaración de variables globales
n     = 2             # Número de sensores
count = 0              # Contador para reinicio del ciclo
t     = 0.03            # Tiempo entre de envío de mensajes sensor a sensor en seg.
T     = 2.5 - ((n - 1) * t) +  t  # Tiempo del ciclo en seg.

aux   = True           # Variable booleana auxiliar para seguir con el ciclo en
                       # caso de detección de errores
mV    = 200          #cambiar a 724
div   = 255


# Función principal: envío, recepción y almacenamiento de datos
def send_checkback_msg(i):
    msg = can.Message(arbitration_id = 0x05,            # Configuración del msg, estamos
                      data = [4,0,0,6,6,6,0,0],      # enviando un mensaje cualquiera
                      is_extended_id = True)
    try:
        bus.send(msg)     # Envío del mensaje  msg al bus
        print("Mensaje enviado a Sensor: {}".format(i))
        try:
            msg_received = list(receive_msg(i))        # Uso de función receiveMsg
            pos = len(msg_received)-1     # Posición del dato en el mensaje
            if msg_received[pos] is not None:    # Se asume que si el dato es cero: error/sin sensor
                msg_rc = msg_received[pos]*(mV/div)
                print("Mensaje recibido: {}".format(msg_rc))
                save_data(i,msg_rc)
            else:
                print("Error: el sensor #{} no está conectado o no existe".format(i))
                save_data(i, 0)
        except Exception as e:
            print(e)
    except can.CanError:
        aux = False
        save_data(i, 0)
        print("Mensage NO enviado/Ningún sensor conectado")

# Función para recibir el mensaje
def receive_msg(i):
    # Recibe el mensaje y lo almancena
    response_msg = bus.recv(t)
    # Se asegura de que el mensaje no sea nulo y que tenga el id de la MTU = 0
    if (response_msg is not None) and (response_msg.arbitration_id == 0):
        return (response_msg.data)
    else:
        return null
        print("Error: el sensor #{} no está conectado o no existe".format(i))

# Función para almacenar los datos en el vector a
def save_data(i,d):
    
    data_aux = []
    data_aux.append(d)

# Ejecución del programa main
if __name__ == "__main__":
    while aux:
        count += 1                 #contador x empieza en 1
        send_checkback_msg(count)  #ejecuta función sendCheckbackMsg
        if count == n:             #cuando el contador x sea igual a n
            time.sleep(T)          # espera un tiempo T para reiniciar el ciclo
            count = 0              #reseteando el valor de x a 0
            # a = zeros(n)           #reseteando el vector a ceros
        aux = True                 #reseteando la variable bool aux a True para
                                   # continuar con el ciclo en caso de que haya
                                   # ocurrido un error

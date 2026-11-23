import socket

PUERTO_TCP_MINIMO = 1
PUERTO_TCP_MAXIMO = 65535
TIEMPO_ESPERA_SEGUNDOS = 0.5


def es_puerto_tcp_valido(puerto):
    if not isinstance(puerto, int) or isinstance(puerto, bool):
        return False
    return PUERTO_TCP_MINIMO <= puerto <= PUERTO_TCP_MAXIMO


def probar_conexion_tcp(host, puerto, tiempo_espera):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(tiempo_espera)
    try:
        resultado = sock.connect_ex((host, puerto))
        if resultado == 0:
            return {
                "estado": "abierto",
                "puerto": puerto,
                "motivo": "conexion_exitosa",
            }
        return {
            "estado": "cerrado",
            "puerto": puerto,
            "motivo": f"codigo_{resultado}",
        }
    except TimeoutError:
        return {
            "estado": "cerrado",
            "puerto": puerto,
            "motivo": "tiempo_agotado",
        }
    except OSError as error:
        return {
            "estado": "cerrado",
            "puerto": puerto,
            "motivo": str(error),
        }
    finally:
        sock.close()


def formatear_aviso(nivel, detalle):
    return f"[{nivel.upper()}] {detalle}"


def emitir_aviso(mensaje):
    print(mensaje)


def auditar_puertos(host, lista_puertos, tiempo_espera=TIEMPO_ESPERA_SEGUNDOS):
    for puerto in lista_puertos:
        if not es_puerto_tcp_valido(puerto):
            aviso = formatear_aviso(
                "alerta critica",
                f"Puerto invalido fuera de rango TCP: {puerto}",
            )
            emitir_aviso(aviso)
            continue

        resultado = probar_conexion_tcp(host, puerto, tiempo_espera)

        if resultado["estado"] == "abierto":
            aviso = formatear_aviso(
                "alerta seguridad",
                f"Puerto sensible ABIERTO detectado -> {puerto} en host {host}",
            )
        else:
            aviso = formatear_aviso(
                "info",
                f"Puerto cerrado -> {puerto} ({resultado['motivo']})",
            )

        emitir_aviso(aviso)


def main():
    target_ip = "127.0.0.1"
    lista_p = [21, 80, 70000, 443, -5, 8080]
    auditar_puertos(target_ip, lista_p)


if __name__ == "__main__":
    main()

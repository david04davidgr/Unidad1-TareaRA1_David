# Documentación y prueba del programa

En este paso se procede a la documentación del codigo mediante comentarios a lo largo de los dos archivos presentes de codigo fuente, ejecución del programa y realizar algunos arreglos para que se ejecute correctamente el programa

## Documentación

A continuación se muestran los dos archivos documentados:

**1-. main_app.py**
```python
# main_app.py

# Importar la clase desde el otro archivo (módulo)
from lavadero import Lavadero

# MODIFICACIÓN CLAVE AQUÍ: La función ahora acepta 3 argumentos
def ejecutarSimulacion(lavadero, prelavado, secado_mano, encerado):
    """
    Simula el proceso de lavado para un vehículo con las opciones dadas.
    Ahora acepta una instancia de lavadero.

    :param lavadero: Instancia de Lavadero.
    :param prelavado: bool, True si se solicita prelavado a mano.
    :param secado_mano: bool, True si se solicita secado a mano.
    :param encerado: bool, True si se solicita encerado.
    """
    
    print("--- INICIO: Prueba de Lavado con Opciones Personalizadas ---")
    
    # Mostrar las opciones solicitadas
    print(f"Opciones solicitadas: [Prelavado: {prelavado}, Secado a mano: {secado_mano}, Encerado: {encerado}]")

    # 1. Iniciar el lavado
    try:
        # Esto establece las opciones y pasa a Fase 0 (Inactivo, pero Ocupado=True)
        lavadero.hacerLavado(prelavado, secado_mano, encerado)
        print("\nCoche entra. Estado inicial:")
        lavadero.imprimir_estado()

        # 2. Avanza por las fases
        print("\nAVANZANDO FASE POR FASE:")
        
        # Usamos un contador para evitar bucles infinitos en caso de error o bucles inesperados
        pasos = 0
        while lavadero.ocupado and pasos < 20: 
            # El cobro ahora ocurre en la primera llamada a avanzarFase (transición 0 -> 1)
            lavadero.avanzarFase()
            print(f"-> Fase actual: ", end="")
            lavadero.imprimir_fase()
            print() 
            pasos += 1
        
        print("\n----------------------------------------")
        print("Lavado completo. Estado final:")
        lavadero.imprimir_estado()
        print(f"Ingresos acumulados: {lavadero.ingresos:.2f} €")
        print("----------------------------------------")
        
    except ValueError as e: # Captura la excepción de regla de negocio (Requisito 2)
        print(f"ERROR DE ARGUMENTO: {e}")
    except RuntimeError as e: # Captura la excepción de estado (Requisito 3)
        print(f"ERROR DE ESTADO: {e}")
    except Exception as e:
        print(f"ERROR INESPERADO: {e}")


    # Punto de entrada (main): Aquí pasamos los parámetros
    if __name__ == "__main__":
    
        lavadero_global = Lavadero() # Usamos una única instancia para acumular ingresos
        
        # EJEMPLO 1: Lavado completo con prelavado, secado a mano, con encerado (Requisito 8 y 14)
        # Precio esperado: 5.00 + 1.50 + 1.00 + 1.20 = 8.70 €
        print("\n=======================================================")
        print("EJEMPLO 1: Prelavado (S), Secado a mano (S), Encerado (S)")
        ejecutarSimulacion(lavadero_global, prelavado=True, secado_mano=True, encerado=True)
        
        # EJEMPLO 2: Lavado rápido sin extras (Requisito 9)
        # Precio esperado: 5.00 €
        print("\n=======================================================")
        print("EJEMPLO 2: Sin extras (Prelavado: N, Secado a mano: N, Encerado: N)")
        ejecutarSimulacion(lavadero_global, prelavado=False, secado_mano=False, encerado=False)

        # EJEMPLO 3: Lavado con encerado, pero sin secado a mano (Debe lanzar ValueError - Requisito 2)
        print("\n=======================================================")
        print("EJEMPLO 3: ERROR (Encerado S, Secado a mano N)")
        ejecutarSimulacion(lavadero_global, prelavado=False, secado_mano=False, encerado=True)

        # EJEMPLO 4: Lavado con prelavado a mano (Requisito 4 y 10)
        # Precio esperado: 5.00 + 1.50 = 6.50 €
        print("\n=======================================================")
        print("EJEMPLO 4: Prelavado (S), Secado a mano (N), Encerado (N)")
        ejecutarSimulacion(lavadero_global, prelavado=True, secado_mano=False, encerado=False)

```

**2-. lavadero.py**
```python
# lavadero.py

class Lavadero:
    """
    Simula el estado y las operaciones de un túnel de lavado de coches.
    Cumple con los requisitos de estado, avance de fase y reglas de negocio.
    """
    # Se declaran las constantes con los valores de cada de las fases del lavadero
    FASE_INACTIVO = 0
    FASE_COBRANDO = 1
    FASE_PRELAVADO_MANO = 2
    FASE_ECHANDO_AGUA = 3
    FASE_ENJABONANDO = 4
    FASE_RODILLOS = 5
    FASE_SECADO_AUTOMATICO = 6
    FASE_SECADO_MANO = 7
    FASE_ENCERADO = 8

    # Constructor para la inicialización del lavadero
    def __init__(self):
        """
        Constructor de la clase. Inicializa el lavadero.
        Cumple con el requisito 1.
        """
        self.__ingresos = 0.0
        self.__fase = self.FASE_INACTIVO
        self.__ocupado = False
        self.__prelavado_a_mano = False
        self.__secado_a_mano = False
        self.__encerado = False
        self.terminar() 

    @property
    def fase(self):
        return self.__fase

    @property
    def ingresos(self):
        return self.__ingresos

    @property
    def ocupado(self):
        return self.__ocupado
    
    @property
    def prelavado_a_mano(self):
        return self.__prelavado_a_mano

    @property
    def secado_a_mano(self):
        return self.__secado_a_mano

    @property
    def encerado(self):
        return self.__encerado

    # Asigna la fase en "Inactivo" y resetea las opciones de lavado a "false", requisito 1
    def terminar(self):
        self.__fase = self.FASE_INACTIVO
        self.__ocupado = False
        self.__prelavado_a_mano = False
        self.__secado_a_mano = False
        self.__encerado = False
    
    # Inicializa un ciclo de lavado segun los parametros dados y valida que los parametros recibidos sean correctos 
    def hacerLavado(self, prelavado_a_mano, secado_a_mano, encerado):
        """
        Inicia un nuevo ciclo de lavado, validando reglas de negocio.
        
        :raises RuntimeError: Si el lavadero está ocupado (Requisito 3).
        :raises ValueError: Si se intenta encerar sin secado a mano (Requisito 2).
        """
        if self.__ocupado: # Si el lavadero está ocupado no se podra iniciar otro lavado diferente
            raise ValueError("No se puede iniciar un nuevo lavado mientras el lavadero está ocupado")
        
        if not secado_a_mano and encerado: # Si no esta secado a mano el coche no se podra seleccionar el encerado
            raise ValueError("No se puede encerar el coche sin secado a mano")
        
        # Se asigna el estado del lavado, se reserva el lavadero para este lavado y se asignan las opciones marcadas para el lavado
        self.__fase = self.FASE_INACTIVO  
        self.__ocupado = True
        self.__prelavado_a_mano = prelavado_a_mano
        self.__secado_a_mano = secado_a_mano
        self.__encerado = encerado
        
    # Calcula el coste del lavado a cobrar
    def _cobrar(self):
        """
        Calcula y añade los ingresos según las opciones seleccionadas (Requisitos 4-8).
        Precio base: 5.00€ (Implícito, 5.00€ de base + 1.50€ de prelavado + 1.00€ de secado + 1.20€ de encerado = 8.70€)
        """
        coste_lavado = 5.00
        
        if self.__prelavado_a_mano:
            coste_lavado += 1.50 
        
        if self.__secado_a_mano:
            coste_lavado += 1.00 
            
        if self.__encerado:
            coste_lavado += 1.20 
            
        self.__ingresos += coste_lavado
        return coste_lavado

    # Avanza la fase actual segun el transcurso de la ejecución y las opciones seleccionadas
    def avanzarFase(self):

        if not self.__ocupado: # Si esta ocupado no permitira avanzar de faser 
            return

        """
        Si esta inactivo se asigna el coste cobrado, 
        se actualiza a la fase cobrando y se muestra por consola el coste cobrado
        """
        if self.__fase == self.FASE_INACTIVO: 
            coste_cobrado = self._cobrar()
            self.__fase = self.FASE_COBRANDO
            print(f" (COBRADO: {coste_cobrado:.2f} €) ", end="")

        elif self.__fase == self.FASE_COBRANDO:
            if self.__prelavado_a_mano:
                self.__fase = self.FASE_PRELAVADO_MANO # Si está en la fase "cobrando" y se ha marcado "prelavado a mano" pasa a la fase "prelavado a mano" 
            else:
                self.__fase = self.FASE_ECHANDO_AGUA # Si está en la fase "cobrando" pero no se ha marcado "prelavado a mano" pasa a la fase "echando agua"
        
        elif self.__fase == self.FASE_PRELAVADO_MANO:
            self.__fase = self.FASE_ECHANDO_AGUA # Si está en la fase "prelavado a mano" pasa a la fase "echando agua"
        
        elif self.__fase == self.FASE_ECHANDO_AGUA:
            self.__fase = self.FASE_ENJABONANDO # Si está en la fase "echando agua" pasa a la fase "enjabonando"

        elif self.__fase == self.FASE_ENJABONANDO:
            self.__fase = self.FASE_RODILLOS # Si esta en la fase "enjabonando" pasa a la fase "rodillos"
        
        elif self.__fase == self.FASE_RODILLOS:
            if self.__secado_a_mano:
                self.__fase = self.FASE_SECADO_MANO # Si está en la fase "rodillos" y se ha marcado "secado a mano" pasa a la fase "secado a mano"
            else:
                self.__fase = self.FASE_SECADO_AUTOMATICO # Si está en la fase "rodillos" pero no se ha marcado "secado a mano" pasa a la fase "secado automático"
        
        elif self.__fase == self.FASE_SECADO_AUTOMATICO:
            self.terminar() # Si está en la fase de secado automático se ejectua la función terminar()
        
        elif self.__fase == self.FASE_SECADO_MANO:
            if self.__encerado:
                self.__fase = self.FASE_ENCERADO # Si está en la fase "secado a mano" y marcado "encerado" pasa a la fase "encerado", sino ejecuta la funcion terminar()
            else:
                self.terminar()

        elif self.__fase == self.FASE_ENCERADO:
            self.terminar() # Si está en la fase "encerado" se ejecuta la función terminar()
        
        else:
            raise RuntimeError(f"Estado no válido: Fase {self.__fase}. El lavadero va a estallar...")
            # Si está en cualquier fase que no sea ninguna de las anteriores se lanza un error en tiempo de ejecución

    # Mapea todas las fases posibles en el programa y muestra la fase actual
    def imprimir_fase(self):
        fases_map = {
            self.FASE_INACTIVO: "0 - Inactivo",
            self.FASE_COBRANDO: "1 - Cobrando",
            self.FASE_PRELAVADO_MANO: "2 - Haciendo prelavado a mano",
            self.FASE_ECHANDO_AGUA: "3 - Echándole agua",
            self.FASE_ENJABONANDO: "4 - Enjabonando",
            self.FASE_RODILLOS: "5 - Pasando rodillos",
            self.FASE_SECADO_AUTOMATICO: "6 - Haciendo secado automático",
            self.FASE_SECADO_MANO: "7 - Haciendo secado a mano",
            self.FASE_ENCERADO: "8 - Encerando a mano",
        }
        print(fases_map.get(self.__fase, f"{self.__fase} - En estado no válido"), end="")

    # Muestra el estado de general del lavadero
    def imprimir_estado(self):
        print("----------------------------------------")
        print(f"Ingresos Acumulados: {self.ingresos:.2f} €")
        print(f"Ocupado: {self.ocupado}")
        print(f"Prelavado a mano: {self.prelavado_a_mano}")
        print(f"Secado a mano: {self.secado_a_mano}")
        print(f"Encerado: {self.encerado}")
        print("Fase: ", end="")
        self.imprimir_fase()
        print("\n----------------------------------------")
        
    # Esta función es útil para pruebas unitarias, no es parte del lavadero real
    # nos crea un array con las fases visitadas en un ciclo completo

    def ejecutar_y_obtener_fases(self, prelavado, secado, encerado):
        """Ejecuta un ciclo completo y devuelve la lista de fases visitadas."""
        self.hacerLavado(prelavado, secado, encerado)
        fases_visitadas = [self.fase]

        while self.ocupado:
            # Usamos un límite de pasos para evitar bucles infinitos en caso de error
            if len(fases_visitadas) > 15:
                raise Exception("Bucle infinito detectado en la simulación de fases.")
            self.avanzarFase()
            fases_visitadas.append(self.fase)

        return fases_visitadas

```

## Solución de errores y ejecución

Durante el proceso de documentación y ejecución se detectan `2 fallos`:

**1-. main_app.py (linea 83):** En la llamada a la función `ejecutarSimulacion()` le falta un parametro, el parametro `encerado=false`

```python
    # EJEMPLO 4: Lavado con prelavado a mano (Requisito 4 y 10)
    # Precio esperado: 5.00 + 1.50 = 6.50 €
    print("\n=======================================================")
    print("EJEMPLO 4: Prelavado (S), Secado a mano (N), Encerado (N)")
    ejecutarSimulacion(lavadero_global, prelavado=True, secado_mano=False, encerado=False)
```

**2-. lavadero.py (linea 136-141):** Error en una de las fases tras realizar un lavado con todos los extras a N (False), ya que al avanzar la fase de "pasando rodillos" a "secado automatico", pasa a "secado a mano" cuando este extra no se ha seleccionado. Esto pasa debido a la validacion en la funcion `avanzarFase()`, por lo que se intercambia en las asignaciones `self.FASE_SECADO_MANO` por `self.FASE_SECADO_AUTOMATICO` y viceversa. Este error no solo se encuentra en el **ejemplo 2**, sino que tambien se puede encontrar en el **ejemplo 4**

```python
# Avanza la fase actual segun el transcurso de la ejecución y las opciones seleccionadas
    def avanzarFase(self):

        if not self.__ocupado: # Si esta ocupado no permitira avanzar de faser 
            return

        """
        Si esta inactivo se asigna el coste cobrado, 
        se actualiza a la fase cobrando y se muestra por consola el coste cobrado
        """
        if self.__fase == self.FASE_INACTIVO: 
            coste_cobrado = self._cobrar()
            self.__fase = self.FASE_COBRANDO
            print(f" (COBRADO: {coste_cobrado:.2f} €) ", end="")

        elif self.__fase == self.FASE_COBRANDO:
            if self.__prelavado_a_mano:
                self.__fase = self.FASE_PRELAVADO_MANO # Si está en la fase "cobrando" y se ha marcado "prelavado a mano" pasa a la fase "prelavado a mano" 
            else:
                self.__fase = self.FASE_ECHANDO_AGUA # Si está en la fase "cobrando" pero no se ha marcado "prelavado a mano" pasa a la fase "echando agua"
        
        elif self.__fase == self.FASE_PRELAVADO_MANO:
            self.__fase = self.FASE_ECHANDO_AGUA # Si está en la fase "prelavado a mano" pasa a la fase "echando agua"
        
        elif self.__fase == self.FASE_ECHANDO_AGUA:
            self.__fase = self.FASE_ENJABONANDO # Si está en la fase "echando agua" pasa a la fase "enjabonando"

        elif self.__fase == self.FASE_ENJABONANDO:
            self.__fase = self.FASE_RODILLOS # Si esta en la fase "enjabonando" pasa a la fase "rodillos"
        
        elif self.__fase == self.FASE_RODILLOS:
            if self.__secado_a_mano:
                self.__fase = self.FASE_SECADO_MANO # Si está en la fase "rodillos" y se ha marcado "secado a mano" pasa a la fase "secado a mano"
            else:
                self.__fase = self.FASE_SECADO_AUTOMATICO # Si está en la fase "rodillos" pero no se ha marcado "secado a mano" pasa a la fase "secado automático"
        
        elif self.__fase == self.FASE_SECADO_AUTOMATICO:
            self.terminar() # Si está en la fase de secado automático se ejectua la función terminar()
        
        elif self.__fase == self.FASE_SECADO_MANO:
            if self.__encerado:
                self.__fase = self.FASE_ENCERADO # Si está en la fase "secado a mano" y marcado "encerado" pasa a la fase "encerado", sino ejecuta la funcion terminar()
            else:
                self.terminar()

        elif self.__fase == self.FASE_ENCERADO:
            self.terminar() # Si está en la fase "encerado" se ejecuta la función terminar()
        
        else:
            raise RuntimeError(f"Estado no válido: Fase {self.__fase}. El lavadero va a estallar...")
            # Si está en cualquier fase que no sea ninguna de las anteriores se lanza un error en tiempo de ejecución
```

## Ejecución del programa
Una vez solucionados estos errores en el código, el codigo ya se ejecuta completo y sin errores en la ejecución, pero para asegurarnos que toda la lógica del programa esta funcionando correctamente debemos realizar los **tests unitarios** (su función es poder testear todos los casos posibles del programa), pero estos los realizaremos en el siguiente apartado

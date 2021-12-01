# SED

library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
entity SEMAFORO is
    Port ( sensor,reset,clk : in  STD_LOGIC;
           semcamin,semcarr : out  STD_LOGIC_VECTOR (0 to 2));
end SEMAFORO;
-- el puerto de salida semcamin hace referencia al semaforo del camino o la via rural
-- el puerto de salida semcarr hace referencia al semaforo de la carretera o la via principal
architecture Behavioral of SEMAFORO is

TYPE estado is --declaracion de estados
(S0,S1,espera);
--S0-> Semaforo carretera verde y semaforo rural rojo
--S1->semaforo carretera rojo y semafoto rural verde
--espera -> espera de 30 s (si la frecuencia es de 1s por periodo)en S1
-- asignacion de constantes que representan los colores de los semaforos
CONSTANT verde: STD_LOGIC_VECTOR(0 TO 1):="01";
CONSTANT rojo: STD_LOGIC_VECTOR(0 to 1):="10";
SIGNAL presente: estado:=S0;--estado actual
SIGNAL rescont: boolean:=false;--setea el contador en 0
SIGNAL fin: boolean; -- finaliza el contador
SIGNAL cuenta: integer RANGE 0 to 63;-- contador
begin
-- Definición de la maquina de estados
maquina:
PROCESS(clk,reset)
BEGIN
	IF reset='1' THEN
		presente<=S0;
	ELSIF rising_edge(clk) then --Detecta el flanco ascendente de una señal
		CASE presente IS
			WHEN S0=> --cuando el estado actual sea inicial
				IF sensor='1' THEN -- si el semaforo de camino detecta que hay un auto cambia el estado actual
				presente<=S1;-- estado del semaforo de la carretera en amarillo
				END IF;
			
			WHEN S1 => -- si el estado actual se encuentra en el semaforo del camino en color verde
				IF fin THEN --si el contador termina
					presente<=S0;--el estado actual cambia al S0
		
			END CASE;
		END IF;
END PROCESS maquina;

salida:
-- independiente de las entradas
PROCESS(presente)
BEGIN
	CASE presente IS
		WHEN S0 => -- cuando el estado actual o presente se encuantra en el estado inicial
			semcarr<=verde;-- el semaforo de la carretera se encuentra en color verde
			semcamin<=rojo;-- el semaforo del camino se mantiene en rojo
	
		WHEN S1 =>-- cuando el estado actual se encuantra en el semaforo del camino en verde
			semcarr<=rojo;--el color del semaforo de la carretera pasa a rojo 
			semcamin<=verde;-- claro que el semaforo del camino en verde
			rescont<=false;-- el contador empieza su funcion
		
		when espera => -- cuando el estado actual se encuentra en espera
			semcarr<=rojo;-- el color del semaforo de carretera se encuentra en verde
			semcamin<=verde;-- el color del semaforo de camino en rojo
			rescont<=false;-- empieza el contador a incrementarce
		end case;
END PROCESS salida;

-- este proceso se encarga de definir el contador

contador:
PROCESS(clk)
BEGIN
	IF clk='1' THEN -- cuando sube los ciclos de reloj
		IF rescont THEN cuenta<=0;-- y el rescont esta en true o verdadero se establece la cuenta en 0
		ELSE cuenta<=cuenta+1;-- en cambio si rescont es false cuenta empieza a incrementarce
		END IF;
	END IF;
END PROCESS contador;

-- el tiempo esta definido por los siguiente subidasde reloj
fin<=true WHEN cuenta=29 ELSE false;

end Behavioral;

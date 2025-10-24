Ahora pasamos a la aplicación más compleja, el sumador restador. Al igual que para el caso del sumador de 4 bits, podemos realizar la implementación a partir del módulo sumador de 1 bit, añadiendo un selector que dirá si nos encontramos trabajando con suma o con resta, como se mencionó en el apartado de diseño lógico. En primer  lugar, veremos la descripción estructural del circuito, que se implementó de la  siguiente manera: 

``` verilog

`include "sumador1bit.v"
module sumadorrestador4b (
  input  [3:0] A,    
  input  [3:0] B,     
  input        M,     // Modo: 0 = suma, 1 = resta
  output [3:0] S,     
  output       Cout   
);
  wire [2:0] C;       
  wire [3:0] B_mod;   

  // XOR para invertir B si M=1
  assign B_mod[0] = B[0] ^ M;
  assign B_mod[1] = B[1] ^ M;
  assign B_mod[2] = B[2] ^ M;
  assign B_mod[3] = B[3] ^ M;

  sumador1b FA0 (A[0], B_mod[0], M,    S[0], C[0]);
  sumador1b FA1 (A[1], B_mod[1], C[0], S[1], C[1]);
  sumador1b FA2 (A[2], B_mod[2], C[1], S[2], C[2]);
  sumador1b FA3 (A[3], B_mod[3], C[2], S[3], Cout);
endmodule

```
Como se puede observar, el módulo implementa un sumador-restador de 4 bits reutilizando cuatro full adders de 1 bit. La señal M, según su estado, define si el circuito se encuentra sumando (M=0 ) o restando (M=1). Para el último caso aplica el complemento a 2 de  B, invirtiendo cada bit de B mediante una compuerta XOR con M, y usando a M como el acarreo inicial del primer sumador. Como se ve, los acarreos se propagan entre los módulos, lo que genera las salidas de resultados S y Cout.


Ahora se pasamos a programar el codigo para el test bench con lso ciclos para poder comprobar todos los ciclos:

<p align="center">
<pre>
`timescale 1ns/1ns
`include "sumadorrestador4bit.v"

module tb_sumadorrestador4b;

  reg [3:0] A, B;
  reg M;
  wire [3:0] S;
  wire Cout;
  wire Cin;

  assign Cin = M;

  integer i, j, m;

 
  sumadorrestador4b uut (
    .A(A),
    .B(B),
    .M(M),
    .S(S),
    .Cout(Cout)
  );

  initial begin
    $dumpfile("tb_sumadorrestador4b.vcd");
    $dumpvars(0, tb_sumadorrestador4b);

    for (m = 0; m < 2; m = m + 1) begin
      M = m;
      for (i = 0; i < 16; i = i + 1) begin
        for (j = 0; j < 16; j = j + 1) begin
          A = i;
          B = j;
          #10;
        end
      end
    end

    #20 $finish;
  end

endmodule

</pre>
</p>

Esto nos da los siguientes resultados en la simulación:


<p align="center"> 
  <b>Figura 7.</b> Simulación del sumador–restador de 4 bits estructural <br> <img src="https://github.com/user-attachments/assets/bc907d9b-4269-4ba6-8965-e276e2e11281" alt="Figura 7" width="1000"> 
</p>

Seguimos entonces con la parte comportamental que como vimos no necesita de un modulo aparte, solo que en el codigo quede explicito las operaciones:

<p align="center">
<pre>
module sumadorrestador4bC (
    input  [3:0] A,     
    input  [3:0] B,     
    input        M,     
    output reg [3:0] S, 
    output reg       Cout 
);
    always @(*) begin
        if (M == 0)
            {Cout, S} = A + B;    
        else
            {Cout, S} = A + (~B) + 1'b1; // Modo resta (A - B = A + complemento a2 de B)
    end
endmodule

</pre>
</p>

Y seguimos con su codigo para el test bench:

<p align="center">
<pre>
`timescale 1ns/1ns
`include "sumadorrestador4bitC.v"

module tb_sumadorrestador4bC;

  reg [3:0] A, B;
  reg M;
  wire [3:0] S;
  wire Cout;
  wire Cin;

  assign Cin = M;

  integer i, j, m;

 
  sumadorrestador4bC uut (
    .A(A),
    .B(B),
    .M(M),
    .S(S),
    .Cout(Cout)
  );

  initial begin
    $dumpfile("tb_sumadorrestador4bC.vcd");
    $dumpvars(0, tb_sumadorrestador4bC);

    for (m = 0; m < 2; m = m + 1) begin
      M = m;
      for (i = 0; i < 16; i = i + 1) begin
        for (j = 0; j < 16; j = j + 1) begin
          A = i;
          B = j;
          #10;
        end
      end
    end

    #20 $finish;
  end

endmodule

</pre>
</p>

Y de este obtenemos los siguientes resultados:

<p align="center"> 
  <b>Figura 8.</b> Simulación 3 del sumador–restador de 4 bits comportamental <br> <img src="https://github.com/user-attachments/assets/0956d275-61f4-4d58-81d8-c9b0d57d496f" alt="Figura 8" width="1000"> 
</p>

Igual que antes, ambas simulaciones cumplen y sus resultados son correcto, Por esto también ya estamos preparados para pasar a la implementación de los codigos de descripción de hardware en nuestra fpga.

## Implementación

Durante la etapa de implementación, los módulos diseñados sumador de 1 bit, sumador de 4 bits y sumador–restador de 4 bits— fueron sintetizados y programados en la FPGA Cyclone IV EP4CE6E22C8N utilizando el software Quartus Prime. Cada entrada del circuito (A, B, Cin o M) se asignó a los switches de la placa, mientras que las salidas (S y Cout) se visualizaron mediante los LEDs integrados, permitiendo comprobar el funcionamiento en tiempo real. Se verificó que el comportamiento físico coincidiera con las simulaciones realizadas en Icarus Verilog y GTKWave, lo que confirmó la correcta implementación de la lógica combinacional.

El uso de recursos lógicos fue mínimo, pues los diseños emplean únicamente compuertas básicas y un control adicional mediante XOR en el caso del sumador–restador. Asimismo, no se requirió reloj ni registros, dado que los circuitos fueron descritos completamente mediante lógica combinacional (always @(*)). La FPGA respondió de manera inmediata ante los cambios de las entradas, validando la correcta propagación del acarreo y el funcionamiento del complemento a dos en las operaciones de resta.



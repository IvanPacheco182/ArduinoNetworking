# Sistema de control para acceso de automoviles en red

***
## indice 
+ [descripcion del programa](#descripcion)
+ [funcionalidad](#funcionalidad)
+ [circuito en digital](#circuito)    
+ [codigo en arduino](#codigo)
+ [Evidencia (fotos)](#evidencia)
+ [Contacto](#contacto)
***
## Descripcion   

Debe diseñarse un sistema de control en red basado en la plataforma Arduino. El Escenario, la aplicación, el objetivo y la motivación deben ser seleccionados por los estudiantes.

Cada propuesta debe cumplir los siguientes requisitos:

Se debe incluir al menos tres sensores y un actuador.
Se deben identificar puertos y protocolos de comunicación.
El sistema de control debe estar conectado en red.
El sistema de control debe de mostrar estadísticas de los sensores y tomar acciones en consecuencia.
 

Se consideran componentes adicionales como aplicaciones móviles.


## Material utilizado:

• 1 Placa arduino uno

•	1 módulo Ethernet shield

•	

•	

•	

•	1 sensor de humedad y temperatura

•	1 fotoresistor

• 

• 25 cables para puntear aprox


## funcionalidad


![funcionalidad](/interfaz.JPG)


***
## circuito 
![circuito](/Diagrama.PNG)
***
## codigo en arduino
~~~
#include <SPI.h>
#include <Ethernet.h> 
#include <Servo.h> 
#include <DHT.h>

#define DHTPIN 2           // Definimos el pin digital donde se conecta el sensor
#define DHTTYPE DHT11      // Dependiendo del tipo de sensor
#define LED 8              // El LED esta conectado en el pin 8 
#define LDR 0              // El LDR esta conectador en el pin A0
 

DHT dht(DHTPIN, DHTTYPE);  // Inicializamos el sensor DHT11

Servo microservo; 

int pos = 0; 
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };   // Asignamos una dirección mac al módulo Ethernet Shield
byte ip[] = {192,168,1,68};                     // Ponemos una ip de la red lan que no esté siendo usada (Es con la que accederemos en el buscador)
//byte gateway[] = { 192, 168, 100, 1 };                 // Acceso a internet vía router
//byte subnet[] = { 255, 255, 255, 0 };                  // Mascara de sub red
EthernetServer server(80);                             // puerto del servidor

int luz = 0;       
int valor_sensor = 0;               
int valor_limite = 490;              // Este valor hara que el LED cambie de estado a una determinada luminosidad 
String readString;

void setup() {
 
  Serial.begin(9600);               //Abra la comunicación serial.
  
  pinMode(LED,OUTPUT);
  pinMode(LDR,INPUT);
  microservo.attach(7);
  Ethernet.begin(mac, ip);    // Iniciar la conexión Ethernet y el servidor:
  server.begin();
  Serial.print("server is at ");
  Serial.println(Ethernet.localIP());
  dht.begin();
  
}


void loop() {

  float h = dht.readHumidity();               // Leemos la humedad relativa
  float t = dht.readTemperature();            // Leemos la temperatura en grados centígrados (por defecto)
  valor_sensor = analogRead(LDR);             // Leemos la intensidad de luz
  luz = (5.0 * valor_sensor * 100.0)/1024.0;  // Formula para darle presición a la intensidad de luz
 // Serial.print(luz);  
 //  Serial.println(" Luz");             
  delay(300);                       
  
  if (luz <= valor_limite)        //Si el valor de luz es menor o igual que el valor limite
  {
    digitalWrite (LED, LOW);      //El led se apaga
  }
  if (luz > valor_limite)         //Si es mayor que el valor limite
  {
    digitalWrite (LED, HIGH);     //El led se enciende
  }
    
  /* Creación de una conexión cliente */
  EthernetClient client = server.available(); // 
  if (client) {
    while (client.connected()) {   
      if (client.available()) {
        char c = client.read();
     
        //Lee  read char by char HTTP request
        if (readString.length() < 100) {
          //store characters to string
          readString += c;
          //Serial.print(c);
         }

         //if HTTP request has ended
         if (c == '\n') {          
           Serial.println(readString); //print to serial monitor for debuging       
           client.println("HTTP/1.1 200 OK"); //send new page
           client.println("Content-Type: text/html");
           client.println();     
           client.println("<HTML>");
           client.println("<HEAD>");
           client.println("<meta name='apple-mobile-web-app-capable' content='yes' />");
           client.println("<meta name='apple-mobile-web-app-status-bar-style' content='black-translucent' />");
           client.println("<link rel=stylesheet type='text/css' href='/home/giz/Escritorio/ethernetshield/ethernetcss.css'>");
           client.println("<TITLE>Sistema para acceso a la vivienda</TITLE>");
           client.println("</HEAD>");
           client.println("<BODY>");
           client.println("<background = '0000'>");
           client.println("<H1>Sistema para acceso a la vivienda</H1>");
           client.println("<hr />");
           client.println("<br />");  
           client.println("<H2>Buenos dias: Usuario</H2>");
           client.println("<br />");
           client.print("<H3>La temperatura en Grados centigrados es de: </H3>");
           client.print(t, 4);
           client.println("<H3>Con una humedad (%) de: </H3>");
           client.println(h, 4);
           client.println("<br />");
           client.println("<br />");   
    //     client.println("<a href=\"/?button1on\"\">Prender led</a>");
    //     client.println("<a href=\"/?button1off\"\">Apagar led</a><br />");   
           client.println("<br />");     
           client.println("<a href=\"/?button2on\"\">Abrir Puerta</a>");
           client.println("<a href=\"/?button2off\"\">Cerrar puerta</a><br />"); 
           client.println("<br />"); 
           client.println("</BODY>");
           client.println("</HTML>");
     
           delay(1);
           //stopping client
           client.stop();

           if (readString.indexOf("?button2on") >0){  //Controla el arduino si tu presionas el botón
                for(pos = 0; pos < 90; pos += 3)      // Ciclo para mover de 0 a 90 grados el microservo 
                {                                     // Se guarda en la variable 'pos'
                  microservo.write(pos);              // Le dice al microservo que tome la posición de 'pos' 
                  delay(20);                          // Espera 15ms para que el servo alcance la posición
                } 
           }
           if (readString.indexOf("?button2off") >0){
                for(pos = 90; pos>=1; pos-=3)         // Ciclo para mover de 90 a 0 grados el microservo 
                {                                
                  microservo.write(pos);               // Le dice al microservo que tome la posición de 'pos' 
                  delay(20);                           // Espera 15ms para que el servo alcance la posición
                } 
           }
            
                 readString="";                       //Limpia el string para la próxima lectura
           
         }
       }
    }
}
}

~~~

## Evidencia

![evidencia](Evidencia/img1.jpg)
![evidencia](Evidencia/img2.jpg)
![evidencia](Evidencia/img3.jpg)
![evidencia](Evidencia/img4.jpg)
![evidencia](Evidencia/img5.jpg)


## Contacto
~~~

Elaborado por: 
Oscar Iván Pacheco Vargas
heavy.pacheco@gmail.com

Carlos Leonardo Luna Castillo
carl_dharius@hotmail.com

Erick Alejandro Ochoa González
Erick.8a@gmail.com


~~~

## Derechos de Autor 
El material mostrado para la elaboración del circuito digital 
planteado es libre para su uso y modificación futura por cualquier
colaborador



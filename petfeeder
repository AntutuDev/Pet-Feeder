#include <SoftwareSerial.h>
#include <LiquidCrystal_I2C.h>
#include <SPI.h> //MISO/MOSI/SCK/SDA
#include <MFRC522.h> //RFID
#include <Servo.h>
#include <Wire.h>
#include <HX711_ADC.h>
#include <RTClib.h>

//Datos Mascota
  char* nombre;
  byte tag[4]={0x00, 0x00, 0x00, 0x00};

//Datos raciones
  byte hora1;
  byte minuto1;
  byte salteada1; //true si salteada 4 veces
  byte postergada1; //vuelve a cero si llega a 4

  byte hora2;
  byte minuto2;
  byte salteada2; //true si salteada 4 veces
  byte postergada2; //vuelve a cero si llega a 4

  byte hora3;
  byte minuto3;
  byte salteada3; //true si salteada 4 veces
  byte postergada3; //vuelve a cero si llega a 4

  byte hora4;
  byte minuto4;
  byte salteada4; //true si salteada 4 veces
  byte postergada4; //vuelve a cero si llega a 4

//Para las estadisticas
int postergadasSemana; //cant que vuelve a 0 el domingo
int postergadasMes; //cant que vuelve a 0 el día 1 a las 00hs
int porcionesSemana; //gramos totales servidos en la semana vuelve a 0 el domingo
int porcionesMes; //gramos totales servidos en el mes vuelve a 0 el dia 1 a las 00hs


//RFID pines
#define RST 49
#define MISO 50
#define MOSI 51
#define SCK 52
#define SDA 53

//Botones pines
#define botonLuz 26
#define botonTapa 27

//Balanza pines
#define DT 34
#define SCK 35

//Ultrasonico pines
#define trigPin 32
#define echoPin 33

//LED bicolor pines
#define ledRojo 36
#define ledVerde 37

MFRC522 lectorRFID(SDA,RST);
Servo servoTolva;
Servo servoTapa;
LiquidCrystal_I2C lcd(0x27, 16, 2);
HX711_ADC balanza(DT,SCK);
SoftwareSerial BTSerial(19,18);
RTC_DS3231 rtc;

boolean tapaCerrada=true;
boolean luz=true;
boolean stopTotal=false;
float valorCal=725;
long duracion;
int distancia;
int porcion;
String datos; //concatenación de los chars que se leen por serial
byte inicializacion=0; //no hay datos en EEPROM

boolean inicializacion(){
  lcd.setCursor(3,1);
  lcd.print("Bienvenido");
  delay(3000);
  lcd.setCursor(0,1);
  lcd.print("Inicializando...");
  delay(3000);
  if(inicializacion==1)
  return true;
  else
  return false;
  //buscar datos de la eeprom
  //cargarlos y mostrar true si habia datos o false si no habia nada
}

void servirRacion(){
  //Comparar hora actual con hora seteada hh mm y ss
  //Tapa cerrada = true
  //que stop total este en false
  // acciones en orden:
  //valida tapa, valida tolva, muestra en lcd, abre tapa, 
  //activa servo tolva, while datoBalanza<porcion, cierra tapa tolva,
  //cierra tapa plato, marca racion servida, lcd muestra hora de nuevo  
  //
}

int medirDistancia(){
  digitalWrite(trigPin,LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin,HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin,LOW);

  duracion=pulseIn(echoPin,HIGH);
  distancia=duracion*0.034/2;

  return distancia;
}

float datoBalanza(){
  balanza.update();
  return (balanza.getData());
}


void mostrarFechaHora(){
  DateTime now = rtc.now();
  lcd.setCursor(0,1);
  
  if(now.hour()<10){
    lcd.print("0");
    lcd.print(now.hour());
    }
  else{
    lcd.print(now.hour());
    }
    
  lcd.print(":");
  
  if(now.minute()<10){
    lcd.print("0");
    lcd.print(now.minute());
    }
  else{
    lcd.print(now.minute());
    }
  lcd.print(":");
  
  if(now.second()<10){
  lcd.print("0");
  lcd.print(now.second());
}
else{
  lcd.print(now.second());
}
lcd.setCursor(11,1);
lcd.print(now.day());
lcd.print("/");
lcd.print(now.month());

}

void agregarMascota(){
  nombre="agregar nombre";

 // Revisamos si hay nuevas tarjetas  presentes
 while(tag[0]==0x00){if ( lectorRFID.PICC_IsNewCardPresent()) 
        {  
      //Seleccionamos una tarjeta
            if ( lectorRFID.PICC_ReadCardSerial()) 
            {
                  // Enviamos serialemente su UID
                  Serial.print("Card UID:");
                  for (byte i = 0; i < lectorRFID.uid.size; i++) {
                          Serial.print(lectorRFID.uid.uidByte[i] < 0x10 ? " 0" : " ");
                          Serial.print(lectorRFID.uid.uidByte[i], HEX);
                          tag[i]=lectorRFID.uid.uidByte[i];   
                  } 
                  Serial.println();
                  // Terminamos la lectura de la tarjeta  actual
                  lectorRFID.PICC_HaltA();         
            }      
  }
}

}

void eliminarMascota(){
  //poner todo en cero
}


bool comparaTag(byte* tagA,byte* tagB){
  for(int i=0; i<4; i++)
  {
    if(tagA[i]!=tagB[i])
    return false;
  }
  return true;
}

void setup() {
  Wire.begin();
  pinMode(botonTapa,INPUT_PULLUP);
  pinMode(botonLuz,INPUT_PULLUP);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledRojo, OUTPUT);
  pinMode(ledVerde, OUTPUT);
  balanza.begin();
  balanza.start(2000);
  balanza.setCalFactor(valorCal);
  lcd.begin(16,2);
  lcd.backlight();
  //rtc.adjust(DateTime(2019,7,16,10,40,0)); año,mes,dia,hh,mm,ss
  BTSerial.begin(9600);
  SPI.begin();
  lectorRFID.PCD_Init();
  servoTolva.attach(8);
  servoTolva.write(90); //tolva cerrada
  servoTapa.attach(9);
  servoTapa.write(76.5); //el punto cero del servo modificado
  lcd.setCursor(2,0);
  lcd.print("SMART FEEDER");
  boolean init = inicializacion();
  if(init==false)
  {
    lcd.setCursor(2,1);
    lcd.print("No hay datos");
    delay(3000);
  }
  else
  {
    lcd.setCursor(0,1);
    lcd.print("Cargando datos..");
    delay(3000);
  }
}

void loop() {
  //Mostrar fecha y hora
  mostrarFechaHora();

  if(medirDistancia()<=20)
  {
    digitalWrite(ledRojo,LOW);
    digitalWrite(ledVerde,HIGH);
  }
  else
  {
    digitalWrite(ledVerde,LOW);
    digitalWrite(ledRojo,HIGH);
  }
      
  //Si se presiona el boton de la tapa
    if(digitalRead(botonTapa)==LOW){
    if(tapaCerrada==true){
        
        servoTapa.write(96); //giro der mov izq
        delay(2000);
        servoTapa.write(76.5);
      }
    
    else{
        servoTapa.write(56); //giro izq mov der
        delay(2000);
        servoTapa.write(76.5);
      }
    tapaCerrada=!tapaCerrada;
  while(digitalRead(botonTapa)==LOW);
  }

  //Si se presiona el boton de backlight
  
  if(digitalRead(botonLuz)==LOW){
    if(luz==false){
      lcd.backlight();
    }
    else{
      lcd.noBacklight();      
    }
    luz=!luz;
    while(digitalRead(botonLuz)==LOW);//evita los rebotes
  }
  
  
  
  
  
  if(BTSerial.available()>0)
  {
    char opcion=BTSerial.read();
    switch(opcion){
      case 'A':{
        agregarMascota();
      }
      break;
      case 'B':{
        //eliminarMascota();
      }
      break;
      case 'I':{
        delay(30);
        while(BTSerial.available()){
          char c=BTSerial.read();
          datos+=c;
        }
        if(datos.length()>0)
        {
          porcion=datos.toInt();
          datos="";
        }
      }
      break;
    }
  }
  
if ( lectorRFID.PICC_IsNewCardPresent()) 
        {  
            if ( lectorRFID.PICC_ReadCardSerial())//selecciona tag 
                 if(comparaTag(lectorRFID.uid.uidByte,tag))//lee tag y lo compara
                 {
                  lcd.setCursor(2,1);
                  lcd.print(nombre);
                 }
                 else
                 {
                  lcd.setCursor(1,1);
                  lcd.print("ALERTA INTRUSO");
                 }
                  lectorRFID.PICC_HaltA();//stop lectura         
            }      
  }
}

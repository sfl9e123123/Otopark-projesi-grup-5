#include <Servo.h>
#include <SPI.h>
#include <MFRC522.h>
#include <EEPROM.h>

#define RST_PIN 9
#define SS_PIN 10

MFRC522 mfrc522(SS_PIN, RST_PIN);

String lastRfid = "";
String kart1 = "";
String kart2 = "";

MFRC522::MIFARE_Key key;
//********************************buzzer******************************//
int buzzerPin = 8 ;
int notaSayisi = 8;
int C = 262;
int D = 294;
int E = 330;
int F = 349;
int G = 392;
int A = 440;
int B = 494;
int C_ = 523;
int notalar[] = {C, D, E, F, G, A, B, C_};

//********************************servo*******************************//
Servo sg90;

//********************************mesafe****************************//
#define trigPin 4   //Sensörün Echo pini Arduinonun 4. pinine bağlanır
#define echoPin 5   //Sensorün Trig pini Arduinonun 5. pinine bağlanır


void setup()
{
  Serial.begin(9600);
  SPI.begin();
  mfrc522.PCD_Init();
  pinMode(buzzerPin, OUTPUT);
  Serial.println("RFID KART OKUMA UYGULAMASI");
  Serial.println("--------------------------");
  Serial.println();
  //EEPROM'dan kart bilgisini oku
  readEEPROM();
{
  sg90.attach(6);
}
{
  pinMode(trigPin, OUTPUT); //4. yani trigpini çıkış olarak ayarlıyoruz
  pinMode(echoPin, INPUT); //5. yani echoPini giriş olarak ayarlıyoruz
  Serial.begin(9600);
  pinMode(3, OUTPUT);
  pinMode(2 , OUTPUT);

}
}

void loop()
{
  //yeni kart okununmadıkça devam etme
  if ( ! mfrc522.PICC_IsNewCardPresent())
  {
    return;
  }
  if ( ! mfrc522.PICC_ReadCardSerial())
  {
    return;
  }
  //kartın UID'sini oku, rfid isimli string'e kaydet
  String rfid = "";
  for (byte i = 0; i < mfrc522.uid.size; i++)
  {
    rfid += mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ";
    rfid += String(mfrc522.uid.uidByte[i], HEX);
  }
  //string'in boyutunu ayarla ve tamamını büyük harfe çevir
  rfid.trim();
  rfid.toUpperCase();
  
  if (rfid == lastRfid)
    return;
  lastRfid = rfid;

  Serial.print("Kart 1: ");
  Serial.println(kart1);
  Serial.print("Kart 2: ");
  Serial.println(kart2);
  Serial.print("Okunan: ");
  Serial.println(rfid);
  Serial.println();
  //1 nolu kart okunduysa LED'i yak, 2 nolu kart okunduysa LED'i söndür
  if (rfid == kart1)
  {
    digitalWrite(buzzerPin, HIGH);
    delay(100);
    digitalWrite(buzzerPin, LOW);
    Serial.println("Giris Basarılı!");
  }
  if (rfid == kart1)
  {
    sg90.write(85);
    delay(2000);
    sg90.write(5);
    Serial.println("Kapı açıldı.");
  }
   if (rfid == kart1)
  {
    long duration, distance;
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(1000);
  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  distance = (duration / 2) / 29.1; //Ölçüm fonksiyonu
  Serial.println(distance);

  if (distance <= 5)
  {
    digitalWrite(3, HIGH);
    digitalWrite(2, LOW);
    delay(3000);
    digitalWrite(3, LOW);
    digitalWrite(2, LOW);
  }
  if (distance > 5 && distance < 200)
  {
    digitalWrite(2, HIGH);
    digitalWrite(3, LOW);
    delay(3000);
    digitalWrite(2, LOW);
    digitalWrite(3, LOW);
  }
  else
  {
    digitalWrite(22, LOW);
    digitalWrite(23, LOW);
  }
  }
  if (rfid == kart2)
  {
    Serial.println("Gecersiz Kart!");
    digitalWrite(buzzerPin, HIGH);
    delay(500);
    digitalWrite(buzzerPin, LOW);
    
  }
  Serial.println();
  delay(200);

}

void readEEPROM()
{
  //EEPROM'dan ilk kartın UID'sini oku (ilk 4 byte)
  for (int i = 0 ; i < 4 ; i++)
  {
    kart1 += EEPROM.read(i) < 0x10 ? " 0" : " ";
    kart1 += String(EEPROM.read(i), HEX);
  }
  //EEPROM'dan ikinci kartın UID'sini oku
  for (int i = 4 ; i < 8 ; i++)
  {
    kart2 += EEPROM.read(i) < 0x10 ? " 0" : " ";
    kart2 += String(EEPROM.read(i), HEX);
  }
  //Okunan stringleri düzenle
  kart1.trim();
  kart1.toUpperCase();
  kart2.trim();
  kart2.toUpperCase();
}

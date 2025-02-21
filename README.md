#include <SPI.h>
#include <MFRC522.h>
#include <Ethernet.h>

#define RST_PIN 7
#define SS_PIN 10

MFRC522 mfrc522(SS_PIN, RST_PIN); 

byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
IPAddress ip(192, 168, 1, 177);
char server[] = "192.168.1.100"; 
EthernetClient client;

void setup() {
  Serial.begin(9600);
  SPI.begin();
  mfrc522.PCD_Init();
  Ethernet.begin(mac, ip);
  Serial.println("RFID Scanner Ready");
}

void loop() {
  if (!mfrc522.PICC_IsNewCardPresent() || !mfrc522.PICC_ReadCardSerial()) {
    return;
  }

  String uid = "";
  for (byte i = 0; i < mfrc522.uid.size; i++) {
    uid += String(mfrc522.uid.uidByte[i], HEX);
  }
  Serial.println("Card UID: " + uid);

  sendUidToServer(uid);

  mfrc522.PICC_HaltA();
  mfrc522.PCD_StopCrypto1();
}

void sendUidToServer(String uid) {
  if (client.connect(server, 5000)) {
    Serial.println("Connected to server");
  
    client.println("POST /get-user-info HTTP/1.1");
    client.println("Host: 192.168.1.100");
    client.println("Content-Type: application/x-www-form-urlencoded");
    client.print("Content-Length: ");
    client.println(uid.length());
    client.println();
    client.print("uid=");
    client.print(uid);
    client.println();
    Serial.println("UID sent to server");
  } else {
    Serial.println("Connection failed");
  }
  client.stop();
}


                

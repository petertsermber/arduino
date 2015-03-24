/**
* Универсальный контроллер сбора данных и исполительное устройство на базе Arduino (к проекту http://smartliving.ru/)
* Platform: Arduino UNO R3 + EthernetShield W5100
* IDE: Arduino 1.6.1
* Author: Peter Tsember by Syktyvkar. 2015
**/

// ----==== ПОДКЛЮЧЕНИЕ БИБЛИОТЕК ====----
  #include <Ethernet.h>                                // (Ethernet Shield)
  #include <SPI.h>                                     // (Для работы с устройствами по протоколу SPI, например с Ethernet Shield)
  #include <Arduino.h>                                 // (Основная библиотека Arduino)
  #include <WebServer.h>                               // (Webserver WebDuino)
  #include <OneWire.h>                                 // (Шина 1-wire)
  #include <DallasTemperature.h>                       // (Датчик температуры DS18B20)


// ----==== НАСТРОЙКИ СЕТИ ====----
  byte mac[] = { 0xDE, 0xAD, 0xBE, 0xE4, 0xDE, 0x35 }; // MAC-адрес контроллера Arduino
  byte ip[] = { 192, 168, 1, 15 };                     // ip-адрес контроллера Arduino
  byte subnet[] = { 255, 255, 255, 0 };                // ip-адрес сервера MajorDoMo
  byte gateway[] = { 192, 168, 1, 250 };               // ip-адрес основного шлюза
  byte dns_server[] = { 192, 168, 1, 250 };            // ip-адрес DNS-сервера
  byte rserver[] = { 192, 168, 1, 123 };               // ip-адрес сервера MajorDoMo
  char buf[100];                                       // Переменная для хранения GET запроса
  char ipbuff[16];                                     // Переменная для IP адреса
  EthernetClient rclient;


// ----==== НАСТРОЙКИ ПОДКЛЮЧЕННЫХ УСТРОЙСТВ ====----
// Вход, на котором находится шина 1-wire с датчиками DS18S20
  #define ONE_WIRE_BUS 2                               // нога 2



// ----==== НАСТРОЙКИ ПЕРЕМЕННЫХ И МАССИВОВ ДАННЫХ ====----
  float current_temp1;                                 // Задаем переменную, в которой будем хранить текущую температуру
  float delta_temp = 0.3;                              // Дельта температур, при превышении которой следует отправлять данные на сервер МЖД
  long requestMillis;                                  // Задаем переменную времени последней передачи данных на сервер
  long interval = 18000000;                            // Задаем максимальный интервал между передачами данных на сервер
  byte numberOfDevices;                                // Количество датчиков DS18B20 на шине 1-wire (Обновляется автоматически)
  DeviceAddress Termometers;                           // Массив с датчиками
  // Размер следующих массивов - предполагаемое максимальное количество датчиков DS18B20.
  // По умолчанию - 30. Нужно больше?! Увеличиваете числа и перезаливаете скетч
  byte na[30];                                         // Массив с порядковыми номерами датчиков температуры DS18B20
  String aa[30];                                       // Массив с HEX-адресами датчиков температуры DS18B20
  float ta[30];                                        // Массив с последней отправленной температурой датчиков DS18B20




// Инициализация веб-сервера на 80 порту, а так же шины 1-wire и датчиков температуры DS18B20
  WebServer webserver("", 80);
  OneWire oneWire(ONE_WIRE_BUS);
  DallasTemperature sensors(&oneWire);




/*
// Вывод информации об устройстве (http://ip-arduino/)
void infoRequest(WebServer &server, WebServer::ConnectionType type, char *url_tail, bool tail_complete)
{
  int numberOfDevices = sensors.getDeviceCount();
   sensors.begin();
   sensors.requestTemperatures();
   for(int i=0;i<numberOfDevices; i++) {
      if(sensors.getAddress(Termometers, i))
      {
          server.print("T");
          server.print(i);
          server.print(":");
          for (uint8_t i = 0; i < 8; i++) {
            if (Termometers[i] < 16) server.print("0");
              server.print(Termometers[i], HEX);
          }
          float tempC = sensors.getTempC(Termometers);
          server.print(":");
          server.print(tempC);
          server.print(";");
      } else {
            // not found
            server.print("NOT FOUND");
      }
    }
}
*/


// ---=== ФУНКЦИЯ ОТПРАВКИ ДАННЫХ (HTTP-ЗАПРОСА) НА СЕРВЕР MAJORDOMO ===---
void sendHTTPRequest() {
  if (rclient.connect(rserver, 80)) { 
   Serial.println("OK"); 
   Serial.println(buf);                                 // Вывод на экран сгенерированного GET запроса
   rclient.print(buf);
   rclient.println(" HTTP/1.0");
   rclient.print("Host: ");
   sprintf(ipbuff, "%u.%u.%u.%u", ip[0], ip[1], ip[2], ip[3]);
   rclient.println(ipbuff);                             // ip адрес нашего контроллера в текстовом виде
   rclient.print("Content-Type: text/html\n");
   rclient.println("Connection: close\n");
   delay(2000);
   rclient.stop();
  } else 
  {
   Serial.println("FAILED SENDING");     
  }
  
}


// ---=== ФУНКЦИЯ НАСТРОЙКИ ===---
void setup()
  {
  Serial.begin(9600);                                   // Инициализируем Serial (монитор последовательного порта)
  Serial.println("Start");
  Ethernet.begin(mac, ip, dns_server, gateway, subnet); // Инициализируем Ethernet Shield/
//webserver.setDefaultCommand(&infoRequest);            // дефолтная страница вывода (информация о контроллере)
  webserver.begin();                                    // Инициализируем Web-Server
  char buff[64];
  int len = 3;                                          // Количество попыток запуска сервера
webserver.processConnection(buff, &len);                // process incoming connections one at a time forever
  Serial.print("server is at ");
  Serial.println(Ethernet.localIP());
  }


// ---=== ЦИКЛИЧЕСКАЯ ФУНКЦИЯ ===---
void loop() {

  
  
  
// Поиск датчиков температуры DS18B20 на шине 1-wire
  sensors.begin();                                   // Инициализация датчиков DS18B20
  sensors.requestTemperatures();                     // Перед каждым получением температуры надо ее запросить
  sensors.setResolution(Termometers, 12);            // Установка чувствительности на 12 бит 
   Serial.print("Vsego datchikov: ");
   numberOfDevices = sensors.getDeviceCount(); 
   Serial.println(numberOfDevices);                  // вывод на экран общего количества найденых датчиков
   
   for(byte i=0;i<numberOfDevices; i++) {            // цикл для записи в массивы данных
     if(sensors.getAddress(Termometers, i))          // если адрес датчика под индексом i определен, то:
      {        
      String aba[8];                                 // Создаем массив с байтами адреса 
      for (int i=0; i<8; i++)                        // цикл получения бит адреса
        { 
          if (Termometers[i] < 16) { aba[i] = "0"+String(Termometers[i],HEX); } //если бит меньше 16, то дописываем в начале 0, так же конвертируем в шеснадцатиричный код
          else {aba[i] = String(Termometers[i],HEX);}// в противном случае просто конвертируем в 16ричный код и записываем байт в массив
        }
      aa[i] = aba[0]+aba[1]+aba[2]+aba[3]+aba[4]+aba[5]+aba[6]+aba[7]; // Записываем полный адрес в массив адресов aa[], в ячейку i.      
    Serial.print("Sensor "); Serial.print(i, DEC); Serial.print(" with address: "); // Датчик под номером i с адресом:
    Serial.println(aa[i]);                           // Выводим на экран i ячейку с 16-ричным адресом устройства  
       }
     }

 
// Получение и обработка температуры
for(byte i=0;i<numberOfDevices; i++)                 // цикл для получения, обработки, округления и отправки температуры каждого датчика DS18B20
    { 
      // Устройство отдало реальное значение температуры (или осталось старое)
      // Округление полученной температуры до десятых
      // Пример: 23.899 => 238.99 => 239.49 => 239 => 23.9) // При округлении изменился первый разряд, после запятой. Правило работает.
       float tempC = sensors.getTempCByIndex(i) * 10;
       tempC = tempC + 0.5;
       tempC = (int)tempC;
       tempC =  tempC / 10;
    float delta = fabsf(ta[i]-tempC);                // Выделяем дельту температуры, между текущей и в последний раз отправленной темп.
    if ((delta > delta_temp)                         // Если разница между прошлым и текущим значением темп. больше обозначенной дельты...
    || ((millis() - requestMillis) > interval))      // Если последняя передача данных на сервер была более 30 минут назад (например температура не менялась)...
     {  
     int temp1 = (tempC - (int)tempC) * 10;          // выделяем дробную часть
     int str_len = aa[i].length() + 1; char char_array[str_len]; aa[i].toCharArray(char_array, str_len);      // Конвертируем строку с адресом в массив символов
     sprintf(buf, "GET /objects/?object=%s&op=m&m=tempChanged&t=%0d.%d", char_array, (int)tempC, abs(temp1)); // Формируем GET запрос
      sendHTTPRequest();                             // И передаем на сервер
      requestMillis = millis();                      // Засекаем время последней передачи данных на сервер
      ta[i]=tempC;                                   // Задаем текущую температуру предыщущей.
     }
    }
}

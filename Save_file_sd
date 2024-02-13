#include <WiFi.h>
#include <AsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include <SPIFFS.h>
#include <FS.h>
#include "SD_MMC.h"
#include <SD.h>

const char* ssid = "Galaxy Note10";
const char* password = "22223333";

WiFiServer server(80);

// HTML web page to handle file upload
const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>ESP32 File Upload</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f0f0f0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }

    .upload-container {
      background-color: #ffffff;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }

    .upload-container h2 {
      margin-top: 0;
      text-align: center;
    }

    .upload-form {
      text-align: center;
    }

    .upload-button {
      background-color: #4CAF50;
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s;
    }

    .upload-button:hover {
      background-color: #45a049;
    }

    .progress-bar {
      width: 100%;
      background-color: #ddd;
      border-radius: 5px;
      margin-top: 10px;
      display: none; /* Початково ховаємо шкалу прогресу */
    }

    .progress-value {
      width: 0%;
      height: 20px;
      background-color: #4CAF50;
      border-radius: 5px;
      text-align: center;
      line-height: 20px;
      color: white;
    }
  </style>
</head>
<body>
  <div class="upload-container">
    <h2>Завантажити файл на SD</h2>
    <form class="upload-form" action="/upload" method="post" enctype="multipart/form-data" >
      <input type="file" name="fileToUpload" id="fileToUpload">
      <br><br>
      <div class="progress-bar" id="progress-bar">
        <div class="progress-value" id="progress-value">0%</div>
      </div>
      <br><br>
      <input type="submit" class="upload-button" value="Завантажити" name="submit" onclick="showUploadMessage()">
    </form>
    <p id="upload-message"></p>
  </div>

<script>
    function showUploadMessage() {
      document.getElementById("upload-message").innerText = 'Завантаження...';
    }
  </script>
</body>
</html>

)rawliteral";
//*******************************************************************

const char ok_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>ESP32 File Upload</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f0f0f0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }

    .upload-container {
      background-color: #ffffff;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
      text-align: center;
    }

    .upload-success {
      color: #4CAF50;
      font-size: 48px;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>
  <div class="upload-container">
    <div class="upload-success">&#10004;</div>
    <p>Файл завантажено успішно!</p>
  </div>

  <script>
    setTimeout(function() {
      window.location.href = '/';
    }, 2000);
  </script>
</body>
</html>

)rawliteral";
//****************************************************************************************
const char error_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>ESP32 File Upload</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f0f0f0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }
    .error-message {
      color: red;
    }
  </style>
</head>
<body>
  <h2 class="error-message">Upload Failed</h2>
  <script>
    // Очистити сторінку
    document.body.innerHTML = '';

    // Показати повідомлення про невдачу завантаження
    var errorMessage = document.createElement('h2');
    errorMessage.classList.add('error-message');
    errorMessage.innerText = 'Upload Failed';
    document.body.appendChild(errorMessage);

    // Перенаправлення на головну сторінку через дві секунди
    setTimeout(function() {
      window.location.href = '/';
    }, 2000);
  </script>
</body>
</html>

)rawliteral";
//****************************************************************************************
void setup() {
  Serial.begin(115200);
  if (!SPIFFS.begin(true)) {
    Serial.println("An error occurred while mounting SPIFFS");
    return;
  }

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
  Serial.println("******************************");
  Serial.print("Visit this page at: http://");
  Serial.println(WiFi.localIP());
  Serial.println("******************************");

  
if(!SD_MMC.begin()) {
    Serial.println("Card Mount Failed");
    return;
  }
  Serial.println("Card Initialized");

  server.begin();
}
int k=0;
long int line=0;
void loop() {
  String nameFile;
  WiFiClient client = server.available();  

  if (client) {                             
    Serial.println("New Client.");        
    String currentLine = "";   
    k=0;     
    while (client.connected()) {       
      delay(10); 
      if (client.available()) {                      
        String currentLine=client.readStringUntil('\r');
        //Serial.print(currentLine); 
        if (currentLine.startsWith("GET / ") or currentLine.startsWith("GET /favicon.ico")) {
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();
            client.print(index_html);
            client.println();
            break;
        } else 
        if (currentLine.startsWith("POST /upload")) {
          String st=client.readStringUntil('\r');
          Serial.print(st);
          while(st.indexOf("Content-Length: ")==-1){st=client.readStringUntil('\r');  }
         /* int i=17;
          while(st[i]!='\n'){
            Serial.print(st[i]);
            Serial.print("  ");
            line=line*10+int(st[i])-48;
            i++;         
          }
          Serial.println(line);
*/
          
          while(st.indexOf("filename=\"")==-1){st=client.readStringUntil('\r'); }
          int k=0;
          while (char(st[k+64])!='\"') {
              nameFile += st[k+64];
              k++;
            }
            Serial.println(nameFile);
            nameFile="/"+nameFile;  
            File file = SD_MMC.open(nameFile, FILE_WRITE);
char             box=client.read();
            while(box!='\n'){   
              box=client.read();
            }
  
            box=client.read();
            while(box!='\n'){   
              box=client.read();
            }
            
            for(int i=0; i<2; i++)
            box=client.read();
            //long int line2=line;
             st=client.readStringUntil('\r');
            while(st.indexOf("WebKitFormBoundary")==-1){ 
              //Serial.print(st);
              file.print(st);  
              file.write(0x0d);
              st=client.readStringUntil('\r');
              /*int length_=st.length();
              line2=line2-length_;
              float procent=(line2*100)/line;
             client.println("HTTP/1.1 200 OK");
              client.println("Content-type:text/plain");
              client.println();
              client.println(100-procent);
              Serial.println(procent);*/
              
            }
            if(st.indexOf("WebKitFormBoundary"))
            {
              while(client.available()) client.read();
              client.println("HTTP/1.1 200 OK");
              client.println("Content-type:text/html");
              client.println();
              
  
              File file = SD_MMC.open(String("/")+line, FILE_WRITE);
              if (file) {
                file.close();
                Serial.println("Write Successful");
                client.print(ok_html);
                
              } else {
                Serial.println("Write Failed");
                client.print(error_html);
                
                
              }
              client.println();
              break;
              Serial.println("*********end*********");
  
            }
          }
         
      
    }
    }
    client.stop();
    Serial.println("Client Disconnected.");
  }
}

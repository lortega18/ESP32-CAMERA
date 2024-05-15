# ESP32-CAMERA

**Name of Project**

ESP32-CAM DIY Smart Doorbell

**Purpose**

Developing Smart doorbell with ESP32-Camera

**Equipment**

ESP32Cam
Camera Lens
Button/Switch
SD card
LED
Breadboard/ Jumpwires
USB-C Cable
Mobile Device

**Link to Documentation Followed**

* https://circuitdigest.com/microcontroller-projects/diy-smart-wifi-video-doorbell-using-esp32-and-camera
* https://circuitdigest.com/microcontroller-projects/how-to-use-esp32-camera-module-for-video-streaming-and-face-recognition
* https://ifttt.com/explore

**Steps I followed**

****- Setting up ESP32 Cam on Arduino - ****
1. Install ESP32 Board on Arduino IDE
2. Now copy the below link and paste it into the “Additional Board Manager URLs” field as shown in the figure below. Then, click the “OK” button: https://dl.espressif.com/dl/package_esp32_index.json
3. Now go to Tools > Board > Boards Manager
4. In Board Manager search for ESP32 and install the “ESP32 by Espressif Systems“
5. Open the ESP32 example by using File > Examples > ESP32 > Camera and open the CameraWebServer example
6. Before uploading the code, you need to enter your Wi-Fi name and password.
7. After that define the ESP camera module. In the code, they have defined 5 camera modules so uncomment the “CAMERA_MODEL_AI_THINKER” and comment rest of the modules
8. Before uploading the code press the ESP32 reset button and then click on the upload button
9. Then open the serial monitor and change the baud rate to 115200. After that press the ESP32 reset button it will print the ESP IP address and port no on a serial monitor as shown below
10. Now to access the camera streaming, navigate to your browser and enter your ESP IP address. It will take you to the streaming page. To start the ESP32 cam video streaming click on ‘Start Stream’ button at the bottom of the page

**- IFTTT Setup for Wi-Fi Doorbell -**
1. First login to IFTTT with your credentials or Sign Up if you don’t have an account on it.
2. Search for ‘Webhooks’ and click on the Webhooks in Services section.
3. In the Webhooks window, click on ‘Documentation’ in the upper right corner to get the private key.
4. Copy this key. It will be used in the program.
5. After getting the private key, you will create an applet using Webhooks and Email services. To create an applet click on your profile and then click on ‘Create.’
6. Search for Webhooks in the search section and click on ‘Webhooks.’
7. Choose ‘Receive a Web Request’ trigger and in the next window, enter the event name as button_pressed and then click on create a trigger.
8. To complete the applet, click on ‘That’ to create a reaction for the button_pressed event.
9. Here you will play a specific song on the phone when the IoT doorbell button is pressed. For that search for ‘Android device’ in the search section.
10. Click on ‘Create action’ and then ‘Finish’ to complete the process.
11. Create another applet to send a message with the webpage link to the phone when the doorbell button is pressed.
12. It will ask to enter the phone number and message body. For this Wi-Fi doorbell project, we are sending a message with the Webserver link so that you can see the live video streaming directly.

**- CODE for IFTTT - **
#include "esp_camera.h"
#include <WiFi.h>

const char* ssid = "Wi-Fi Name";
const char* password = "Wi-Fi Password";

const char *host = "maker.ifttt.com";
const char *privateKey = "Your Private Key";

const int buttonPin = 2;
const int led1 = 14;
const int buzzer = 15;

void setup() {
  pinMode(buttonPin, INPUT);
  pinMode(led1, OUTPUT);
  pinMode(buzzer, OUTPUT);

  WiFi.begin(ssid, password);
  int led = LOW; 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    digitalWrite(led1, led);
    led = !led;
  }
  Serial.println("");
  Serial.println("WiFi connected");
  digitalWrite(led1, HIGH);

  while (WiFi.status() == WL_DISCONNECTED) {
    ESP.restart();
    digitalWrite(led1, LOW);
    Serial.print("Connection Lost");

    int reading = digitalRead(buttonPin);
if (buttonState == LOW) {
        send_event("button_pressed");
        Serial.print("button pressed");
        digitalWrite(buzzer, HIGH);
        delay(3000);
          digitalWrite(buzzer, LOW)



**Code On Arduino**

 #include "esp_camera.h"
#include <WiFi.h>
//
// WARNING!!! Make sure that you have either selected ESP32 Wrover Module,
//            or another board which has PSRAM enabled
//

// Select camera model
//#define CAMERA_MODEL_WROVER_KIT
//#define CAMERA_MODEL_ESP_EYE
//#define CAMERA_MODEL_M5STACK_PSRAM
//#define CAMERA_MODEL_M5STACK_WIDE
#define CAMERA_MODEL_AI_THINKER

#include "camera_pins.h"

void send_event(const char *event);

const char* ssid = "Galaxy-M20";
const char* password = "ac312124";

const char *host = "maker.ifttt.com";
const char *privateKey = "hUAAAz0AVvc6-NW1UmqWXXv6VQWmpiGFxx3sV5rnaM9";

const int buttonPin = 2; 
int buttonState;             // the current reading from the input pin
int lastButtonState = LOW;   // the previous reading from the input pin
const int led1 = 14;
const int buzzer = 15;

long lastDebounceTime = 0;  // the last time the output pin was toggled
long debounceDelay = 50;    // the debounce time; increase if the output flickers

void startCameraServer();

void setup() {

  pinMode(buttonPin, INPUT);
  pinMode(led1, OUTPUT);
  pinMode(buzzer, OUTPUT);
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Serial.println();

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  //init with high specs to pre-allocate larger buffers
  if(psramFound()){
    config.frame_size = FRAMESIZE_UXGA;
    config.jpeg_quality = 10;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_SVGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;
  }

#if defined(CAMERA_MODEL_ESP_EYE)
  pinMode(13, INPUT_PULLUP);
  pinMode(14, INPUT_PULLUP);
#endif

  // camera init
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

  sensor_t * s = esp_camera_sensor_get();
  //initial sensors are flipped vertically and colors are a bit saturated
  if (s->id.PID == OV3660_PID) {
    s->set_vflip(s, 1);//flip it back
    s->set_brightness(s, 1);//up the blightness just a bit
    s->set_saturation(s, -2);//lower the saturation
  }
  //drop down frame size for higher initial frame rate
  s->set_framesize(s, FRAMESIZE_QVGA);

#if defined(CAMERA_MODEL_M5STACK_WIDE)
  s->set_vflip(s, 1);
  s->set_hmirror(s, 1);
#endif

  WiFi.begin(ssid, password);
  int led = LOW;  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    digitalWrite(led1, led);
    led = !led;
  }
  
  Serial.println("");
  Serial.println("WiFi connected");
  digitalWrite(led1, HIGH);


  startCameraServer();

  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");
}

void loop() {


    while (WiFi.status() == WL_DISCONNECTED) {
    ESP.restart();
    digitalWrite(led1, LOW);
    Serial.print("Connection Lost");
      
    }
   int reading = digitalRead(buttonPin);
   if (reading != lastButtonState) {
    
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) 
  {
    // if the button state has changed:
    if (reading != buttonState) 
    {
      Serial.print("Button now ");
      Serial.println(HIGH == reading ? "HIGH" : "LOW");
      buttonState = reading;

// When the button is in the LOW state (pulled high) the button has been pressed so send the event.
      if (buttonState == LOW) {
        send_event("button_pressed");
        Serial.print("button pressed");
        digitalWrite(buzzer, HIGH);
        delay(3000);
          digitalWrite(buzzer, LOW);
      }
    }
  }

  // save the reading.  Next time through the loop,
 
  lastButtonState = reading;
}

void send_event(const char *event)
{
  Serial.print("Connecting to ");
  Serial.println(host);
  
  // Use WiFiClient class to create TCP connections
  WiFiClient client;
  const int httpPort = 80;
  if (!client.connect(host, httpPort)) {
    Serial.println("Connection failed");
    return;
  }
  
  // We now create a URI for the request
  String url = "/trigger/";
  url += event;
  url += "/with/key/";
  url += privateKey;
  
  Serial.print("Requesting URL: ");
  Serial.println(url);
  
  // This will send the request to the server
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" + 
               "Connection: close\r\n\r\n");
  while(client.connected())
  {
    if(client.available())
    {
      String line = client.readStringUntil('\r');
      Serial.print(line);
    } else {
      // No data yet, wait a bit
      delay(50);
    };
  }
  
  Serial.println();
  Serial.println("closing connection");
  client.stop();
}

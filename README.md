# Bhimospeed

## Penjelasan Code
1. Bagian header dan monfigurasi

```
#include <WiFi.h>
#include <FirebaseESP32.h>
#include "esp_camera.h"
#include "base64.h"

// Replace with your network credentials
const char* ssid = "BAROKAH";
const char* password = "alwi1973";

// Firebase configuration
#define FIREBASE_HOST "bhimospeed-default-rtdb.firebaseio.com"
#define FIREBASE_AUTH "zXdOnjPPe3xjfeuYJnhsTTBLWSNJHcE5NP5LIN5Z"

// Camera pins (for AI Thinker model)
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM     0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27

#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM       5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

```
- `Library Inclusions`: Menyertakan perpustakaan yang diperlukan untuk koneksi WiFi, Firebase, operasi kamera, dan encoding base64.
- `WiFi Credentials`: Mendefinisikan SSID dan kata sandi jaringan WiFi.
- `Firebase Configuration`: Mendefinisikan host dan token autentikasi untuk Firebase.
- `Camera Pins`: Mendefinisikan pin yang digunakan oleh kamera AI Thinker ESP32-CAM.

2. Inisialisasi Firebase dan Kamera
```
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }

  Serial.println("Connected to WiFi");

  // Set the Firebase configuration
  config.host = FIREBASE_HOST;
  config.signer.tokens.legacy_token = FIREBASE_AUTH;

  // Initialize Firebase
  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);

  // Camera configuration
  camera_config_t camera_config;
  camera_config.ledc_channel = LEDC_CHANNEL_0;
  camera_config.ledc_timer = LEDC_TIMER_0;
  camera_config.pin_d0 = Y2_GPIO_NUM;
  camera_config.pin_d1 = Y3_GPIO_NUM;
  camera_config.pin_d2 = Y4_GPIO_NUM;
  camera_config.pin_d3 = Y5_GPIO_NUM;
  camera_config.pin_d4 = Y6_GPIO_NUM;
  camera_config.pin_d5 = Y7_GPIO_NUM;
  camera_config.pin_d6 = Y8_GPIO_NUM;
  camera_config.pin_d7 = Y9_GPIO_NUM;
  camera_config.pin_xclk = XCLK_GPIO_NUM;
  camera_config.pin_pclk = PCLK_GPIO_NUM;
  camera_config.pin_vsync = VSYNC_GPIO_NUM;
  camera_config.pin_href = HREF_GPIO_NUM;
  camera_config.pin_sccb_sda = SIOD_GPIO_NUM;
  camera_config.pin_sccb_scl = SIOC_GPIO_NUM;
  camera_config.pin_pwdn = PWDN_GPIO_NUM;
  camera_config.pin_reset = RESET_GPIO_NUM;
  camera_config.xclk_freq_hz = 20000000;
  camera_config.pixel_format = PIXFORMAT_JPEG;

  if (psramFound()) {
    camera_config.frame_size = FRAMESIZE_UXGA;
    camera_config.jpeg_quality = 10;
    camera_config.fb_count = 2;
  } else {
    camera_config.frame_size = FRAMESIZE_SVGA;
    camera_config.jpeg_quality = 12;
    camera_config.fb_count = 1;
  }

  // Initialize camera
  esp_err_t err = esp_camera_init(&camera_config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }
}

```
- `FirebaseData`: Mendeklarasikan objek FirebaseData untuk mengelola operasi Firebase.
- `WiFi Connection`: Menghubungkan ke WiFi dan menunggu sampai terhubung.
- `Firebase Initialization`: Mengonfigurasi dan menginisialisasi Firebase.
- `Camera Configuration`: Mengonfigurasi pin dan pengaturan kamera sesuai dengan model `AI Thinker ESP32-CAM`.
- `Camera Initialization`: Menginisialisasi kamera dan mengecek apakah ada kesalahan.

3. Pengambilan Gambar dan Upload ke firebase
```
void loop() {
   camera_fb_t * fb = NULL;

   // Take Picture with Camera
   fb = esp_camera_fb_get();
   if (!fb) {
     return;
   } else {
     Serial.println("Camera capture Success");
   }

   // Encode the image as Base64
   String base64Image = base64::encode(fb->buf, fb->len);

   // Create a Firebase path
   String path = "/photos/photo";

   // Upload Base64 image to Firebase Realtime Database
   if (Firebase.setString(fbdo, path, base64Image)) {
   } else {
     Serial.println(fbdo.errorReason());
   }

   esp_camera_fb_return(fb);
}

```
- `Take Picture`: Mengambil gambar menggunakan kamera dan menyimpan di buffer fb.
- `Check Capture`: Memastikan pengambilan gambar berhasil. Jika gagal, kembali dari fungsi loop.
- `Encode Image`: Mengkonversi gambar ke format base64.
- `Firebase Path`: Mendefinisikan path untuk menyimpan gambar di `Firebase Realtime Database`.
- `Upload Image`: Mengunggah gambar yang telah di-encode ke Firebase.
- `Return Frame Buffer`: Mengembalikan buffer gambar ke sistem kamera.

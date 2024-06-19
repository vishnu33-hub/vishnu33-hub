#include "esp_camera.h"
#include <WiFi.h>
#include <Arduino.h>

// Camera model definition
#define CAMERA_MODEL_AI_THINKER

// WiFi credentials
const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";

// Fire detection threshold values
const int fireThreshold = 200; // Adjust based on your environment
const int firePixelThreshold = 100; // Adjust based on your environment

// Function to detect fire in an image
bool detectFire(uint8_t* imageBuffer, int imageSize) {
  int firePixels = 0;
  for (int i = 0; i < imageSize; i++) {
    uint8_t pixel = imageBuffer[i];
    if (pixel > fireThreshold) {
      firePixels++;
    }
  }
  return firePixels > firePixelThreshold;
}

void setup() {
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Serial.println();

  // Initialize camera
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = GPIO_NUM_5;
  config.pin_d1 = GPIO_NUM_18;
  config.pin_d2 = GPIO_NUM_19;
  config.pin_d3 = GPIO_NUM_21;
  config.pin_d4 = GPIO_NUM_36;
  config.pin_d5 = GPIO_NUM_39;
  config.pin_d6 = GPIO_NUM_34;
  config.pin_d7 = GPIO_NUM_35;
  config.pin_xclk = GPIO_NUM_0;
  config.pin_pclk = GPIO_NUM_22;
  config.pin_vsync = GPIO_NUM_25;
  config.pin_href = GPIO_NUM_23;
  config.pin_sccb_sda = GPIO_NUM_26;
  config.pin_sccb_scl = GPIO_NUM_27;
  config.pin_pwdn = GPIO_NUM_32;
  config.pin_reset = GPIO_NUM_4;
  config.xclk_freq_hz = 20000000;
  config.frame_size = FRAMESIZE_QVGA; // Smaller frame size for faster processing
  config.pixel_format = PIXFORMAT_GRAYSCALE; // Use grayscale for easier processing
  config.grab_mode = CAMERA_GRAB_WHEN_EMPTY;
  config.fb_location = CAMERA_FB_IN_PSRAM;
  config.jpeg_quality = 12;
  config.fb_count = 1;

  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

void loop() {
  // Capture an image
  camera_fb_t* fb = esp_camera_fb_get();
  if (!fb) {
    Serial.println("Camera capture failed");
    return;
  }

  // Detect fire in the image
  bool fireDetected = detectFire(fb->buf, fb->len);
  if (fireDetected) {
    Serial.println("Fire detected!");
    // Take action here, e.g., send an alert to a server or trigger an alarm
  } else {
    Serial.println("No fire detected");
  }

  // Release the image buffer
  esp_camera_fb_return(fb);
  delay(1000);
}

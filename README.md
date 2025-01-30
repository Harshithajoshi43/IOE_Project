# IOE_Project

## Ambuclear

### Introduction
This project is designed to empower ambulance drivers to navigate traffic signals efficiently during emergencies, minimizing delays and potentially saving lives. By enabling direct control over traffic signals, ambulances can turn red signals green, clearing their path through intersections.

### Overview
Traffic congestion in urban areas often hinders ambulances, delaying critical care during emergencies. This project provides a practical and scalable solution to address this challenge by allowing ambulance drivers to control traffic signals.

Using a secure code-based system, the ambulance driver inputs the signal's unique identifier displayed at each traffic light. The transmitter unit sends this code to the receiver unit at the traffic signal, which verifies and triggers the green light. This mechanism ensures efficient traffic management without disrupting the flow for other road users.

### Components Required
- VSD Squadron Mini Board
- Arduino Nano
- Red LED
- Green LED
- Yellow LED
- SIM 800L
- 4x4 Keypad
- Li-ion batteries

### Transmitter Circuit Diagram
<img width="719" alt="ambutranscr" src="https://github.com/user-attachments/assets/067adcb6-95ea-4bef-921d-0021785c5974" />

### Table of Connection for Transmitter

| Mini | SIM 800L | 4x4 Keypad | 
| -----|----------| --------- |
| PD6 | TXD |        |
| PD5 | RXD |        |
| GND | GND |        |
| PC7 | | C4          |
| PD2 | | C3 |
| PD3 |  | C2 |
| PD4 |  | C1 |
| PD7 | | R4 |
| PD0 | | R3 |
| PC0 |  | R2 |
| PC1 | | R1 |


### Transmitter Program

```
#include "debug.h"
#include "ch32v00x.h"
#include "ch32v00x_usart.h"
#include "ch32v00x_gpio.h"
#include "ch32v00x_rcc.h"
#include <string.h>

// Function prototypes
void USART1_Init(void);
void USART1_SendString(const char *str);

void appendCharToString(char *str, char c) {
    int i = 0;

    // Find the null terminator
    while (str[i] != '\0') {
        i++;
    }

    // Append the character and add the null terminator
    str[i] = c;
    str[i + 1] = '\0';
    Delay_Ms(1000);
}



void sendsms(char *sms){

    Delay_Ms(1000); // Wait for SIM800L to stabilize

    // Send AT commands to SIM800L
    USART1_SendString("AT\r\n");          // Handshake test
    Delay_Ms(500);

    USART1_SendString("AT+CMGF=1\r\n");   // Set TEXT mode
    Delay_Ms(500);

    USART1_SendString("AT+CMGS=\"+917892539980\"\r\n"); // Replace with country code and phone number
    Delay_Ms(500);

    USART1_SendString(sms);           // SMS text content
    Delay_Ms(500);

    USART1_SendString("\x1A");            // CTRL+Z to send the SMS
    Delay_Ms(500);


}

void GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure = {0};

    // Enable clock for GPIO port D (For the pins are connected to port D)
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC | RCC_APB2Periph_GPIOD , ENABLE);

    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_3 | GPIO_Pin_0 | GPIO_Pin_4 | GPIO_Pin_5; 
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; // Output push-pull
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz; // Maximum speed
    GPIO_Init(GPIOC, &GPIO_InitStructure); 

    // Light pin configuration
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_7; 
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPD;
    GPIO_Init(GPIOC, &GPIO_InitStructure);

    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_2 | GPIO_Pin_3 | GPIO_Pin_4; 
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPD; // Input with pull-up resistor
    GPIO_Init(GPIOD, &GPIO_InitStructure); 
    
    }
    
    
int main(void){

    SystemCoreClockUpdate();
    Delay_Init();
    GPIO_Config();
    SystemInit();
    USART1_Init();

    char str[20]="";

    while(1){


        GPIO_WriteBit(GPIOC, GPIO_Pin_0, SET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_3, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_4, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_5, RESET);

        if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_4)==1){
            appendCharToString(str, '1');}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_3)==1){
            appendCharToString(str, '2');}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_2)==1){
            appendCharToString(str, '3');}
        else if(GPIO_ReadInputDataBit(GPIOC,GPIO_Pin_7)==1){
            appendCharToString(str, 'A');}
        
        GPIO_WriteBit(GPIOC, GPIO_Pin_0, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_3, SET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_4, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_5, RESET);

        if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_4)==1){    
            appendCharToString(str, '4');}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_3)==1){
            appendCharToString(str, '5');}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_2)==1){
            appendCharToString(str, '6');}
        else if(GPIO_ReadInputDataBit(GPIOC,GPIO_Pin_7)==1){
            appendCharToString(str, 'B');}

        GPIO_WriteBit(GPIOC, GPIO_Pin_0, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_3, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_4, SET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_5, RESET);

        if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_4)==1){    
            appendCharToString(str, '7');}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_3)==1){
            appendCharToString(str, '8');}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_2)==1){
            appendCharToString(str, '9');}
        else if(GPIO_ReadInputDataBit(GPIOC,GPIO_Pin_7)==1){
            appendCharToString(str, 'C');}

        GPIO_WriteBit(GPIOC, GPIO_Pin_0, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_3, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_4, RESET);
        GPIO_WriteBit(GPIOC, GPIO_Pin_5, SET);

        if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_4)==1){    
            strcpy(str, "");}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_3)==1){
            appendCharToString(str, '0');}
        else if(GPIO_ReadInputDataBit(GPIOD,GPIO_Pin_2)==1){
            sendsms(str);
            strcpy(str, "");}
        else if(GPIO_ReadInputDataBit(GPIOC,GPIO_Pin_7)==1){
            appendCharToString(str, 'D');}
        
    }


}



void USART1_Init(void) {
    // Enable clocks for USART1 and GPIOD
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1 | RCC_APB2Periph_GPIOD, ENABLE);

    GPIO_InitTypeDef GPIO_InitStructure;
    USART_InitTypeDef USART_InitStructure;

    // Configure TX (PD5) as alternate function push-pull
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_Init(GPIOD, &GPIO_InitStructure);

    // Configure RX (PD6) as input floating
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    GPIO_Init(GPIOD, &GPIO_InitStructure);

    // USART1 configuration
    USART_InitStructure.USART_BaudRate = 9600;
    USART_InitStructure.USART_WordLength = USART_WordLength_8b;
    USART_InitStructure.USART_StopBits = USART_StopBits_1;
    USART_InitStructure.USART_Parity = USART_Parity_No;
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
    USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;

    USART_Init(USART1, &USART_InitStructure);
    USART_Cmd(USART1, ENABLE);
}

// Send a string via USART1
void USART1_SendString(const char *str) {
    while (*str) {
        USART_SendData(USART1, *str++);
        while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
    }
}

```


### Receiver Circuit Diagram

<img width="960" alt="amburec" src="https://github.com/user-attachments/assets/204bbe7b-49a8-4a41-8d9c-dd68400c677f" />

### Table for Connection of Receiver
| Arduino Nano | SIM 800L |
| --------------| ---------|
| RXD | TXD |
| TXD | RXD |

### Receiver Program

```
#include <SoftwareSerial.h>
// Create software serial object to communicate with SIM800L
SoftwareSerial mySerial(3, 2); // SIM800L Tx & Rx is connected to Arduino #3 & #2

String receivedSMS = ""; // Variable to store the received SMS

void setup()
{
  pinMode(8,OUTPUT);
  pinMode(7,OUTPUT);
  pinMode(6,OUTPUT);
  digitalWrite(8,HIGH);
  Serial.begin(9600);
  
  // Begin serial communication with Arduino and SIM800L
  mySerial.begin(9600);

  Serial.println("Initializing..."); 
  delay(1000);

  mySerial.println("AT"); // Once the handshake test is successful, it will back to OK
  updateSerial();
  
  mySerial.println("AT+CMGF=1"); // Configuring TEXT mode
  updateSerial();
  mySerial.println("AT+CNMI=1,2,0,0,0"); // Decides how newly arrived SMS messages should be handled
  updateSerial();
}

void loop()
{
  
  updateSerial();
  
  
}

void updateSerial()
{
  delay(500);
  while (Serial.available()) 
  {
    mySerial.write(Serial.read()); // Forward what Serial received to Software Serial Port
  }
  while (mySerial.available()) 
  {
    String receivedSMS = mySerial.readString();
     // Read each character from Software Serial
    Serial.print(receivedSMS.substring(51,55));
    if(receivedSMS.substring(51,55)=="123A"){
      digitalWrite(8,LOW);
      digitalWrite(7,HIGH);
      delay(2000);
      digitalWrite(7,LOW);
      digitalWrite(6,HIGH);
      delay(10000);
      digitalWrite(8,HIGH);
    }
    
  }
}

```

Project demo

https://drive.google.com/file/d/11vvJWWoTyaqmB10Zlg9usAFnEkuu_c9l/view?usp=sharing



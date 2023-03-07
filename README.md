# Sunsynk Home Assistant Dashboard
Home Assistant Dashboard to display Sunsynk info

![image](https://user-images.githubusercontent.com/7227275/223528709-bdd77d16-3352-4e40-b020-e3e25942e489.png)
![image](https://user-images.githubusercontent.com/7227275/223529017-0702437a-d12e-475a-8695-b617d8da4d97.png)
![image](https://user-images.githubusercontent.com/7227275/223529206-a97324df-0985-49e2-b63c-47e610e4ca52.png)
![image](https://user-images.githubusercontent.com/7227275/223529352-679c13b3-08e3-45b9-a3d3-041002e7d9b0.png)

## Installation
Data can be collected from the inverter using the RS485 port. There are a number of different ways to do this but I'm using an ESP32 chip running ESPHome

Create a new view on your current Dashboard and set the view type to Panel (1 card) as shown below:

![image](https://user-images.githubusercontent.com/7227275/223527428-b4508e6c-cf2d-473a-b63c-ffad11d2630d.png)

You can then edit the Dashboard (using the code editor) and paste the contents of the Dashboard.
You'll need to adjust all the sensor names to match yours and install the necessary custom components

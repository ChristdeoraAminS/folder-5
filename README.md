# folder-5
program c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Text;
using System.Windows.Forms;
using Modbus.Device;
using System.IO.Ports;



namespace WindowsFormsApplication1
{
    public partial class Form1 : Form 
    {
        ModbusSerialMaster modbusMasterRTU;
        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            foreach (string item in SerialPort.GetPortNames())
            {
                buttonConnect.Enabled = true;
                buttonDisconnect.Enabled = false;
                comboBoxSerial.Items.Add(item);
                comboBoxSerial.SelectedIndex = 0;
            }

        }

        private void buttonConnect_Click(object sender, EventArgs e)
        {
            serialPortModbus.PortName = comboBoxSerial.Text;
            if (!serialPortModbus.IsOpen)
            {
                serialPortModbus.Open();
                buttonConnect.Enabled = false;
                buttonDisconnect.Enabled = true;
                modbusMasterRTU = ModbusSerialMaster.CreateRtu(serialPortModbus);
                timerGetData.Enabled = true;
            }
            else
            {
                MessageBox.Show("Failed To Open Port");
            }

        }

        private void buttonDisconnect_Click(object sender, EventArgs e)
        {
            timerGetData.Enabled = false;
            if (serialPortModbus.IsOpen)
            {
                serialPortModbus.Close();
                buttonConnect.Enabled = true;
                buttonDisconnect.Enabled = false;
            }

        }

        
        private void buttonWriteReg_Click(object sender, EventArgs e)
        {
            ushort[] value = { 0 };
            try
            {
                value[0] = ushort.Parse(textBox7.Text);
            }
            catch (Exception err)
            {
                MessageBox.Show(err.ToString());
            }
            modbusMasterRTU.WriteMultipleRegisters(1, 6, value);

        }

        private void timerGetData_Tick(object sender, EventArgs e)
        {
             try
            {
                ushort[] regMW = modbusMasterRTU.ReadHoldingRegisters(1, 0, 8);
                textBox1.Text = regMW[0].ToString();
                textBox2.Text = regMW[1].ToString();
                textBox3.Text = regMW[2].ToString();
                textBox4.Text = regMW[3].ToString();
                textBox5.Text = regMW[4].ToString();
                textBox6.Text = regMW[5].ToString();
                textBox7.Text = regMW[6].ToString();
                textBox8.Text = regMW[7].ToString();
            }
             catch (Exception err)
             {
                 MessageBox.Show(err.ToString());
             }
        }                 
    }
}
program arduino
#include <SimpleModbusSlave.h>
#define ledPin 13 // onboard led
#define buttonPin 7 // push button
//////////////// registers of your slave /////////////////// 
enum
{
 ADC0,
 ADC1,
 ADC2,
 ADC3,
 ADC4,
 ADC5,
 LED_STATE,
 BUTTON_STATE,
 TOTAL_ERRORS,
 // leave this one
 TOTAL_REGS_SIZE
};
unsigned int holdingRegs[TOTAL_REGS_SIZE]; // function 3 and 16 register array 
void setup()
{
 modbus_configure(115200, 1, 2, TOTAL_REGS_SIZE, 0);
 pinMode(ledPin, OUTPUT);
 pinMode(buttonPin, INPUT);
}
void loop()
{
 holdingRegs[TOTAL_ERRORS] = modbus_update(holdingRegs); 
 for (byte i = 0; i < 6; i++)
 {
 holdingRegs[i] = analogRead(i);
 delayMicroseconds(50);
 }

 byte buttonState = digitalRead(buttonPin); // read button states 
  holdingRegs[BUTTON_STATE] = buttonState;

 byte ledState = holdingRegs[LED_STATE];

 if (ledState) // set led
 digitalWrite(ledPin, HIGH);
 if (ledState == 0 || buttonState) // reset led
 {
 digitalWrite(ledPin, LOW);
 holdingRegs[LED_STATE] = 0; 
 }
} 

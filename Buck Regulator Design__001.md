## Power my IC circuit

A buck regulator gives an output voltage which is less than the input voltage, this regulator offers high efficiency. Unlike Linear regulators which dissipates the excess power as heat, buck regulator does not , instead it uses PWM to adjust the switching time and thus regulate the power throughput.

This regulator has found  many applications especially in electronics, microprocessor, microcontrollers, ASICs and different embedded systems require different power level, some logic gates require 3.3 volts, others 5V and some may go as high as 15V i.e driver circuits.  In order to power each of these circuits from a single power source, we have to split the power into different voltages for different applications.

The design of a regulator is fairly straightforward,Theoretically, lets  say you have  28pins IC, each pin may require 1mA as a driving current that gives a total sum of 28mA of current, and maybe you have 5 such ICs resulting to 140mA. Giving a margin of 50% we get total current of 210mA at 5 V, you have couple of sensors each roughly taking 500mA at 5V, say 2 of such resulting to a 1A,  at 5V,furthermore you have a BJT switch with a driving gate voltage of 15V at 1A and finally a crystal oscillator that requires 3.3V at 20mA .



In such a circuit you have  stages of voltage, that is 15V, 5V and 3.3 V all in one circuit board. The first thing I do is to compute the total power required for my circuit, that is 

(5 x 0.21) + 2 (5.0 x 0.5) + (15 x 1) + (3.3 x 0.02)  = 21.116W

My circuit is fed by 24V,1.5A DC adapter.

To power the IC circuit, I need roughly 5 x 1 = 5W, my regulator requirement will be as follows

* Input voltage = 24V
* Output voltage = 5V
* Peak to peak output Voltage 20mV
* Switching frequency
* Peak to peak inductor ripple to be at 0.2A

Notice the switching frequency I have left blank as I am to determine this frequency based on the MOSFET switch I will choose, lets say I choose a switch with a switching time of 0.1uS, for maximum efficiency and reduction of switching loss, the switching period should be 100 times bigger than this which results to 10uS translating to s switching frequency of 100kHz.

### Design power circuit for 5 ICs, 5V, 1A.

Considering an input voltage of 24V, using an LDO to power this circuit is inefficient, as I need to dissipate 19V @ 1A as heat to achieve the power requirement. Besides being inefficient, this will call for increase in manufacturing cost as I need a fan , heat sinks to do some thermal management, therefore a buck regulator is the best choice to power this circuit.

**Calculating the required duty cycle (k)**
$$
k=\frac{V_a}{V_s}=\frac{5}{24} \approx 0.21
$$
**Calculating the size of a filter inductor (L)**
$$
\Delta I = \frac{V_a(V_s-V_a)}{fLV_s}
$$

$$
L=\frac{V_a(V_s-V_a)}{f\Delta IV_s}=\frac{5(24-5)}{100kHz(0.2*24)} \approx 198\mu H
$$

**Calculating the size of a filter capacitor (C)**
$$
\Delta V_c = \frac{\Delta I }{8fC}
$$

$$
C=\frac{\Delta I }{\Delta V_c8f}=\frac{0.2}{0.02*8*100kHz}=12.5\mu F
$$

<img src="C:\Users\Victor\Desktop\Buck_001_5V.png" style="zoom:50%;" />

```
Average Voltage = 5.3V at load of 12 Ohms
```

To get a better view on how the supply voltage changes with time, I am going to plot values voltage on changing resistance from 1 ohm to 5 ohms to simulate the effects of change in temperature and how it will going to affect my regulator.

<img src="C:\Users\Victor\Desktop\Buck_001_5V_Sweep.png" style="zoom:100%;"/>[]

From the simulation at 1 and 2 ohm the controller gives a very nice response and might not be in need of a controller, thereafter there are some overshoots and thus damping is required. To be safe there is a need to implement a PI controller.



### Designing 3.3V power circuit for Crystal oscillator.

For the 3.3V,20mA a buck regulator would be an overkill since I will require an inductor, capacitor, MOSFET switch which generally would result to an increase in cost of my circuit. Since this circuit only require 3.3 V @ 20mA, an LDO is the best fit. I draw power from 5V output of the buck converter and dissipate 1.7v as heat to get my required 3.3V.

Calculating efficiency
$$
P=I^2R=VI=1.7*0.02=0.034W
$$
The power drawn by the 3.3V circuit is 
$$
P=3.3*0.02 + 0.034 = 0.1W
$$


Efficiency
$$
\eta=\frac{P_{out}}{P_{in}}=\frac{0.066}{0.1}*100 = 66\%
$$
this is a good result for an LDO,considering the fact that by choosing this I have cut down the manufacturing cost by a huge margin. Besides 0.034W as heat is negligible and I can do some thermal management by means of a simple heat sink.

### Designing 15V,1A circuit to power gate drivers.

Just like the 5V,1A circuit, this circuit needs to be powered by a buck regulator. 

Calculating L,C,k values from the requirements 

Input Voltage = 24V

Output Voltage =  15V

Output peak to peak ripple = 20mV

Inductor ripple = 0.2A

| Description      | Value    |
| ---------------- | -------- |
| Filter Capacitor | 12.5uF   |
| Filter Inductor  | 281.25uH |
| Duty ratio(k)    | 0.625    |

**Simulation Results**

![voltage output](C:\Users\Victor\Desktop\Buck_001_15V.png)

Capacitor and Inductor Current waveforms

![](C:\Users\Victor\Desktop\Buck_001_15V_C_L.png)

Simulation Ripple current

![](C:\Users\Victor\Desktop\Buck_001_15V_L_C_Ripple.png)

### Component selection

In as much as what I have designed gives me the needed results, most parts are to be bought from off the shelf vendors, therefore this design process is iterative, I select a component that closely matches the design value then adjust values to fit.
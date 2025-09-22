# Application-2-Preemptive-Scheduling-with-Sensor-Integration

# Implementation Guide Questions:

1. 3.3V was used for input voltage.  For a lux of 10000, it was calculated to be 
approximately 396 ohms.


2. I do not agree with the equation because when using the formulas and nodal analysis,
Rmeasured did not work when using the provided formula.
The formula should be:
    - float resistance = 3030 * voltage / (1 - voltage / 3.3)

3. A simpler way to compute Rmeasured is:
      - Rmeasured = (Vmeasured * Rfixed) / (Vin - Vmeasured)

# Q1. Task Timing and Jitter:

The LED blink and print task both have a period of 1000 ms. The sensor task has a 
period of 500 ms. When doing further tests, the period for the LED blink changed to 510
ms per state, for a total of a 1020 ms period. I theorized that because it is the
lowest priority, it affected the initial timing. Presumably, this means there is drift
for the LED blink task. The sensor readings and alert messages occur every 500 ms. 
vTaskDelayUntil is more stable because it takes the current time and dynamically uses 
that to equal a period of 500 ms, rather than having a flat delay. The jitter that 
occurs could be possibly due to them having a lower priority than the sensor task.
Even if the blink and print task are ready, they must wait for the sensor task, so
it adds a bit of a delay. 

# Q2. Priority-Based Preemption: 

When the task for print and sensor occur at the same time, the sensor task takes
precedence. This can be shown through a console snippet:

> Solar Energy Average is 10045!

> Sensor Period is 4010 ms

> Signal received @: 4010 ms [Signal period = 1000 ms]!

> LED Status ON @ 4090

The sensor happens first because it is higher priority, followed by the print task,
then the LED task, from highest to lowest priority. 

# Q3. Effect of Task Execution Time:

If the sensor task took a longer time to execeute than normal, the lower priority 
tasks would proceed as normal. If the sensor task exceeded its period, it would run
for that extra time period. The symptoms you would see in the system would be
mistimings and improper priority behavior. 
A system designer could use more task delays if a higher priority task starved lower
priority tasks. 



# Q4.

We used vTaskDelayUntil for the sensor loop because we wanted the periods to be 
exactly 500 ms each. vTaskDelayUntil takes a time and a time period and does an 
absolute tick value. vtaskdelay is a relative period. If a period of 500 ms is applied
for 
- vTaskDelayUntil(&lastWakeTime, periodTicks);

where periodTicks is 500 ms and lastwaketime is potentially 100 ms, it will ready the
task at exactly 500ms. This is perfect for periodic real-time tasks because it is
absolute timing. On the other hand,

- vTaskDelay(pdMS_TO_TICKS(500));

is relative and delays the task for 500 ms from when it was called. If we used 
vTaskDelay(500 ms) for the sensor task, small errors could accumulate by adding
extra delays when tasks are being calculated. It would be 500 ms from that current
time and process time. For the LED task, vtaskdelay is acceptable because it is 
purely for visual effect and fine if it turns on and off. The period for the LED
does not have to be exactly 1000 ms (500 ms for states), only that it functions
correctly.

# Q5. Thematic Integration Reflection:

For the theme, I chose the space systems again because I chose it for application 1.
The LED blink task represents the sattelite system running and shows that it is. The print
status task represents the signal received by the sattelite at periodic times. The
sensor task represents the sattelite sensing the amount of light reaching it and
alerting if it is not getting enough solar power. Task priority reflects the importance
of the functions for sattelites because one of the most important tasks is to make 
sure that the sattelite is receiving power. The LED blinking status is very negligible
and low priority. The status message is important, but other crucial tasks can precede it.

# Q6. Starvation

You can starve out the system by commenting out line

- vTaskDelayUntil(&lastWakeTime, periodTicks);

This removes the delay for the sensor task, making it constantly run due to its higher
priority. The result is that the system only does the sensor task.

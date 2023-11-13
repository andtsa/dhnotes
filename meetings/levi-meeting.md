Levitation detailed control review

Centralised controller
1. take average air gap
2. 3 PIDs for roll, pitch, average air gap
3. Roll and pitch PIDs output torque, air gap PID outputs total lift
4. Lift force divided equally, forces to perform roll/pitch are added/subtracted
5. Next step is to optimise the division of forces

> [!question]- what's a PID controller?
>
> A PID (Proportional, Integral, Derivative) controller is a control loop feedback mechanism widely used in industrial control systems and applications requiring continuously modulated control. It calculates an 'error' value as the difference between a desired setpoint and a measured process variable. The controller attempts to minimize the error by adjusting the process control inputs. 
> $$u(t)=K_pe(t)+K_i\int e(t)dt+K_p\frac{de}{dt}$$
> The PID controller uses three basic modes of control: proportional, integral and derivative, which can be used separately or in combination to regulate the system.

[Sensecon] → air gap sensors very important
? → can we measure offset from lane-switch beam

[Feedforward] = take a list of well-known disturbances into account for force calculations

?? ← *cascaded current control*

[Feedforward]\ [Lane Switch] ← know precise layout of track hardcoded for calculations

[Feedforward]
![[Pasted image 20231027103106.png]]

[Minimum effort calculation] = 3 axes of freedom, 4 actuators → how do you distribute the work such that the desired force is achieved with least effort

naive algorithm → divide equally

2 approaches:
- find necessary force, calculate required current for equilibrium 
	- analytical computation
	- interpretable system
	- hard in practice because your lookup table has hardcoded zero-current point, might not correspond to reality
	- required force is really hard to measure and inaccurate to compute, since in integration your already noisy signals become too degraded
- $\Omega(n^2)$ current computation based on air gap (DH07)
	- → ensures convergence

[Cygnuses] = desired current → computations → actual current
\+ controller code for interpreting sensor inputs directly fed to the cygnuses (eg voltage) (dh07 had purely analog)
→ arcas already has access to the values from pre-existing control code

[Arcas] (still not completely sure what it does)
- communicate with ethercat to cygnuses but we don't care
- communicate with ethernet to the rest
- dh07 plugged arcas into switch, gave it ip address and directly talked to the ground station
- api is proprietary, to wrap it for running on bare metal we would need ***a lot*** of software support from prodrive which we won't get 
- EHW would like for arcas to be connected to main controller
	- unfortunately a little hard as we would need to unwrap the packets and figure how to interpret them
	- good for getting some data

[Offset Sensors]
- very noisy → problem
- dh07
	- lowpass the sensors recommend had cutoff too high 
	- → signal still noisy 
	- → they added a moving average on arcas to act as a pretend lowpass
	- arcas runs on ~5kHz, frequency analysis is impossible, would take seconds, can't implement proper lowpass
	- there's already a delay for getting the required current on the coils so a little computation time is fine.
- <u>offset sensors important</u>
- offset sensors directly plugged into the cygnuses
- dh07 **→** ***LOTS*** of EMI 

[Data path]
$$\underset{data}{\textit{Sensor}} \rightarrow \underset{\textit{Cygnus}}{\textit{Interpret sensor data}} \rightarrow$$$$\rightarrow \underset{Arcas}{\textit{Access sensor data, process it, compute desired current}} $$$$\rightarrow \underset{Cygnus}{\textit{Current computation}} \rightarrow {\textit{Coils}}$$


HAL Examples
=============

PWM to 0-10v spindle control simple
+++++++++++++++++++++++++++++++++++

.. code-block::

	#spindle DAC 0-10 control
		loadrt scale count=1
		addf scale.0 servo-thread
		setp scale.0.gain 1 #this will make m3 s1000 give 100% output and m3 s100 10%
		net spindle-speed-scale spindle.0.speed-out => scale.0.in
		net spindle-speed-abs scale.0.out => abs.0.in
		net spindle-speed-DAC abs.0.out  => remora.SP.3
	
| In the above example we create a scale and give it a value of 1
| then we link the spindle speed to the scale input
| Then we must pass the scale value into a abs as the spindle value can go negative. linux cnc uses negative to define spindle direction and we need to avoid this as the abs will always return a positive number.
| Lastly we take the abs out and link it to the remora.sp.3
| remora needs a value between 0-100 for the pwm gen.



PWM to 0-10v spindle control with inverted lincurve compensation.
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code-block::

	#spindle DAC 0-10 control
		loadrt lincurve personality=9
		addf lincurve.0 servo-thread
		loadrt scale count=1
		addf scale.0 servo-thread
		setp scale.0.gain 1 #this will make m3 s1000 give 100% output and m3 s100 10%
		net spindle-speed-scale spindle.0.speed-out => scale.0.in
		net spindle-speed-abs scale.0.out => abs.0.in
		net spindle-speed-DAC abs.0.out  => lincurve.0.in
		#Lincurve compensation
		setp lincurve.0.x-val-00 10
		setp lincurve.0.y-val-00 100
		setp lincurve.0.x-val-01 100
		setp lincurve.0.y-val-01 98
		setp lincurve.0.x-val-02 200
		setp lincurve.0.y-val-02 90
		setp lincurve.0.x-val-03 300
		setp lincurve.0.y-val-03 81
		setp lincurve.0.x-val-04 400
		setp lincurve.0.y-val-04 69
		setp lincurve.0.x-val-05 500
		setp lincurve.0.y-val-05 59
		setp lincurve.0.x-val-06 600
		setp lincurve.0.y-val-06 48.6
		setp lincurve.0.x-val-07 700
		setp lincurve.0.y-val-07 39.6
		setp lincurve.0.x-val-08 800
		setp lincurve.0.y-val-08 29.9
		setp lincurve.0.x-val-08 900
		setp lincurve.0.y-val-08 20.8
		setp lincurve.0.x-val-08 1000
		setp lincurve.0.y-val-08 12.6
		net spindle-corrected lincurve.0.out => remora.SP.3
	
| This is almost the exact same as above but adds a lincurve component to fix for the non linear PWM to 0-10v control board selected, it also fixes a problem of the cnc break out board logic being reversed.
| Such that without the lincurve 0%pwm would give out 10V(max spindle speed) and 100% pwm would give out 0V
| We take the abs out and pass it into lincurve then the table in lincurve takes a value X and replaces it with value Y and scale any value between the points.
| in the above any value between 0-10 for spindle speed gives 100 as the output thus the logic is inverted
| in the above any value between 1000 or more for spindle speed gives 12.6 
| For more info about lincurve
| https://linuxcnc.org/docs/html/man/man9/lincurve.9.html
| 
| This was tuned via a scope watching the values and making the table such that the output would be roughly linear.
	
Spindle control and coolant signal outputs
++++++++++++++++++++++++++++++++++++++++++++

.. code-block::

	# outputs
		net coolant-flood <= iocontrol.0.coolant-flood
		net spindle-on => remora.output.0
		net spindle-ccw => remora.output.1
		net coolant-flood   => remora.output.2
		


QEI Encoder without index
++++++++++++++++++++++++++++

.. code-block::

	# Initialize the encoder (spindle)
	loadrt PRUencoder names=encoder.0
	addf PRUencoder.capture-position servo-thread
	setp encoder.0.position-scale 1200.000000 #6
	# connect the hal encoder to linuxcnc
	net encoder-count <= remora.PV.2 => encoder.0.raw_count
	
| This example we add the encoder module to the linux cnc servo thread
| Then define/set the encoder with its pulse per revolution, example: 300pulse per rev encoder x4 for being a quadrature encoder equals 1200.
| Then we link the "encoder-count" to the remora PV value and pass it all into encoder.0.raw_count (the PV value will be what ever you set in the config tool/file)

QEI Encoder with index
++++++++++++++++++++++++++++++

.. code-block::

	# Initialize the encoder (spindle)
	loadrt PRUencoder names=encoder.0
	addf PRUencoder.capture-position servo-thread
	setp encoder.0.position-scale 1200.000000 #6
	# connect the hal encoder to linuxcnc
	net encoder-count <= remora.PV.2 => encoder.0.raw_count
	net encoder-phaseZ <= remora.input.7 => encoder.0.phase-Z
	
| This example we add the encoder module to the linux cnc servo thread
| Then define/set the encoder with its pulse per revolution, example: 300pulse per rev encoder x4 for being a quadrature encoder equals 1200.
| Then we link the "encoder-count" to the remora PV value and pass it all into encoder.0.raw_count (the PV value will be what ever you set in the config tool/file)
| Finally we link encoder-phaseZ to the remora input that has the index pulse connected and pass it to encoder.0.phase-Z.


Endstops + home switches
+++++++++++++++++++++++++++++++

.. code-block::

	# end-stops
	net X-min 	remora.input.0 	=> joint.0.home-sw-in joint.0.neg-lim-sw-in
	net Y-min 	remora.input.2 	=> joint.1.home-sw-in joint.1.neg-lim-sw-in
	net Z-min 	remora.input.4 	=> joint.2.home-sw-in joint.2.neg-lim-sw-in

| In the above example we are sending the value of the input to both the home-sw-in and neg-lim-sw-in
| The advantage to this is we can save on pins and simplify the machine wiring
Phase-Imagaing Ion-Cyclotron-Resonance (PI-ICR) Basics
###########

The PI-ICR technique is a precision mass spectrometry technique. It determines the cyclotron frequency by finding the phase difference between ion clusters of different accumulation times projected on a position-sensitive detector.

Theory
**********

.. image:: final and reference spots.png
   :align: center

   Figure 1: The cyclotron frequency (⍵c) is determined via the phase difference (ɸc) between two spots with different accumulation times (Tacc). In this figure, 'a' is the distance between te reference and final spots, 'b' is the distance between the center and the reference spot, and 'c' is the distance between the center and the final spot. PhIAT determines the X/Y positions of these spots to find each of their phases. 

.. image:: angles.png
   :align: center

   Figure 2: The relationship between a, b, c, and phi.

Measurement Technique
**********

* Determine the center spot 
   * Inject ions at the center of the trap
   * Reduce residual motion
   * Extract ions towards detector
* Determine magnetron spot
   * Inject ions at radius r (using Lorentz steerers)
   * Free revolution for time t_acc (accumulation time) 
   * Extract ions towards detector
* Determine reduced cyclotron spot
   * Inject ions at radius r (using Lorentz steerers)
   * Quadrupolar pulse (to ensure ion motion is entirely reduced cyclotron motion)
   * Free evolution for time t_acc (accumulation time)
   * Quadrupolar pulse to convert motion to magnetron motion (smearing effect if extract at reduced cyclotron)
   * Extract ions towards detector

This procedure will result in 3 spots/ clusters: center, magnetron, and reduced cyclotron.

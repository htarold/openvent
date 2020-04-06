# Description
This document collects my thoughts on [my demo ventilator](https://youtu.be/ME6oNFs-UiU).
This is a proof-of-concept mandatory mode pressure-controlled ventilator built around a [gasometer](https://en.wikipedia.org/wiki/Gas_holder).  The prototype is half scale (approx. 250cm3, 10cmH2O), due to materials at hand: small water trough and a single low-rate aerator.  Hoever it shows the basic layout and operation.  Briefly, the advantages are that it is inherently safe; can operate from any available air/O2 source with variable FiO2, very transparent operation, and is simple enough to be constructed by any handyman.  It can be powered by USB, avoiding dangers with mains power.

## Pressure control is safe
I gather most ventilators are usually used in volume control. However I do not see any safe, simple, and transparent way to implement volume control.

Pressure control is safe because no high pressures exist anywhere in the system.  If the trough height is chosen to be say 50cm, then even if the gasometer were to jam in the lowest position, insp pressure cannot rise above 50cmH2O, the excess being spilled.

## Peak/plateau pressures
A consequence of pressure control is that it becomes impossible to obtain peak/plateau pressures. But lung compliance and airway constriction could be diagnosed another way.  _Warning:_ all the following needs to be vetted by medical people, I am not offering medical advice, only the engineer's perspective.

If the patient's lung compliance reduces over time, then the tidal volume at the same set pressure will also reduce.  Tidal volume can be read easily from the gasometer: temporarily cut off the air supply at the very start of the insp cycle and observe the fall of the gasometer.  This is the tidal volume.

If the patient's airway becomes more constricted, then it will become necessary to increase the insp time (maintaining the same I/E ratio) in order to achieve the same tidal volume.

(If you are an EE or MechE or are versed in control theory, or have access to such a person, then the lung/airway may be modelled as a first order system, an RC low pass filter.  When volume control is used, this is analogous to applying a step current source input.  The output is the voltage, whose curve will look exactly like the usual ventilator pressure curve.  However if a step pressure is applied (pressure control), then instead of peak/plateau, you'll see a spike in current output decaying exponentially.  Integrate this to get the charge curve which rises rapidly then approaches a limit, again showing the exponential curve.  This charge curve is analogous to the volume observed in the gasometer.  Now the rub here is whether or not the lungs/airway may be fairly modelled as a first order system.)

# Modules and components, and their alternatives
If you're considering building a ventilator in quantity, then the design will depend heavily on the components available.
## Control
As convenient as the microncontroller e.g. Arduino is for this purpose, there may be concerns.  For instance the code is too easy to tamper with, and the libraries may not have been audited.  To some extent this may be addressed by writing the software in MISRA-C or in assembly.  Another option is to use analogue hardware to control the ventilator.

The basic component of analogue control is the retriggerable one-shot timer.  This electronic module, when triggered, waits a certain amount of time before turning on/off it's connected accessory which could be a valve or an alarm.  With two such timers triggering each other and their insp/exp valves, the basic mandatory mode ventilator is achieved.  These timers can be designed and the first units of a batch of several thousand received within 10 days, maybe less.

## Valve issues
Availability of valves may be the biggest determinant of ventilator design.
### Solenoid valves
It's tempting to consider solenoid valves, and if exactly the right ones are available then they would be ideal.  However, the most common such valves are pilot valves which require a significant pressure drop to operate properly.  Direct acting solenoid valves are better in this regard, but their flow rate is lower, so it may be necessary to parallel multiple valves, even as valves become harder to find.  Solenoid valves are usually 12V or 24V or mains powered which is less than ideal.
### Pinch and soft-kink valves
These are like like soft-kink valves used in the prototype but do not need bending strain space.  Like soft-kink valves, it's possible to use a single double-acting actuator instead of requiring 2, reducing parts count.  But be careful it is break-before-make, i.e. that at no time are both valves open; then air just goes straight from gasometer to PEEP valve without being breathed first.
### RC servos
The prototype used RC servos.  These have the drive electronics already integrated, so interfacing to them is trivial.  Older "analogue" servos are preferred IMO, the newer digital ones draw more current.  If nice supply silicone tubing is used, then peak current can be held to below 1A (this must be measured), and conveniently phone power banks + USB chargers can be used for power supply.
### Auto door lock actuators
Every modern car has 4 door lock actuators.  These are rated at 12V but will run at 5V.  They operate a bit like solenoids, but are capable of push _and_ pull.
### Mechanical vs. solid state relays, MOSFETs
It's tempting to use relays to actuate the valve, but mechanical relays typically have a life of less then 100k cycles.  This means they can be expected to fail in a few days of continuous use.  Solid state relays are better, but are more expensive.  MOSFETs are the best if you are building a circuit board. Integrating the actuator driver into the board is easily the best option.  But mind you 12V+ driver chips are not common, so discret transistors may be necessary (and desireable IMO).
### Tubing
The prototype used 10mm ID PVC tubing.  Ideally thin walled silicone tubing should be used, as this is more flexible (suitable for the soft-kink valves and pinch valves).

# Extra components required for operational use
## Water reservoir location
It should go without saying that all water-bearing components must be mounted at a (much) lower level than the patient.
## Water traps
The Insp and Exp tubes must carry inline water traps of sufficient capacity, to prevent water from the PEEP valve and gasometer from drowning the patient.  
## Alarms
At least 1 alarm must be installed, it's purpose being to monitor the gasometer's rising and falling.  The alarm switch can be a reed switch with magnet, or a microswitch, or inductive proximity sensor.  In either case, the alarm timer is reset when the switch closes and reopens, as will happen when the gasometer reaches it's highest travel and starts back down.  When the alarm timer runs out, the alarm sounds.
## Drip tray
## Exp filter
To decontaminate exhaled air.

This alarm switch can then detect a variety of problems: water trough leak, reduced air/O2 flow, patient distress, broken/disconnected tubing, etc.

If a microcontroller (e.g. Arduino) is used to control the valves, then the same microcontroller can be used to monitor the alarm switch.

# Desideratum
An assist mode can be added quite easily: a pressure switch can be added to the insp circuit which closes when the patient inhales or tries to.  This microcontroller senses this and immediately cuts short the current insp/exp cycle and starts a new one.  An alarm may be included to sound when assist mode is entered, *and also* upon reversion to mandatory mode.

This inhale sensor can be constructed out of a U-tube and float switch.  Naturally this *must* be situated *before* the water trap.

# Operational gotchas
. Bubbles should not be too large, and there should be a generous gap between trough wall and gasometer.  Large bubbles spill violently, and may clog up the annular gap between trough and gasometer.  This will change the pressure set point.
. For similar reasons, a notch should be cut in the gasometer wall, so that air spillage occurs initially at this point and in an orderly manner, instead of erupting suddenly.
. Before the patient is hooked up, the air supply should be set quite high.  After the patient is hooked up and the I/E is dialed in, and periodically thereafter, the air should be adjusted such that at least a *small amount* of air is seen to spill past the gasometer.  This excess air ensures that if the patient's condition changes to require more air, or if the air pressure dwindles, that the ventilator can accommodate.

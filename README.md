# Why
There was a lot of concern about the lack of ventilators so I gave
it some thought and came up with this.  The ventilator would
have to be made of commonly available parts, as it would be
uncertain what would be available.  It would have to be safe,
and this means the operation must be transparent.  It would have
to be simple, in order to permit rapid construction and
verification of performance.  It turns out these goals are
easily achievable.

Today however it seems folks have the problem well in hand.
I'm keeping this up anyway, as it is simpler than any other I've
seen, and it may yet be useful to someone, somewhere.

# Description
## Prototype
This document collects my thoughts on [my demo ventilator](https://youtu.be/ME6oNFs-UiU).

This is a proof-of-concept mandatory mode
pressure-controlled ventilator built around a
[gasometer](https://en.wikipedia.org/wiki/Gas_holder) which is
simply an upturned vessel in a water trough that collects air
under it.  As it fills, the gasometer floats higher in the water.
It cannot be overfilled; air would simply leak out from under it.
The pressure in the gasometer is determined by the weight placed on
it, and this pressure is read as the difference between
the water levels inside and outside the vessel.  To increase the
pressure, more weight is placed on the gasometer, forcing it
to sit deeper in the water.  Schematically it looks like
[this](./mech-vent.pdf).

The gasometer maintains a steady supply of air in addition to
ensuring it is provided at the prescribed pressure.  In normal
operation, the air supply is adjusted such that there is a
slight excess which can be seen to occasionally spill from the
gasometer.  This is the correct setting.

It is clear that O2 as well as air may also be be used to fill
the gasometer, in any fraction.  Humidification of the air takes
place automatically, and this may be adjusted by raising or
lowering the exit of the delivery tube(s) in the water column.

Any source of air can be used, not just hospital air.

Air from the gasometer feeds the patient via a tube;
here it is PVC, although ideally it would be silicone which is more
flexible and better suited to the kink valve, which serves as the
inspiratory valve opening and shutting to control air to the patient.

Another tube carries away the patient's exhalation, and this is
controlled by the expiratory kink valve of identical construction,
which is open if the inspiratory valve is closed, and closed if
the inspiratory valve is open.

The 2 kink valves are actuated from a single RC servo, which is
controlled by an Arduino Nano microcontroller.  Both the
RC servo and microcontroller operate from a 5V electrical supply.

After the expiratory valve, the gases are piped into a PEEP
valve to provide back pressure.  This is made by inserting the
expiratory tube the requisite depth into a container of water.

The operation of all mechanical components is transparent, so
any faults are trivial to detect and clear.

This prototype is half scale (approx. 250cm3, 10cmH2O insp,
2cmH2O exp), which limitation is due to materials at hand: small
water trough and a single low-rate aerator.  However the
basic layout and operation is shown.

I expect the weak points in the design to be the
servo and the kink valve, but as of this writing the prototype
has been running non-stop 8 days and counting without failure.

## Operationalising
In order to make a useable model, some changes must be made.

Capacity needs to be scaled up to at least 1000ml or two breaths.
This means the gasometer needs to be larger, and the water trough
needs to be taller, totalling approximately 50cm in height in
order to generate 35-40 cmH2O of insp pressure.  Both these items
should be transparent so that the water levels are easily visible.
Preferably they would be graduated so volumes can be easily read.

The water trough needs to be mounted low to avoid possibility of water
entering the insp and exp circuits.  Table-top operation is not
a good idea, it should be floor mounted, possibly on castors.
An office chair may be used, with the seat removed and a pole added
upon which to mount the components.

The gasometer should be able to rise and fall freely, and it
should have a guard above it to prevent unintentional contact
that can alter the gasometer pressure.

Water traps of sufficient capacity must be added in line with
the insp and exp circuits.  These should also be mounted low.

An alarm must be added.  This is preferably a magnetic reed switch,
inductive
proximity switch, or capacitive sensor which closes only when the
gasometer reaches its maximum height.  This is sufficient to detect
various fault conditions:
- loss or reduction of air/O2 flow
- incorrect air/O2 adjustment
- lack of insp/exp cycling (for any reason)
- alarm switch failure
- alarm misadjusted

The code to implement the alarm basically resets a watchdog in
the microcontroller
upon any change in alarm switch state.  Watchdog timeout can be
fixed at between 1 and 2 breath cycles.

Insp/exp time should be set using rotary potentiometers, the position
of which is read by the microcontroller each cycle.  Instead of
showing the time on a display, the position of the knob on the
potentiometer indicates clearly the time duration set.

These two potentiometers are the only 2 electronic controls on
the system.  In addition, weight on the gasometer controls insp
pressure, and PEEP delivery tube insertion depth controls exp
pressure.

Mechanical potentiometers obviate the need for display software:
less code means less code that can go wrong.  Probably only a couple
of hundred lines of code are needed in total for the system.

If there is still concern regarding the software, the entire
control system is simple enough to be implemented in hardware
alone, the basic building block being the retriggerable monostable
multivibrator with adjustable duration.  This approach is probably
unnecessarily conservative.

Power for the microcontroller and servo should come from a phone
power bank, and this should be kept charged with a normal phone
charger.  Even a small power bank (single 18650) should be enough
to power the system for an hour if disconnected from the charger.
Many power banks come with state-of-charge indicators, which may
prove useful.  Mains power does not enter the system.  With a
portable oxygen bottle and castors on the ventilator base, the
patient can be readily moved around with the ventilator.

### Assist mode
Assist mode after a fashion may be optionally and easily implemented.

A pressure sensor will need to be attached to the insp circuit,
which triggers when insp pressure falls to a preset
value.  Then the microcontroller is retriggered to immediately
start a new breath cycle.

There are many MEMS pressure sensors that can be used, but a very
simple mechanical one may be made with a U-tube and electrical
float switch.
This U-tube must be situated before the water trap.  If a float
switch is to be used, it may be integrated with the PEEP valve
instead of being a separate device.

Whenever the mode changes (to/from assist or mandatory),
the alarm should be activated.

## Parts
- Tubing.  For the valve, PVC tubing was used but silicone being
  more elastic is preferable.  ID/OD was 8mm/10mm.  However as
  of this writing, the original PVC tubing has survived 8 days
  non-stop.  The tubing must be routed to take the motion (of
  the kink valve and of the gasometer) without stressing the
  material or kinking at the wrong place.
- Servo can be [MG995](https://www.ebay.com/sch/i.html?_nkw=mg995)
  /996 or MG945/946, or any available standard
  or mini size digital servo.  The suggested servos are cheap
  and can be had in quantity.  Being recreational items
  there are not likely to be in short supply.
- Valve assembly can be laser cut from acrylic then assembled
  and cemented.  This is a fast operation since there is little
  machine set up time.
- Water trough.  If transparent DWV PVC pipe can be found, this
  would be perfect.  Otherwise, the best bet is probably acrylic.
  This can be
  made by businesses dealing with illuminated signs etc.  If you
  are able to source acrylic pipe, 4-5 inches ID and 50-60cm
  tall would be
  suitable.  This material does not tap well, so air delivery
  tubes should be in the form of a U so it dips into the water
  trough from above, to avoid putting any holes in the material
  below the water line. It is best to avoid an overlarge trough
  and too much water.
- Gasometer can be made similarly to the trough, or use a
  Nalgene water bottle.  Electrical cable glands can be used to
  connect the insp tube.
- Microcontroller  Prototype used an [Arduino
  Nano](https://www.ebay.com/sch/i.html?_nkw=arduino+nano) because it is
  easily available.  This is perfectly useable but a 3.3V
  microcontroller will have more headroom to filter noise.
- Pressure assist sensor [This](http://www.faranux.com/product/mps20n0040d-d-sphygmomanometer-pressure-sensor-0-40kpa-dip-6-for-arduino/)
  is suitable, and there many similar parts.  A [float
  switch](https://www.ebay.com/sch/i.html?_nkw=float+switch) can
  also be used.


# Setting a prescription
_The following is a suggestion of procedure, it is not medical
advice._

Say the patient is to be prescribed a certain tidal volume,
PEEP pressure, and bpm.

1. Plug the insp tube (where it would attach to the ET tube).
1. Start the air (with high initial FiO2), and set to quite a high
  flow rate.  Power on the system if not already on.
1. Adjust gasometer weight to a safe pressure, say 25 cmH2O.
  I.e. water level in the gasometer is 25 cm below trough water
  level.
1. Adjust PEEP pressure to prescribed, e.g. delivery tube is inserted
  10 cm deep into the water for 10 cmH2O.
1. Adjust system to 2 seconds insp, 4 seconds exp
  or equivalent in terms of I/E and bpm.
1. Connect patient.  Allow breathing to settle (no stacking).
1. Ensure there is a decided excess of air: increase air until some of it
  can be seen to escape past the gasometer when it is at max height.
1. Verify PEEP bubbles on exp.
1. Check tidal volume: when gasometer rises to max
  height, cut off the air temporarily.  Note height the gasometer falls
  to; the difference is the tidal volume.  Turn on the air
  again.  (This is why gasometer capacity should be 2 breaths,
  and why excess air is necessary).
1. If tidal volume is too low, either increase insp pressure or
  increase insp time.  Judgment is needed here.  A
  useable strategy might be to increase insp time until volume no
  longer increases; thereafter increase pressure.
1. To increase insp pressure, increase gasometer weight without
  exceeding max prescribed, e.g. 50 cmH2O.
  If pressure would exceed, then this prescription
  cannot be fulfilled with this patient.
1. Verify excess air.  Tidal volume reading will be wrong
  without excess air.
1. Adjust FiO2 down to desired.

This is simpler and more commonsensical than it may sound.

Also periodically:
1. Optionally verify alarm works: hold it closed for 2 breath cycles.
  Alarm should sound.
1. Verify there is excess air (some bubbles escape the
  gasometer when at max height).
1. Check PEEP and replenish water as it evaporates, and
  adjust PEEP pressure.
1. Replenish trough water as it evaporates, and verify
  alarm is set to correct height.
1. If tidal volume has decreased, see if temporarily increasing the insp
  time will restore volume.  If it does, this can indicate worsening airway
  constriction.  This is similar to seeing an increase in peak
  pressure in a volume controlled vent.
1. Check and note tidal volume, readjust if necessary.
  A reduction in tidal volume over time (other settings remaining
  the same) can only mean lung compliance has decreased.  This is
  like taking the plateau pressure: if lung
  compliance has reduced, under volume control plateau pressure would
  increase.  Under pressure control, tidal volume would decrease
  (plateau pressure not being meaningful here).



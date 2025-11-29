# ESP32Devboard
A custom esp32 devboard with traces for all buttons and potentiometer inputs. Integrated with another esp32 and a GC+2 from 4layertech to emulate a gamecube controller on supported hardware(Wii/Gamecube).


I first needed to consider applications of the MCU. I'm using a GC +2.0 gamecube controller emulator from 4layer tech.

<img width="750" height="623" alt="image" src="https://github.com/user-attachments/assets/ad639df6-509c-4356-82ad-24845fc4d9f6" />

To access my required button features, I determined it necessary to use 17 IO pins for a combination of digital and PWM outputs. For my vibration motors, I needed to figure out if they required digital or analog input. Researching the electrical diagram for the motor controller on the GC+2.0 showed that PWM was used for the controls as well, allowing me to directly tap the PWM and feed it through an ESP32 IO input.
<img width="1334" height="645" alt="image" src="https://github.com/user-attachments/assets/b132d056-6d50-479f-afc6-8b115213d64f" />


For flash memory access, I decided to implement the USB-C spec. The ESP32 has D+ and D- pins that can directly interface with it.

<img width="638" height="632" alt="image" src="https://github.com/user-attachments/assets/0307c93c-bc62-4d4d-984b-0ed6b5ca8e3e" />


To drive the motors wirelessly, a small motor controller would be required. I decided on the L293D, since its easily implementable, has a small footprint, and has the ability to control more than 1 motor if I would need it in the future.

<img width="798" height="527" alt="image" src="https://github.com/user-attachments/assets/b91a693b-7174-4d41-8cc5-61a15fa480a3" />

Gamecube controller rumble motors draw 5v or 3.3v. For the current draw on the 3.3v line, the controller requires a max of 1.5A. Standard voltage regulators cannot supply this, so I had to spring for a high-end 2A voltage regulator. There wasn't much documentation, so I had to mimic the standard circuit provided in the datasheet; mostly standard, with power lines connected to decoupling capacitors which bypass to ground.

<img width="552" height="289" alt="image" src="https://github.com/user-attachments/assets/276c1d2c-c82f-4b84-84fb-e038846e6678" />

For interfacing and troubleshooting components, I added 3 status LEDs(3.3v test, 5v test, blink/file upload) as well as 40 pins.

Boot and reset buttons were standard, pulling EN high and pulling IO0 low.

Finally, for wireless low latency capabilities, I decided on using ESPNOW as a communication protocol, requiring antenna access on the module.

With the rough layout completed of what I would need, I drafted a schematic using the ESP32 schematic checklist and external open source schematics. Be aware if you are replicating that the voltage regulator VOUT sense is marked as an output instead of an input on the TI symbol; ERC will throw an error, its the symbol's fault, shouldn't be an issue.

<img width="484" height="365" alt="image" src="https://github.com/user-attachments/assets/fff0bb04-4bbc-4585-82d0-c1528d27e818" />


The footprint for the USB-C port in KiCad for the M-12 USB C is faulty, you need to download a revised version from the manufacturer. The mechanical holes only have .2mm clearance as well, so I shrunk some of the pads on the final pcb to account for this.

<img width="1248" height="699" alt="image" src="https://github.com/user-attachments/assets/8d0e2dc7-9f34-4d52-bfd9-e32f144849f6" />

With the schematic done, I began to lay out the PCB. Organization was fairly self-explanatory, with my goal to minimize overlapping traces and minimizing space between pins and decoupling capacitors/pulldown resistors. Because of the compact nature of my board, as well as integration for 5v and 3.3v circuits, I decided to use a 4 layer board to maximize signal integrity. Layer 1 was for signal wires and short power connections, layer 2 was my ground plane, layer 3 was a mix of 3.3v copper fill and 5v fill, and layer 4 was for aux connections. Figuring out layer 3 was the biggest hurdle, since I wasn't able to overlap 3.3v and 5v traces, including signal wires. I had to minimize signal traces moving over fill boundaries as well. After some work, I managed to get a decent arrangement to start routing.

<img width="621" height="722" alt="image" src="https://github.com/user-attachments/assets/457c4754-99f9-4013-a484-daa045c22ae2" />

I prioritized USB-C signal integrity by mapping the data pins as a differential pair.

<img width="216" height="645" alt="image" src="https://github.com/user-attachments/assets/bd4414b8-9fe9-4ed0-85f9-762621a41bda" />

Because of the nature of the board as a 4 layer, I was able to have easy trace management with little overlap. The power and ground planes allowed me to just connect through-hole vias to any 3v3, 5v, and GND(with the exemption of those needing pulldowns or decoupling). Notably, I needed to swap my motor PS capacitor for a bulk capacitor due to its potentially high impact on PWM motor signal integrity. Routing all the traces got me this finished board.

<img width="830" height="749" alt="image" src="https://github.com/user-attachments/assets/a16ddcfe-720c-455e-88fa-6d11d8fba493" />

Because of the variety of parts outside the JLCPCB and KiCad standard parts library, and due to component shortages by JLCPCB, many components needed off-market alternatives, like the LEDs and my bulk capacitor. Notable choice was to use the n16 model of the ESP-32 for high flash storage(macro/recall).

<img width="1125" height="680" alt="image" src="https://github.com/user-attachments/assets/d274dd8c-f0d7-454d-9d02-d2053e5b0764" />

JLCPCB CAD Viewer(USBC and ESP are offset for some reason, pins line up but apparently its an issue that gets fixed in DFM).

<img width="814" height="625" alt="image" src="https://github.com/user-attachments/assets/489e43dd-d8c6-4a08-91bd-8b3172943948" />

Overall, the project turned out really well for a first time PCB. I had little to no references for my requirements, but I managed to actually make something that could work! No errors in ERC(except aforementioned ignore) or DRC, just warnings for cosmetic errors, isolated copper fills, and reference errors to the standard footprint library since I edited the footprints. Practically everything follows the recommended spec from manufacturer datasheets.

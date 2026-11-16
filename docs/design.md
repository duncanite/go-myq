# go-myq: the design

Research into Chamberlain and LiftMaster garage door openers with myQ, done on 2026-11-15
from public sources, to decide what a Go module for them would be. Nothing is built and
nothing here was run against an opener. The decisions at the end are open.

In short: myQ has no local interface, and Chamberlain closed its cloud to outside clients
in late 2023. The local route is a board wired to the opener's wall-control terminals,
which speaks the opener's own protocol and serves the door on the LAN. It works on
openers of the Security+ and Security+ 2.0 generations, and not on Security+ 3.0.

## The device

- An opener with myQ reaches Chamberlain's cloud over Wi-Fi, built in or through an
  add-on hub. Every source describes the app as going through the cloud; nothing found
  shows it on the LAN.
- The opener and its wall control talk over the two wires between them. On Security+ 2.0
  that is an obfuscated serial signal, which a relay closing the contacts cannot imitate
  ([the first ratgdo](https://github.com/PaulWieland/ratgdo)).
- On Security+ 3.0 the wires carry power only and the wall control talks to the opener
  over encrypted Bluetooth
  ([an open issue of esphome-ratgdo](https://github.com/ratgdo/esphome-ratgdo/issues/492)).

## Ways in

### Chamberlain's cloud

Closed. Home Assistant [removed its myQ integration](https://www.home-assistant.io/blog/2023/11/06/removal-of-myq-integration/)
in its release of December 2023, after Chamberlain blocked it and offered a paid partner
program instead. The open clients have since been archived:
[homebridge-myq](https://github.com/hjdhjd/homebridge-myq) in 2024, pointing its users to
ratgdo, and [pymyq](https://github.com/arraylabs/pymyq) in 2026.

### A board on the wall-control terminals (local, with hardware)

- **ratgdo**, with the [esphome-ratgdo](https://github.com/ratgdo/esphome-ratgdo)
  firmware: three wires to the opener's terminals. It speaks Security+ 2.0, Security+ 1.0
  and dry contact. It gives the door, with its position once calibrated, the light, the
  lock-out of remotes, motion, obstruction, the motor, the wall button and a count of
  openings, over ESPHome's native API. Firmware that serves HomeKit exists as well.
- **Konnected GDO blaQ**, with [its ESPHome firmware](https://github.com/konnected-io/konnected-esphome):
  Security+ 1.0 and 2.0, the same kind of entities, and a local REST interface beside
  ESPHome's API.
- Neither works on Security+ 3.0: the issue above is where its Bluetooth is being
  reverse-engineered, without a working result.

### Matter, HomeKit

Chamberlain sells no Matter product, and its HomeKit bridge is discontinued.

## What this module would be

Not decided. The options, most local first:

1. **A client of a board on the opener**, over ESPHome's native API: the entities
   listed, their state subscribed to, the door, the light and the lock driven. Protobuf
   over TCP, optionally encrypted. The Go work is that client, or the choice of an
   existing one; buying and wiring the board is the owner's.
2. **Nothing in software alone.** A client of Chamberlain's cloud would be written
   against an interface its owner breaks on purpose.

No Go code for myQ or Security+ was found. Go clients of ESPHome's native API exist
([esphome-apiclient](https://github.com/richard87/esphome-apiclient),
[esphome_api](https://github.com/mycontroller-org/esphome_api)); none was evaluated.

The client of ESPHome's native API is the same piece the Emporia unit and a GE appliance
would need after their own hardware step: one module could serve the three.

## Open questions

- The colour of the opener's learn button: yellow is Security+ 2.0, purple, red, orange
  or brown Security+ 1.0, white Security+ 3.0, for which no board works.
- The opener's model number.
- Whether a board on the opener is acceptable.

## Not verified

- Nothing above was observed on an opener.
- Chamberlain's own statement on the block: its press pages did not render; it is known
  here through Home Assistant's account.
- That the blaQ does not work on Security+ 3.0 is from a secondary source.
- The other relay-and-sensor devices sold for garage doors were not looked into.
- The Go clients of ESPHome's API: their state and completeness.

## Sources

- [Home Assistant, removal of the myQ integration](https://www.home-assistant.io/blog/2023/11/06/removal-of-myq-integration/):
  the block of 2023 and what followed.
- [esphome-ratgdo](https://github.com/ratgdo/esphome-ratgdo): the board's firmware,
  protocols and entities; [its issue 492](https://github.com/ratgdo/esphome-ratgdo/issues/492):
  Security+ 3.0.
- [ratgdo](https://github.com/PaulWieland/ratgdo): the first firmware; the wire's signal.
- [konnected-esphome](https://github.com/konnected-io/konnected-esphome): the blaQ.
- [secplus](https://github.com/argilo/secplus): the Security+ codes decoded.
- [homebridge-myq](https://github.com/hjdhjd/homebridge-myq) and
  [pymyq](https://github.com/arraylabs/pymyq): the archived cloud clients.
- [esphome-apiclient](https://github.com/richard87/esphome-apiclient) and
  [esphome_api](https://github.com/mycontroller-org/esphome_api): Go clients of ESPHome's
  native API.

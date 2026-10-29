# Flying Computer II

We build an open, modular radio and video stack for a small UAV rover: embedded firmware on the vehicle, a matching ground station, and tools for video, control, and clock synchronization.

## System at a glance

```mermaid
flowchart LR
  subgraph R[ROVER]
    CAM[OV5647 camera] --> TX[ESP32-P4\nH.264 + RTP/FEC]
    TX --> RL3[winject-l3]
    RC[RC + telemetry] <--> RL3
    CLK[krono client] <--> RL3
    RL3 --> RP[ESP32-C6 radio]
  end

  RP <-->|wireless link| GP[ESP32-C6 radio]

  subgraph G[GROUND]
    GP --> GL3[winject-l3]
    GL3 --> RX[ESP32-P4\nvideo receiver]
    RX --> USB[USB]
    USB --> AND[Android ground app]
    RX --> MC[Ethernet RTP multicast]
    MC --> HOST[GS Linux host\ndecode + display]
    GL3 <--> CLK2[krono reference]
  end
```

The rover and ground P4 boards run matching winject-l3 radio configurations. The C6 handles the radio interface over SDIO. On the rover, the P4 captures OV5647 video and encodes H.264; on the ground P4, the receiver recovers the stream and sends it to Android over USB and to the local network over RTP multicast.

## Repositories

### Rover

- [rov-esp32p4](https://github.com/FlyingComputer2/rov-esp32p4) — Rover P4 application: radio proxy, winject-l3, live camera/video transmit, clock-sync client, and RC rover sample.
- [vstreamer](https://github.com/FlyingComputer2/vstreamer) — Composable camera, codec, RTP, and streaming components used by the video pipeline.
- [HAL](https://github.com/FlyingComputer2/HAL) — Hardware abstraction interface declarations.

### Ground station

- [gnd-esp32p4](https://github.com/FlyingComputer2/gnd-esp32p4) — Ground P4 firmware: radio proxy, winject-l3, video receiver, USB transport, and Ethernet multicast.
- [gnd-android](https://github.com/FlyingComputer2/gnd-android) — Android USB ground app for viewing rover video and carrying ground-side channels.
- [gnd-vstreamer-mcsink](https://github.com/FlyingComputer2/gnd-vstreamer-mcsink) — Linux receiver and display path for RTP H.264 multicast, with decode metrics and OSD.
- [gnd](https://github.com/FlyingComputer2/gnd) — Earlier Orange Pi 5 ground station using RTL8812AU; archived as the stack moved to the P4 ground board.

### Shared components

- [krono](https://github.com/FlyingComputer2/krono) — Clock sources and synchronization for rover, ground boards, and host.
- [winject-l3](https://github.com/winject-wireless/winject-l3) — Layer 3 radio manager.
- [winject-radio-esp32-sdio-proxy](https://github.com/winject-wireless/winject-radio-esp32-sdio-proxy) and [winject-radio-esp32c6](https://github.com/winject-wireless/winject-radio-esp32c6) — ESP32-P4/C6 radio interface and C6 radio firmware.

## Video, control, and time

- **Video:** OV5647 camera → hardware H.264 encode → RTP with application-level Reed–Solomon FEC → radio link → ground receive and fanout.
- **Control and telemetry:** RC and rover telemetry travel on a separate radio upstream; the ground P4 bridges frames to Android USB and bench tools.
- **Clock sync:** The ground P4 provides the reference clock. The rover synchronizes over the radio, while the Android app synchronizes over USB; the ground host follows the ground P4.
- **Ground viewing:** Android receives video over USB. The P4 also multicasts recovered RTP H.264 to the ground network for a Linux decode/display path.

The repositories contain build instructions, interface details, wiring notes, and bench procedures. Start with [rov-esp32p4](https://github.com/FlyingComputer2/rov-esp32p4) for the rover or [gnd-esp32p4](https://github.com/FlyingComputer2/gnd-esp32p4) for the ground board.

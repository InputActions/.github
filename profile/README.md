# InputActions
- **Installation guide & getting started:** [wiki.inputactions.org/main/getting-started](https://wiki.inputactions.org/main/getting-started)
- **Issue tracker and discussions for all projects:** [github.com/InputActions/discussions](https://github.com/InputActions/discussions)

InputActions is a Linux utility for binding various input device actions (keyboard shortcuts, mouse/touchpad gestures etc.) to system actions.

> [!IMPORTANT]
> **The project is created primarily for KDE Plasma and will work best in that environment.** In other environments certain features may not be available,
more information about that can be found in the aforementioned installation guide.

Features:
- Supported device types: keyboard, mouse, touchpad, touchscreen
- Input event filtering
- Uses libinput, which handles device quirks
- Complementary evdev input backend for better touchpad support (may be disabled in case of issues)
- Conditional triggers
- Built-in action for simulating keyboard and mouse input

<details>
  <summary>Example configuration</summary>

  ```yaml
  device_rules:
    # ignore a device
    - conditions: $name contains YubiKey
      ignore: true

  keyboard:
    gestures:
      # shift + meta + q -> kill window under pointer
      - type: shortcut
        shortcut: [ leftshift, leftmeta, q ]

        actions:
          - on: begin
            command: kill -9 $window_under_pointer_pid

      # text expansion (compositor plugins only)
      - type: shortcut
        shortcut: [ leftctrl, space ]

        actions:
          - on: begin
            replace_text:
              # :calc{2+2} -> 4
              - regex: :calc{(.*)}
                replace:
                  command: printf "$(qalc -t "$match_1")"

               # :email -> example@example.com
              - regex: :email
                replace: example@example.com

  mouse:
    gestures:
      # right + draw circle clockwise -> open dolphin
      - type: stroke
        strokes: [ 'Gw4A/DELBwxLFRAZWiUXJWM6HzBkSyRKWlQpYShYOasISUXHACVR6Q8WWP0zEmQA' ]
        mouse_buttons: [ right ]

        actions:
          - command: dolphin

      # trigger group - condition is applied to all subtriggers specified in 'gestures'
      - conditions: $window_class == firefox
        gestures:
          # meta + vertical wheel -> volume control
          - type: wheel
            direction: up_down

            conditions: $keyboard_modifiers == meta

            actions:
              - on: update
                interval: '+'
                input:
                  - keyboard: [ volumedown ]

              - on: update
                interval: '-'
                input:
                  - keyboard: [ volumeup ]

      # this trigger will override the one below due to higher priority, but only if firefox is focused
      - type: press
        mouse_buttons: [ middle ]
        instant: true

        conditions: $window_class == firefox

        actions: []

      - type: press
        mouse_buttons: [ middle ]
        instant: true

        actions: []

  touchpad:
    gestures:
      # place 2 fingers, at least 1 on the top/bottom edge, then move in circular motion -> circular scrolling
      - type: circle
        fingers: 2
        direction: any

        conditions:
          any:
            - $finger_1_initial_position_percentage_y <= 0.05
            - $finger_2_initial_position_percentage_y <= 0.05
            - $finger_1_initial_position_percentage_y >= 0.95
            - $finger_2_initial_position_percentage_y >= 0.95

        actions:
          - on: update
            interval: -0.5
            input:
              - mouse: [ wheel 0 -1 ]

          - on: update
            interval: 0.5
            input:
              - mouse: [ wheel 0 1 ]

      # place 2 fingers on the left half, then click -> navigate back
      - type: click
        fingers: 2

        conditions:
          - $finger_1_position_percentage_x <= 0.5
          - $finger_2_position_percentage_x <= 0.5

        actions:
          - on: begin
            input:
              - mouse: [ back ]

      # move 3 fingers -> drag window
      - type: swipe
        fingers: 3
        direction: any
        resume_timeout: 500 # optional: allow lifting fingers for 500 ms

        actions:
          - on: begin
            input:
              - keyboard: [ +leftmeta ]
              - mouse: [ +left ]

          - on: update
            input:
              - mouse: [ move_by_delta ]

          - on: end_cancel
            input:
              - keyboard: [ -leftmeta ]
              - mouse: [ -left ]
  ```
</details>

## Repositories
| Name | Description |
|-|-|
| [ctl](https://github.com/InputActions/ctl) | The ``inputactions`` control tool |
| [installer](https://github.com/InputActions/installer) | Installation script |
| [wiki](https://github.com/InputActions/wiki) | Source for [wiki.inputactions.org](https://wiki.inputactions.org) |

### Implementations
| Name | Description |
|-|-|
| [hyprland](https://github.com/InputActions/hyprland) | Hyprland plugin implementation |
| [kwin](https://github.com/InputActions/kwin) | KWin (Plasma's compositor) plugin implementation |
| [standalone](https://github.com/InputActions/standalone) | Standalone implementation |

### Libraries
| Name | Description |
|-|-|
| [core](https://github.com/InputActions/core) | Contains all of InputActions' environment-independent code |
| [libevdev-cpp](https://github.com/InputActions/libevdev-cpp) | C++ wrapper for libevdev |
| [libinput-cpp](https://github.com/InputActions/libinput-cpp) | C++ wrapper for libinput |

## Contact
Users who do not wish to use GitHub may submit issues and patches by sending an e-mail to m``[at]``rcin``[dot]``dev (address subject to change).

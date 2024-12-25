# Конфиг файл для Home Assistant для Neptun Smart системы защиты от протечек
- работает по Modbus TCP c WiFi, Ethernet, (Tuya модуль нужно отключить на время подключения, потом можно вернуть если нужен).
- изменяем host в modbus на IP адресс своего Нептуна.
- добавлено 5 беспроводных датчиков, если нужно больше или меньше изменяем по примеру.
- заблокировал своему Neptun Smart полностью доступ в интернет на роутере. нет теперь никакого смысла в этом.
- сделано на основе [Карта памяти регистров модуля управления Neptun Smart для протокола MODBUS-RTU](https://s.siteapi.org/1b05f7bbad9a56b/docs/3jl34bgaunmsgcgog04ooccsswws4s)
- также основано на конфигах(config-wb-neptun.json) для WirenBoard от Ермухамедов Максим (MaxE). [Статья на sprut.ai](https://sprut.ai/article/neptun-v-umnyy-dom)

![image](https://github.com/user-attachments/assets/873a1922-a747-4c4b-8847-96f22fd79b56)
![image](https://github.com/user-attachments/assets/6225688e-f1ec-4cfa-89b2-94264520e880)


```
modbus:
  - name: neptun_smart
    type: tcp
    host: 192.168.1.198
    port: 503
    delay: 3
    timeout: 3

    sensors:
      - name: "Neptun Alarm and Mode Raw"
        address: 0
        slave: 240
        input_type: holding
        data_type: uint16
        scan_interval: 10
      - name: "Leak Sensor Raw"
        address: 3
        slave: 240
        input_type: holding
        data_type: uint16
        scan_interval: 10
      - name: "Number of Connected Wireless Sensors"
        address: 6
        slave: 240
        input_type: holding
        data_type: uint16
        unit_of_measurement: "шт."
        unique_id: "number_of_connected_wireless_sensors"
        scan_interval: 30
      - name: "Wireless Sensor 1 Raw"
        address: 57
        slave: 240
        input_type: holding
        data_type: uint16
        scan_interval: 10

      - name: "Wireless Sensor 2 Raw"
        address: 58
        slave: 240
        input_type: holding
        data_type: uint16
        scan_interval: 10

      - name: "Wireless Sensor 3 Raw"
        address: 59
        slave: 240
        input_type: holding
        data_type: uint16
        scan_interval: 10

      - name: "Wireless Sensor 4 Raw"
        address: 60
        slave: 240
        input_type: holding
        data_type: uint16
        scan_interval: 10

      - name: "Wireless Sensor 5 Raw"
        address: 61
        slave: 240
        input_type: holding
        data_type: uint16
        scan_interval: 10



# Template sensor для двоичного представления значения
template:
  sensor:
    - name: "Neptun Alarm and Mode Binary"
      state: "{{ '{:016b}'.format(states('sensor.neptun_alarm_and_mode_raw') | int) }}"  # Двоичное представление 16 бит

### Wireless sensors ####
    - name: "Battery Level Sensor 1"
      unit_of_measurement: "%"
      state: "{{ ((states('sensor.wireless_sensor_1_raw') | int) // (2 ** 8)) % 256 }}"  # Извлечение битов 8–15
      icon: mdi:battery

    - name: "Sensor Signal Level 1"
      unit_of_measurement: "%"
      state: "{{ (((states('sensor.wireless_sensor_1_raw') | int) // (2 ** 3)) % 8) * 25 }}"  # Извлечение битов 3–5 и умножение на 25 для масштабирования
      icon: mdi:signal

    - name: "Battery Level Sensor 2"
      unit_of_measurement: "%"
      state: "{{ ((states('sensor.wireless_sensor_2_raw') | int) // (2 ** 8)) % 256 }}"
      icon: mdi:battery

    - name: "Sensor Signal Level 2"
      unit_of_measurement: "%"
      state: "{{ (((states('sensor.wireless_sensor_2_raw') | int) // (2 ** 3)) % 8) * 25 }}"
      icon: mdi:signal

    # Новые сенсоры для Sensor 3
    - name: "Battery Level Sensor 3"
      unit_of_measurement: "%"
      state: "{{ ((states('sensor.wireless_sensor_3_raw') | int) // (2 ** 8)) % 256 }}"
      icon: mdi:battery

    - name: "Sensor Signal Level 3"
      unit_of_measurement: "%"
      state: "{{ (((states('sensor.wireless_sensor_3_raw') | int) // (2 ** 3)) % 8) * 25 }}"
      icon: mdi:signal

    # Новые сенсоры для Sensor 4
    - name: "Battery Level Sensor 4"
      unit_of_measurement: "%"
      state: "{{ ((states('sensor.wireless_sensor_4_raw') | int) // (2 ** 8)) % 256 }}"
      icon: mdi:battery

    - name: "Sensor Signal Level 4"
      unit_of_measurement: "%"
      state: "{{ (((states('sensor.wireless_sensor_4_raw') | int) // (2 ** 3)) % 8) * 25 }}"
      icon: mdi:signal

    # Новые сенсоры для Sensor 5
    - name: "Battery Level Sensor 5"
      unit_of_measurement: "%"
      state: "{{ ((states('sensor.wireless_sensor_5_raw') | int) // (2 ** 8)) % 256 }}"
      icon: mdi:battery

    - name: "Sensor Signal Level 5"
      unit_of_measurement: "%"
      state: "{{ (((states('sensor.wireless_sensor_5_raw') | int) // (2 ** 3)) % 8) * 25 }}"
      icon: mdi:signal
### END Wireless sensors ####


# Template binary sensors для отдельных битов
  binary_sensor:
    - name: "Floor Washing Mode"
      state: "{{ states('sensor.neptun_alarm_and_mode_binary')[15] == '1' }}"  # 0-й бит

    - name: "Alarm zona 1"
      state: "{{ states('sensor.neptun_alarm_and_mode_binary')[14] == '1' }}"  # 1-й бит

    - name: "Alarm zona 2"
      state: "{{ states('sensor.neptun_alarm_and_mode_binary')[13] == '1' }}"  # 2-й бит

    - name: "LeakSensor 1"
      state: "{{ states('sensor.leak_sensor_raw') | int | bitwise_and(1) > 0 }}"  # 0-й бит регистра 3

    - name: "Zona 1"
      state: "{{ states('sensor.neptun_alarm_and_mode_binary')[7] == '1' }}"  # 8-й бит

    - name: "Zona 2"
      state: "{{ states('sensor.neptun_alarm_and_mode_binary')[6] == '1' }}"  # 9-й бит

    - name: "Keypad Locks"
      state: "{{ states('sensor.neptun_alarm_and_mode_binary')[3] == '1' }}"  # 12-й бит

### Wireless sensors ####
    - name: "Presence of Alarm Sensor 1"
      state: "{{ (states('sensor.wireless_sensor_1_raw') | int) % 2 == 1 }}"  # Проверка 0-го бита
      icon: mdi:alarm
      
    - name: "Availability of Category Sensor 1"
      state: "{{ ((states('sensor.wireless_sensor_1_raw') | int) // (2 ** 1)) % 2 == 1 }}"
      icon: mdi:check-circle

    - name: "Sensor Loss Sensor 1"
      state: "{{ ((states('sensor.wireless_sensor_1_raw') | int) // (2 ** 2)) % 2 == 1 }}"
      icon: mdi:alert-circle

    - name: "Presence of Alarm Sensor 2"
      state: "{{ (states('sensor.wireless_sensor_2_raw') | int) % 2 == 1 }}"
      icon: mdi:alarm

    - name: "Availability of Category Sensor 2"
      state: "{{ ((states('sensor.wireless_sensor_2_raw') | int) // (2 ** 1)) % 2 == 1 }}"
      icon: mdi:check-circle

    - name: "Sensor Loss Sensor 2"
      state: "{{ ((states('sensor.wireless_sensor_2_raw') | int) // (2 ** 2)) % 2 == 1 }}"
      icon: mdi:alert-circle

    - name: "Presence of Alarm Sensor 3"
      state: "{{ (states('sensor.wireless_sensor_3_raw') | int) % 2 == 1 }}"
      icon: mdi:alarm

    - name: "Availability of Category Sensor 3"
      state: "{{ ((states('sensor.wireless_sensor_3_raw') | int) // (2 ** 1)) % 2 == 1 }}"
      icon: mdi:check-circle

    - name: "Sensor Loss Sensor 3"
      state: "{{ ((states('sensor.wireless_sensor_3_raw') | int) // (2 ** 2)) % 2 == 1 }}"
      icon: mdi:alert-circle

    - name: "Presence of Alarm Sensor 4"
      state: "{{ (states('sensor.wireless_sensor_4_raw') | int) % 2 == 1 }}"
      icon: mdi:alarm

    - name: "Availability of Category Sensor 4"
      state: "{{ ((states('sensor.wireless_sensor_4_raw') | int) // (2 ** 1)) % 2 == 1 }}"
      icon: mdi:check-circle

    - name: "Sensor Loss Sensor 4"
      state: "{{ ((states('sensor.wireless_sensor_4_raw') | int) // (2 ** 2)) % 2 == 1 }}"
      icon: mdi:alert-circle

    - name: "Presence of Alarm Sensor 5"
      state: "{{ (states('sensor.wireless_sensor_5_raw') | int) % 2 == 1 }}"
      icon: mdi:alarm

    - name: "Availability of Category Sensor 5"
      state: "{{ ((states('sensor.wireless_sensor_5_raw') | int) // (2 ** 1)) % 2 == 1 }}"
      icon: mdi:check-circle

    - name: "Sensor Loss Sensor 5"
      state: "{{ ((states('sensor.wireless_sensor_5_raw') | int) // (2 ** 2)) % 2 == 1 }}"
      icon: mdi:alert-circle

### END Wireless sensors ####


# Template binary_sensor для Dual Zone Mode
binary_sensor:
  - platform: template
    sensors:
      dual_zone_mode:
        friendly_name: "Dual Zone Mode"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(1024) > 0 }}"  # 10-й бит
      battery_drain_wireless_sensors:
        friendly_name: "Battery Drain in Wireless Sensors"
        unique_id: "battery_drain_wireless_sensors"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(8) > 0 }}"  # Проверка 3-го бита
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(8) > 0 %}
            mdi:battery-alert
          {% else %}
            mdi:battery
          {% endif %}

      lost_connection_wireless_sensors:
        friendly_name: "Lost Connection with Wireless Sensors"
        unique_id: "lost_connection_wireless_sensors"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(16) > 0 }}"  # Проверка 4-го бита
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(16) > 0 %}
            mdi:signal-off
          {% else %}
            mdi:signal
          {% endif %}




# Template switches для управления Zona 1, Zona 2, и Zona 1 + Zona 2 с доступностью для Zona 2
switch:
  - platform: template
    switches:
      zona_1_switch:
        friendly_name: "Zona 1 Switch"
        unique_id: "zona_1_switch"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(256) > 0 }}"
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(256) > 0 %}
            mdi:water-pump
          {% else %}
            mdi:water-pump-off
          {% endif %}
        turn_on:
          service: modbus.write_register
          data_template:
            hub: neptun_smart
            unit: 240
            address: 0
            value: >
              {% if is_state('binary_sensor.dual_zone_mode', 'off') %}
                {{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(768) }}
              {% else %}
                {{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(256) }}
              {% endif %}
        turn_off:
          service: modbus.write_register
          data_template:
            hub: neptun_smart
            unit: 240
            address: 0
            value: >
              {% if is_state('binary_sensor.dual_zone_mode', 'off') %}
                {{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(768)) }}
              {% else %}
                {{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(256)) }}
              {% endif %}


      zona_2_switch:
        friendly_name: "Zona 2 Switch"
        unique_id: "zona_2_switch"
        availability_template: "{{ is_state('binary_sensor.dual_zone_mode', 'on') }}"  # Доступен только если Dual Zone Mode включён
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(512) > 0 }}"
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(256) > 0 %}
            mdi:water-pump
          {% else %}
            mdi:water-pump-off
          {% endif %}
        turn_on:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(512) }}"
        turn_off:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(512)) }}"

      zona_1_2_switch:
        friendly_name: "Zona 1 + Zona 2 Switch"
        value_template: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(256)) > 0 and (states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(512)) > 0 }}"
        turn_on:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(768) }}"  # Устанавливаем 256 и 512 одновременно
        turn_off:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(768)) }}"  # Обнуляем 256 и 512

      dual_zone_mode_switch:
        friendly_name: "Dual Zone Mode Switch"
        unique_id: "dual_zone_mode_switch"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(1024) > 0 }}"  # Проверка 10-го бита
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(1024) > 0 %}
            mdi:tally-mark-2
          {% else %}
            mdi:tally-mark-1
          {% endif %}
        turn_on:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(1024) }}"  # Устанавливаем 10-й бит
        turn_off:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(1024)) }}"  # Обнуляем 10-й бит

      floor_washing_mode_switch:
        friendly_name: "Floor Washing Mode Switch"
        unique_id: "floor_washing_mode_switch"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(1) > 0 }}"  # Проверка 0-го бита
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(1) > 0 %}
            mdi:shower
          {% else %}
            mdi:shower-head
          {% endif %}
        turn_on:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(1) }}"  # Устанавливаем 0-й бит
        turn_off:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(1)) }}"  # Обнуляем 0-й бит

      keypad_locks_switch:
        friendly_name: "Keypad Locks Switch"
        unique_id: "keypad_locks_switch"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(4096) > 0 }}"  # Проверка 12-го бита
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(4096) > 0 %}
            mdi:keyboard-off
          {% else %}
            mdi:keyboard
          {% endif %}
        turn_on:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(4096) }}"  # Устанавливаем 12-й бит
        turn_off:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(4096)) }}"  # Обнуляем 12-й бит

      closing_taps_on_sensor_lost:
        friendly_name: "Closing Taps on Sensor Lost"
        unique_id: "closing_taps_on_sensor_lost_switch"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(2048) > 0 }}"  # Проверка 11-го бита
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(2048) > 0 %}
            mdi:water-off
          {% else %}
            mdi:water
          {% endif %}
        turn_on:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(2048) }}"  # Устанавливаем 11-й бит
        turn_off:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(2048)) }}"  # Обнуляем 11-й бит

      procedure_for_connecting_wireless_devices:
        friendly_name: "Procedure for Connecting Wireless Devices"
        unique_id: "procedure_for_connecting_wireless_devices_switch"
        value_template: "{{ states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(128) > 0 }}"  # Проверка 7-го бита
        icon_template: >
          {% if states('sensor.neptun_alarm_and_mode_raw') | int | bitwise_and(128) > 0 %}
            mdi:wifi
          {% else %}
            mdi:wifi-off
          {% endif %}
        turn_on:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_or(128) }}"  # Устанавливаем 7-й бит
        turn_off:
          service: modbus.write_register
          data:
            hub: neptun_smart
            unit: 240
            address: 0
            value: "{{ (states('sensor.neptun_alarm_and_mode_raw') | int) - ((states('sensor.neptun_alarm_and_mode_raw') | int) | bitwise_and(128)) }}"  # Обнуляем 7-й бит


```

В основном конфиге только 1 проводная линия. Если нужно добавить дополнительные проводные линии, берем отсюда и вставляем там где LeakSensor 1:
```
    - name: "LeakSensor 1"
      state: "{{ states('sensor.leak_sensor_raw') | int | bitwise_and(1) > 0 }}"  # Бит 0 регистра 3

    - name: "LeakSensor 2"
      state: "{{ states('sensor.leak_sensor_raw') | int | bitwise_and(2) > 0 }}"  # Бит 1 регистра 3

    - name: "LeakSensor 3"
      state: "{{ states('sensor.leak_sensor_raw') | int | bitwise_and(4) > 0 }}"  # Бит 2 регистра 3

    - name: "LeakSensor 4"
      state: "{{ states('sensor.leak_sensor_raw') | int | bitwise_and(8) > 0 }}"  # Бит 3 регистра 3
```

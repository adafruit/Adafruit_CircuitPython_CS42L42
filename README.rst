Introduction
============


.. image:: https://readthedocs.org/projects/adafruit-circuitpython-cs42l42/badge/?version=latest
    :target: https://docs.circuitpython.org/projects/cs42l42/en/latest/
    :alt: Documentation Status


.. image:: https://raw.githubusercontent.com/adafruit/Adafruit_CircuitPython_Bundle/main/badges/adafruit_discord.svg
    :target: https://adafru.it/discord
    :alt: Discord


.. image:: https://github.com/adafruit/Adafruit_CircuitPython_CS42L42/workflows/Build%20CI/badge.svg
    :target: https://github.com/adafruit/Adafruit_CircuitPython_CS42L42/actions
    :alt: Build Status


.. image:: https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json
    :target: https://github.com/astral-sh/ruff
    :alt: Code Style: Ruff

CircuitPython driver library for Cirrus CS42L42 audio codec


Dependencies
=============
This driver depends on:

* `Adafruit CircuitPython <https://github.com/adafruit/circuitpython>`_
* `Bus Device <https://github.com/adafruit/Adafruit_CircuitPython_BusDevice>`_
* `Register <https://github.com/adafruit/Adafruit_CircuitPython_Register>`_

Please ensure all dependencies are available on the CircuitPython filesystem.
This is easily achieved by downloading
`the Adafruit library and driver bundle <https://circuitpython.org/libraries>`_
or individual libraries can be installed using
`circup <https://github.com/adafruit/circup>`_.


Installing from PyPI
=====================

On supported GNU/Linux systems like the Raspberry Pi, you can install the driver locally `from
PyPI <https://pypi.org/project/adafruit-circuitpython-cs42l42/>`_.
To install for current user:

.. code-block:: shell

    pip3 install adafruit-circuitpython-cs42l42

To install system-wide (this may be required in some cases):

.. code-block:: shell

    sudo pip3 install adafruit-circuitpython-cs42l42

To install in a virtual environment in your current project:

.. code-block:: shell

    mkdir project-name && cd project-name
    python3 -m venv .venv
    source .env/bin/activate
    pip3 install adafruit-circuitpython-cs42l42

Installing to a Connected CircuitPython Device with Circup
==========================================================

Make sure that you have ``circup`` installed in your Python environment.
Install it with the following command if necessary:

.. code-block:: shell

    pip3 install circup

With ``circup`` installed and your CircuitPython device connected use the
following command to install:

.. code-block:: shell

    circup install adafruit_cs42l42

Or the following command to update an existing version:

.. code-block:: shell

    circup update

Usage Example
=============

.. code-block:: python

    import time

    import audiobusio
    import board
    import digitalio
    import synthio

    import adafruit_cs42l42

    RATE = 48000
    # CircuitPython sends 32 bit clocks per stereo frame, so the bit clock the
    # codec sees -- and the number configure_clocks wants -- is 32 x the rate.
    SCLK = RATE * 32

    # Mixer attenuation, dB. 0 is full scale; the quickstart starts at -20.
    DAC_VOLUME = -20

    # MIDI note numbers for a C-major scale, C4 up to C5.
    SCALE = (60, 62, 64, 65, 67, 69, 71, 72)

    # CAUTION: this drives a headphone amplifier. Take the headphones off before
    # the first run and confirm the level is comfortable before wearing them.

    oscillator = digitalio.DigitalInOut(board.OSC_EN)
    oscillator.direction = digitalio.Direction.OUTPUT
    oscillator.value = True

    reset = digitalio.DigitalInOut(board.CS42_RESET)
    reset.direction = digitalio.Direction.OUTPUT
    reset.value = False  # active low: hold the codec in reset while clocks start

    i2s = audiobusio.I2SOut(board.I2S_BIT_CLOCK, board.I2S_WORD_SELECT, board.I2S_DOUT)

    synth = synthio.Synthesizer(sample_rate=RATE)

    # Playing the synth starts the bit clock, which the codec needs before it can
    # switch off its own oscillator. The synth streams silence until a note is
    # pressed, so nothing is audible yet.
    i2s.play(synth)

    # Constructing the driver releases the reset
    codec = adafruit_cs42l42.CS42L42(board.I2C(), reset_pin=reset)
    codec.configure_clocks(sclk_hz=SCLK)
    codec.configure_asp(sample_rate=RATE, bit_depth=16)

    codec.headphone_output = True
    codec.dac_volume = DAC_VOLUME

    print("playing scale")
    try:
        while True:
            for note in SCALE:
                synth.press(note)
                time.sleep(0.25)
                synth.release(note)
                time.sleep(0.125)
    finally:
        codec.muted = True
        i2s.stop()
        i2s.deinit()
        reset.value = False
        reset.deinit()
        oscillator.value = False
        oscillator.deinit()


Documentation
=============
API documentation for this library can be found on `Read the Docs <https://docs.circuitpython.org/projects/cs42l42/en/latest/>`_.

For information on building library documentation, please check out
`this guide <https://learn.adafruit.com/creating-and-sharing-a-circuitpython-library/sharing-our-docs-on-readthedocs#sphinx-5-1>`_.

Contributing
============

Contributions are welcome! Please read our `Code of Conduct
<https://github.com/adafruit/Adafruit_CircuitPython_CS42L42/blob/HEAD/CODE_OF_CONDUCT.md>`_
before contributing to help this project stay welcoming.

![WS2812FX library](https://raw.githubusercontent.com/kitesurfer1404/WS2812FX/master/WS2812FX_logo.png)

WS2812FX - More Blinken for your LEDs!
======================================

This is port of the [WS2812FX](https://github.com/kitesurfer1404/WS2812FX) library for use with the Espressif IoT Development Framework (IDF).

This library features a variety of blinken effects for the WS2811/WS2812/NeoPixel LEDs. It is meant to be a drop-in replacement for the Adafruit NeoPixel library with additional features.

Features
--------

* 55 different effects. And counting.
* Tested on ESP32.
* All effects with printable names - easy to use in user interfaces.
* FX, speed and brightness controllable on the fly.
* Ready for sound-to-light (see external trigger example)


Download, Install and Example
-----------------------------

* Install the [Espressif IDF](https://www.espressif.com/en/products/sdks/esp-idf) (v5 or newer)
* Download this repository.
* Build and flash using the `idf.py` uiltity

In it's most simple form, here's the code to get you started!

```cpp
#include <WS2812FX.h>

extern "C" void app_main(void)
{
    led_strip_config_t strip_config = {
        .strip_gpio_num = DATA_PIN,
        .max_leds = NUM_LEDS,
        .led_model = LED_MODEL_WS2812,
        .color_component_format = LED_STRIP_COLOR_COMPONENT_FMT_GRB,
        .flags = {
            .invert_out = false,
        }
    };

    led_strip_rmt_config_t rmt_config = {
        .clk_src = RMT_CLK_SRC_DEFAULT,
        .resolution_hz = 10 * 1000 * 1000,
        .mem_block_symbols = 0, // Let the driver choose
        .flags = {
            .with_dma = false,
        }
    };

    led_strip_handle_t led_strip;
    ESP_ERROR_CHECK(led_strip_new_rmt_device(&strip_config, &rmt_config, &led_strip));

    WS2812FX *fx = new WS2812FX(NUM_LEDS, led_strip);
    fx->init();
    fx->setBrightness(30);
    fx->setSpeed(1000);
    fx->setMode(FX_MODE_RAINBOW_CYCLE);
    fx->start();

    while (true) {
        fx->service();
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}
```

More complex effects can be created by dividing your string of LEDs into segments (up to ten) and programming each segment independently. Use the **setSegment()** function to program each segment's mode, color, speed and direction (normal or reverse):
  * setSegment(segment index, start LED, stop LED, mode, color, speed, reverse);

Note, some effects make use of more than one color (up to three) and are programmed by specifying an array of colors:
  * setSegment(segment index, start LED, stop LED, mode, colors[], speed, reverse);

```cpp
// divide the string of LEDs into two independent segments
uint32_t colors[] = {RED, GREEN};
ws2812fx.setSegment(0, 0,           (LED_COUNT/2)-1, FX_MODE_BLINK, colors, 1000, false);
ws2812fx.setSegment(1, LED_COUNT/2, LED_COUNT-1,     FX_MODE_BLINK, COLORS(ORANGE, PURPLE), 1000, false);
```


Effects
-------

0. **Static** - No blinking. Just plain old static light.
1. **Blink** - Normal blinking. 50% on/off time.
2. **Breath** - Does the "standby-breathing" of well known i-Devices. Fixed Speed.
3. **Color Wipe** - Lights all LEDs after each other up. Then turns them in that order off. Repeat.
4. **Color Wipe Inverse** - Same as Color Wipe, except swaps on/off colors.
5. **Color Wipe Reverse** - Lights all LEDs after each other up. Then turns them in reverse order off. Repeat.
6. **Color Wipe Reverse Inverse** - Same as Color Wipe Reverse, except swaps on/off colors.
7. **Color Wipe Random** - Turns all LEDs after each other to a random color. Then starts over with another color.
8. **Random Color** - Lights all LEDs in one random color up. Then switches them to the next random color.
9. **Single Dynamic** - Lights every LED in a random color. Changes one random LED after the other to a random color.
10. **Multi Dynamic** - Lights every LED in a random color. Changes all LED at the same time to new random colors.
11. **Rainbow** - Cycles all LEDs at once through a rainbow.
12. **Rainbow Cycle** - Cycles a rainbow over the entire string of LEDs.
13. **Scan** - Runs a single pixel back and forth.
14. **Dual Scan** - Runs two pixel back and forth in opposite directions.
15. **Fade** - Fades the LEDs on and (almost) off again.
16. **Theater Chase** - Theatre-style crawling lights. Inspired by the Adafruit examples.
17. **Theater Chase Rainbow** - Theatre-style crawling lights with rainbow effect. Inspired by the Adafruit examples.
18. **Running Lights** - Running lights effect with smooth sine transition.
19. **Twinkle** - Blink several LEDs on, reset, repeat.
20. **Twinkle Random** - Blink several LEDs in random colors on, reset, repeat.
21. **Twinkle Fade** - Blink several LEDs on, fading out.
22. **Twinkle Fade Random** - Blink several LEDs in random colors on, fading out.
23. **Sparkle** - Blinks one LED at a time.
24. **Flash Sparkle** - Lights all LEDs in the selected color. Flashes single white pixels randomly.
25. **Hyper Sparkle** - Like flash sparkle. With more flash.
26. **Strobe** - Classic Strobe effect.
27. **Strobe Rainbow** - Classic Strobe effect. Cycling through the rainbow.
28. **Multi Strobe** - Strobe effect with different strobe count and pause, controlled by speed setting.
29. **Blink Rainbow** - Classic Blink effect. Cycling through the rainbow.
30. **Chase White** - Color running on white.
31. **Chase Color** - White running on color.
32. **Chase Random** - White running followed by random color.
33. **Chase Rainbow** - White running on rainbow.
34. **Chase Flash** - White flashes running on color.
35. **Chase Flash Random** - White flashes running, followed by random color.
36. **Chase Rainbow White** - Rainbow running on white.
37. **Chase Blackout** - Black running on color.
38. **Chase Blackout Rainbow** - Black running on rainbow.
39. **Color Sweep Random** - Random color introduced alternating from start and end of strip.
40. **Running Color** - Alternating color/white pixels running.
41. **Running Red Blue** - Alternating red/blue pixels running.
42. **Running Random** - Random colored pixels running.
43. **Larson Scanner** - K.I.T.T.
44. **Comet** - Firing comets from one end.
45. **Fireworks** - Firework sparks.
46. **Fireworks Random** - Random colored firework sparks.
47. **Merry Christmas** - Alternating green/red pixels running.
48. **Fire Flicker** - Fire flickering effect. Like in harsh wind.
49. **Fire Flicker (soft)** - Fire flickering effect. Runs slower/softer.
50. **Fire Flicker (intense)** - Fire flickering effect. More range of color.
51. **Circus Combustus** - Alternating white/red/black pixels running.
52. **Halloween** - Alternating orange/purple pixels running.
53. **Bicolor Chase** - Two LEDs running on a background color.
54. **Tricolor Chase** - Alternating three color pixels running.
55. **TwinkleFOX** - Lights fading in and out randomly.
56. thru 63. **Custom** - Up to eight user created custom effects.


Projects using WS2812FX
-----------------------

* [Smart Home project by renat2985](https://github.com/renat2985/rgb) using the ESP8266. Including a nice webinterface in the demo video!
* [WiFi LED Star by kitesurfer1404](http://www.kitesurfer1404.de/tech/led-star/en)
* [McLighting by toblum](https://github.com/toblum/McLighting) is a multi-client lighting gadget for ESP8266

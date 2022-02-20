# STM32F411 (blackpill) ws2812 ledstrip

This project is an example of running ws2812b ledstrip on STM32F411CE also know as blackpill, using Rust. Implementation uses SPI, so data line needs to be connected to pin A7. I'll also try to write down some general data, how to get it to run on microcontroller.

To get started with embedded rust, there's a really good book: [The Embedded Rust Book ](https://docs.rust-embedded.org/book/intro/index.html), I really recommend reading at least first few chapters as it describes how to get entire toolchain up and running. 

Code also relies on [stm32f4xx-hal](https://github.com/stm32-rs/stm32f4xx-hal) which is, as name implies, a HAL written in rust for stm32f4 series.

To work with ws2812 ledstrip, I used [smart-leds](https://github.com/smart-leds-rs/smart-leds) library, which uses [SPI driver](https://github.com/smart-leds-rs/ws2812-spi-rs). It's simple to use and works great.

## Important

To get this to work, you'll need to cross-compile the code and compile it with --release tag then load it with help of OpenOCD. There's also details about the entire project, that I will describe below. Most of this info is explained in more detail in above mentioned book and HAL repo.

### Toolchain

Everything about installation and details about tools are explained in [The Embedded Rust Book, section Tooling and Installation
](https://docs.rust-embedded.org/book/intro/tooling.html)Following is a quick overview.

This code needs to be cross-compiled to work on STM32 microcontroller, which is an Arm Cortex M4 with FPU core. More info about the chip itself can be found in [Reference manual PDF](https://www.st.com/resource/en/reference_manual/dm00119316-stm32f411xc-e-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf).


In order to cross-compile, you'll need to add target used for this microcontroller <b>thumbv7em-none-eabihf</b>, using following command
```
rustup target add thumbv7em-none-eabihf
```

As stated in the book, I also installed, cargo-binutils and cargo-generate.
More details can be found [here](https://docs.rust-embedded.org/book/intro/install.html).
On the same page you can also find OS-Specific instructions for installing following needed tools:
- GDB for Arm programs
- OpenOCD to work with ST-LINK
- ST-LINK usb driver

Once all of these are installed, you can very installation is working with following command:
```
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
```
Something along the following output should be produced:
```
...
Info : clock speed 2000 kHz
Info : STLINK V2J37S7 (API v2) VID:PID 0483:3748
Info : Target voltage: 3.240957
Info : stm32f4x.cpu: Cortex-M4 r0p1 processor detected
Info : stm32f4x.cpu: target has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f4x.cpu on 3333
Info : Listening on port 3333 for gdb connections
```
If it is not working, please refer to [following page](https://docs.rust-embedded.org/book/intro/install/verify.html).

### Project settings

For project to work correctly with HAL and given microcontroller, you'll also need <b>[.cargo/config](https://github.com/blaz-r/STM32F411-rust-neopixel/tree/main/.cargo/config)</b> and <b>[memory.x](https://github.com/blaz-r/STM32F411-rust-neopixel/tree/main/memory.x)</b> files. These are already present in this project.
- Config file is used to select the right target and in order for linker to work correctly.
- Memory.x is used to specify where RAM and FLASH are mapped on microcontroller. This info is currently set for blackpill with 128kB ram and 512kB flash. Info about this can be found in [above specified reference manual for stm32f411](https://www.st.com/resource/en/reference_manual/dm00119316-stm32f411xc-e-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf)  in "Memory and bus architecture" section page 42.

Since the project uses HAL and smart-led crates as stated above, they are listed in [Cargo.toml](https://github.com/blaz-r/STM32F411-rust-neopixel/tree/main/Cargo.toml) under dependencies. There's also some other dependencies needed, more information about these can be found in above mentioned book and HAL repo.

### Building project

With all properly configured, project can be simply built using following command
```
cargo build --release
```
-- release flag here was needed in my case, without this flag, the project didn't work correctly.

### Running project

To load and run the project on microcontroller, you'll need OpenOCD and GDB for Arm, which you can install as described above.

OpenOCD serves as an intermediate between GDB and ST-LINK and GDB itself is used for debugging and loading the code to microcontroller.

Following are the command I use on my windows setup, it should be very similar for linux:
- first run openOCD: 
    ```
    openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
    ```
- then run GDB. Following is command executed in root of project:
    ```
    arm-none-eabi-gdb -q .\target\thumbv7em-none-eabihf\release\stm32f411-rust-neopixel
    ```

This now opens GDB. It can be used to debug code, or in this case just load it and run the project.

First write te following to connect with openOCD:
```
target remote :3333
```

Then to load the project:
```
load
```

And at this point, if you just want to run it:
```
continue
```

Now the onboard led should turn on and the ledstrip should have a rainbow animation.


### Other

I hope that this project and guide helps anyone with running rust on blackpill. It is really not that complicated, since the whole process is nicely explained in the book. I'm also open to any suggestions about the code.

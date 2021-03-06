# I2C Library

The Onion I2C Library, `libonioni2c` is a static C library that provides functions to easily read from and write to devices communicating with the Omega via I2C. The library can be used in C and C++ programs.

Also available is a Python module that implements an I2C object using functions from the C library. The module is called `onionI2C` and is part of the `OmegaExpansion` package.


[[_TOC_]]


[//]: # (Linux and I2C)

# Linux and I2C

I2C devices are usually controlled by a kernel driver, however, it is also possible to access devices through an adapter in the Linux filesystem. The adapter can be found at `/dev/i2c-X`, where `X` is the adapter number which can range from 0 to 255. On the Omega, `/dev/i2c-0` is available by default. This allows users to interact with I2C slaves from the Linux environment.

The Onion I2C library uses the `/dev/i2c-0` adapter, and implements read and write functions I2C devices.



[//]: # (MAJOR HEADING)
[//]: # (The C Library)

# The C Library

The `libonioni2c` C library is a series of functions that implement I2C communication through the Linux device interface. 


[//]: # (Programming Flow)

## Programming Flow

Each of the read and write functions have been written to be self-contained, so one function call will complete the desired action.



[//]: # (Using the Library)

## Using the Library

**Header File**

To add the Onion I2C Library to your C/C++ program, include the header file in your C code:
``` c
#include <onion-i2c.h>
```

**Library for Linker**

In your project's makefile, you will need to add the following static libraries to the linker command:
``` c
-loniondebug -lonioni2c 
```

The static libraries are stored in `/usr/lib` on the Omega.



[//]: # (Example Code)

## Example

An example of how the `libonioni2c` library is used can be found in the implementation of the drivers for the [PWM, Relay, and OLED Expansions](https://github.com/OnionIoT/i2c-exp-driver)

Specifically, a variety of functions are used in the [PWM Expansion source code](https://github.com/OnionIoT/i2c-exp-driver/blob/master/src/pwm-exp.c).


[//]: # (Functions)

## Functions

Each of the main functions implemented in this library are described below.


[//]: # (Return Values)

### Return Values

All functions follow the same pattern with return values:

If the function operation is successful, the return value will be `EXIT_SUCCESS` which is a macro defined as `0` in `cstdlib.h.`

If the function operation is not successful, the function will return `EXIT_FAILURE` which is defined as `1`. 

A few reasons why the function might not be successful:
* The specified device address cannot be found on the I2C bus
* The system I2C interface is currently in use elsewhere

An error message will be printed that will give more information on the reason behind the failure.


[//]: # (Read Functions)

### Read Functions

Functions that perform reads from devices on the I2C bus


[//]: # (i2c_readByte)

#### Function: `i2c_readByte`

The `i2c_readByte` function will read one byte from a register address on a specified device on the I2C bus.

The function is declared as follows:
``` c
int i2c_readByte (int devNum, int devAddr, int addr, int *val);
```

**Arguments**

| Argument | Explanation               |
|----------|---------------------------|
| devNum   | Omega I2C adapter number  |
| devAddr  | I2C device address        |
| addr     | Device register to access |
| *val     | Returns: read value       |

The adapter number on the Omega should always be 0 since it will use `/dev/i2c-0` to interface with I2C devices. The I2C device address is unique to each type of device and can be found in the device datasheet. The datasheet will also define register addresses to read specific data. 

Finally, the val argument is passed by reference, and after the function runs will contain the single byte that was read.


**Examples**

To read a byte from the 0x01 register from an I2C device with address of 0x5a (the Servo Expansion):
``` c
int 	status, rdByte;
status 	= i2c_write(0, 0x5a, 0x01, &rdByte);
```


[//]: # (i2c_read)

#### Function: `i2c_read`

The `i2c_read` function will read a specified number of bytes from a register address on a device on the I2C bus.

The function is declared as follows:
``` c
int i2c_read (int devNum, int devAddr, int addr, uint8_t *buffer, int numBytes);
```

**Arguments**

| Argument | Explanation                              |
|----------|------------------------------------------|
| devNum   | Omega I2C adapter number                 |
| devAddr  | I2C device address                       |
| addr     | Device register to access                |
| *buffer  | Pointer that will contain the bytes read |
| numBytes | Number of bytes to read                  |

The adapter number on the Omega should always be 0 since it will use `/dev/i2c-0` to interface with I2C devices. The I2C device address is unique to each type of device and can be found in the device datasheet. The datasheet will also define register addresses to read specific data. 

The `numBytes` argument specifies how many bytes to read from a specific address on the I2C device. 
The `*buffer` argument is a unsigned 8-bit integer pointer that will contain all of the bytes read from the device once the function returns

Note that when using a pointer for the `buffer` argument, it must be allocated before being passed into the function, and it must be allocated with at least the number specified by `numBytes`. When using an array for the buffer, it must be declared with at least the number of elements specified by the numBytes argument.


**C Examples**

To read 4 bytes from address 0x00 on a device with an address of 0x48, using a pointer for the buffer:
``` c
int 	status;
uint8_t *buffer = malloc(4 * sizeof *buffer);
status 			= i2c_read(0, 0x48, 0x00, buffer, 4);
```

To read 2 bytes from address 0x10 on a device with an address of 0x48, using an array for the buffer:
``` c
int 	status;
uint8_t	buffer[32];
status 			= i2c_read(0, 0x48, 0x10, buffer, 2);
```

**C++ Examples**

To read 2 bytes from address 0x00 on a device with an address of 0x48, using a pointer for the buffer:
``` c++
int 	status;
uint8_t	*buffer = new uint8_t[16];
status 			= i2c_read(0, 0x48, 0x00, buffer, 2);
```


[//]: # (Write Functions)

### Write Functions

Functions that perform writes to devices on the I2C bus.


[//]: # (i2c_writeBuffer)

#### Function: `i2c_writeBuffer`

The `i2c_writeBuffer` function will write a specified number of bytes from a previously populated pointer or array to a register address on an I2C device.

The function is declared as follows:
``` c
int i2c_writeBuffer	(int devNum, int devAddr, int addr, uint8_t *buffer, int size);
```

**Arguments**

| Argument | Explanation                              |
|----------|------------------------------------------|
| devNum   | Omega I2C adapter number                 |
| devAddr  | I2C device address                       |
| addr     | Device register to access                |
| *buffer  | Bytes to be written to the device        |
| size     | Number of bytes to be written            |

The adapter number on the Omega should always be 0 since it will use `/dev/i2c-0` to interface with I2C devices. The I2C device address is unique to each type of device and can be found in the device datasheet. The datasheet will also define register addresses to read specific data. 

The `size` argument is the number of bytes to write from the buffer pointer. The `*buffer` argument is a unsigned 8-bit integer pointer that contains all of the bytes to be written to the specified address on the I2C device. Note that `buffer[0]` will be written first, and then `buffer[1]`, and so on.

When using a pointer for the `buffer` argument, it must be allocated and populated before being passed into the function, and it must be allocated with at least the number specified by `size`. When using an array for the buffer, it must be declared with at least the number of elements specified by the numBytes argument.


**Examples**

Write 2 bytes to the 0x02 register address on an I2C device with an address of 0x08, *using an array for the `buffer` argument*:
``` c
int 	status;
uint8_t	buffer[32];

// populate the buffer
buffer[0] 	= 0xde;
buffer[1] 	= 0xad;

status 		= i2c_writeBuffer(0, 0x08, 0x01, buffer, 2);
```

Write 3 bytes to the 0x05 register address on the same device as above using an array for the `buffer` argument:
``` c
int 	status;
uint8_t	buffer[3] = {0xbe, 0xef, 0x80};

status 		= i2c_writeBuffer(0, 0x08, 0x05, buffer, 3);
```

Write 4 bytes to the 0x54 register address on an I2C device with an address of 0x30, *using a pointer for the `buffer` argument:
``` c
int 	status;
uint8_t *buffer = malloc(4 * sizeof *buffer);

// populate the buffer
buffer[0] 	= 0xb6;
buffer[1] 	= 0xd1;
buffer[2] 	= 0xe3;
buffer[3] 	= 0xff;

status 		= i2c_writeBuffer(0, 0x30, 0x54, buffer, 4);
```



[//]: # (i2c_writeBytes)

#### Function: `i2c_writeBytes`

The `i2c_writeBytes` function will write a specified number of bytes from an integer variable to an address on an I2C device. Sometimes it's a little quicker to pass in an integer rather than create a buffer like the `i2c_writeBuffer` function above requires. Note that the Least Significant Byte (LSB) of the integer will be written first and that the maximum number of bytes is 4 (since an int holds 32 bits on the Omega).

The function is declared as follows:
``` c
int i2c_writeBytes 	(int devNum, int devAddr, int addr, int val, int numBytes);
```

**Arguments**

| Argument | Explanation                              |
|----------|------------------------------------------|
| devNum   | Omega I2C adapter number                 |
| devAddr  | I2C device address                       |
| addr     | Device register to access                |
| val      | Integer to be written                    |
| numBytes | Number of bytes to be written            |

The adapter number on the Omega should always be 0 since it will use `/dev/i2c-0` to interface with I2C devices. The I2C device address is unique to each type of device and can be found in the device datasheet. The datasheet will also define register addresses to read specific data. 

The `numBytes` argument is the number of bytes to write from the integer. The `val` argument is an integer variable that should be preprogrammed with a value to be written. 

Note that the bytes of the integer will be written in the following order, assuming that 4 bytes are being written:
* val[7:0]
* val[15:8]
* val[23:16]
* val[31:14]


**Examples**

Write a byte to the 0x02 register address on an I2C device with an address of 0x08:
``` c
int 	status;
int		val  	= 0x04;

status 		= i2c_writeBytes(0, 0x08, 0x01, val, 1);
```

Write 2 bytes to the 0x05 register address on the same device as above:
``` c
int 	status;
int		val  	= 0x1304;

// write 0x04 and then 0x13 to the 0x01 address
status 		= i2c_writeBytes(0, 0x08, 0x01, val, 2);

```

Write 3 bytes to the 0x54 register address on an I2C device with an address of 0x30:
``` c
int 	status;
int		val  	= 0xfe082324;

// write 0x24, then 0x23, and then 0x08 to the 0x54 address
status 		= i2c_writeBytes(0, 0x30, 0x54, val, 3);
```

Write 4 bytes to the 0x00 register address on an I2C device with an address of 0x30:
``` c
int 	status;
int		val  	= 0x27f8e460;

// write 0x60, then 0xe4, then 0xf8, and then 0x27 to the 0x00 address
status 		= i2c_writeBytes(0, 0x30, 0x00, val, 4);
```


[//]: # (Function Flow)

## Function Flow

All of the functions follow the same general pattern:
* Get a file descriptor to the I2C adapter
* Inform the adapter which device to communicate with
* Perform the function's main operation (read, write to the I2C adapter)
* Release the I2C adapter device handle
* Return the function status



[//]: # (MAJOR HEADING)
[//]: # (The Python Module)

# The Python Module

The `onionI2C` Python module in the `OmegaExpansion` package provides a Python object that serves as a wrapper around the C library functions. The usage is slightly different since the Python module is object oriented and the C library is just a set of functions.


[//]: # (Python: Programming Flow)

## Programming Flow

Once the I2C object is initialized, the read and write functions can be called freely using the object. 


[//]: # (Using the Python Module)

## Using the Python Module

**Installing the Module**

To install the Python module, run the following commands:
```
opkg update
opkg install python-light pyOnionI2C
```

This will install the module to `/usr/lib/python2.7/OmegaExpansion/`

*Note: this only has to be done once.*


**Using the Module**

To add the Onion I2C Module to your Python program, include the following in your code:
``` python
from OmegaExpansion import onionI2C
```

[//]: # (Example Code)

## Example

An example of how the `onionI2C` library is used can be found in the [`i2c-exp-driver` repo.](https://github.com/OnionIoT/i2c-exp-driver/blob/master/examples/onion-i2c.py)

The example code programs the Relay Expansion directly.


[//]: # (Functions)

## Functions

Each of the main functions implemented in this module are described below.


[//]: # (Initialization)

### Initialization

The object needs to be initialized before it can be used for reading and writing:
``` python
i2c 	= onionI2C.OnionI2C()
```

After this call, the object can be used for reading and writing.

**Arguments**

The constructor has an optional argument that defines the I2C device adapter number. This corresponds to the `X` in `/dev/i2c-X`.

If no argument is supplied, the adapter will be set to `/dev/i2c-0`. This is the only default adapter on the Omega and should suit most use cases.

If your use case requires a different adapter, add an integer argument to the constructor call.



[//]: # (Reading)

### Reading from an I2C Slave

#### Reading Bytes

This function reads a specified number of bytes from a specific device on the I2C bus, and returns them in a list:
``` python
valList = i2c.readBytes(devAddr, addr, size)
```

**Arguments**

The `devAddr` argument defines the I2C slave device address. 

The `addr` argument defines the address on the device from which to read.

The number of bytes to be read should be placed in the `size` argument.


**Examples**

Read 4 bytes from address `0x00` on a device with an address of `0x48`:
``` python
rdBytes   = i2c.readBytes(0x48, 0x00, 4)
```

Read a byte from address `0x24` on a device with an address of `0x27`:
``` python
byteList  = i2c.readBytes(0x27, 0x24, 1)
```
Note that even though only a single byte is being read, the variable `byteList` will be in the form of a list.


[//]: # (Writing)

### Writing to an I2C Slave

All writing functions share the same schema for return values:
* For a successful write, `0` will be returned
* An unsuccessful write will return `1`

[//]: # (Writing: Single Byte)

#### Write a Single Byte

This function will write a single byte to a specific device on the I2C bus:
``` python
status 	= i2c.writeByte(devAddr, addr, value)
```

**Arguments**

The `devAddr` argument defines the I2C slave device address. 

The `addr` argument defines the address on the device that will be written to.

The `value` argument is the single byte to be written.


**Examples**

Write `0xef` to address `0xf1` on a device with an address of `0x11`:
``` python
status  = i2c.writeByte(0x11, 0xf1, 0xef)
```

Write `0xbe` to address `0xaa` on a device with an address of` 0x33`:
``` python
status  = i2c.writeByte(0x33, 0xaa, 0xbe)
```


[//]: # (Writing: Multiple Bytes)

#### Write a List of Bytes

This function will write a list of bytes to an address on a specific device on the I2C bus:
``` python
status  = i2c.writeBytes(devAddr, addr, values)
```

**Arguments**

The `devAddr` argument defines the I2C slave device address. 

The `addr` argument defines the address on the device that will be written to.

The `values` argument is the list of bytes to be written.


**Examples**

Write `0xde, 0xad, 0xbe, 0xef` to address `0x1a` on a device with an address of `0x66`:
``` python
bytes   = [0xde, 0xad, 0xbe, 0xef]
status  = i2c.writeBytes(0x66, 0x1a, bytes)
```

Write `0xbe` to address `0xaa` on a device with an address of `0x33`:
``` python
status  = i2c.writeBytes(0x33, 0xaa, [0xbe])
```

Write `0x01, 0x03, 0x05` to address `0x55` on a device with an address of `0x24`:
``` python
status  = i2c.writeBytes(0x24, 0x55, [0x01, 0x03, 0x05])
```


[//]: # (Writing: Multiple Bytes w/ No Address)

#### Write a List of Bytes without Specifying an Address

This function will write a list of bytes to a specific device on the I2C bus:
``` python
status  = i2c.write(devAddr, values)
```

It can be used when no specific on address on the device needs to be specified.

**Arguments**

The `devAddr` argument defines the I2C slave device address. 

The `values` argument is the list of bytes to be written.


**Examples**

Write `0xde, 0xad, 0xbe, 0xef` to a device with an address of `0x67`:
``` python
bytes   = [0xde, 0xad, 0xbe, 0xef]
status  = i2c.write(0x67, bytes)
```

Write `0xaa, 0xbe` to a device with an address of `0x34`:
``` python
status  = i2c.write(0x34, [0xaa, 0xbe])
```

Write `0x01, 0x03, 0x05, 0x02, 0x04, 0x06, 0xaa` to a device with an address of `0x13`:
``` python
bytes   = [0x01, 0x03, 0x05, 0x02, 0x04, 0x06, 0xaa]
status  = i2c.write(0x13, bytes)
```


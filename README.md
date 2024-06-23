# BasicONFIDriver
A python bit-banging driver which can support ONFI storage chips, originally designed for the MT29F2G08ABAEAWP

## Usage
Example
```python
initialise()
// Max columnAddress is 2112 for the MT29F2G08ABAEAWP
eraseBlock([6 bit pageAddress (ignored)], [11 bit blockAddress])
programPage([12 bit columnAddress], [6 bit pageAddress], [11 bit blockAddress], [data in bytes])
readPage([12 bit columnAddress], [6 bit pageAddress], [11 bit blockAddress], [numOfBytesToRead])
readPageCacheSequential([12 bit columnAddress], [6 bit pageAddress], [11 bit blockAddress], [numOfBytesToRead], [numOfPagesToRead])

eraseBlock("000000", "00000000000")
programPageString("000000000000", "000000", "00000000000", "Hello World!")
readPageCacheSequential("000000000000", "000000", "00000000000", 2112, 8) //2112 bytes in a page for the MT29F2G08ABAEAWP
```

initialise() should be called at the start only

A typical read would use readPage(), if large amounts of data is utilised, readPageSequentional() should used
A typical write would use either programPage() or programPageString() depending on data type, make sure to eraseBlock() before writing data over an area with already written data

Use FlashStorage.py for demonstration of the main features

## Driver functions:
- Helper functions:
    - toByte(value) will convert a value to an 8 bit value of type string e.g. "00000000"
    - stringToByte(string) will convert a string to binary, having a length of a multiple of 8
    - verifyColumnAddress(columnAddress) and verifyAddress(columnAddress, pageAddress, blockAddress) are used to make sure the addresses to the chip are valid, returning false if not valid
- Raw operations
    - setPins() (ONFIDriverParallel.py only) will set the state of Pins to all be inputs, SHOULD be called before a readPins() command, preferable as early as possible after a writeCommand() or writeAddress()
    - readRaw() will read the values of the pins without toggling read enable, sometimes used to clean out the registers
    - readPins() will read the values of the pins, toggling read enable - Will actually get useful data
    - writePins(data) will write the 8 bit data to the pins, toggling write enable - This sends data to the NAND chip
    - writeCommand(data) will call writePins(data) but toggles command latch enable - This sends a command to the NAND chip
    - writeAddress(data) will call writePins(data) but toggles the address latch enable - This sends an address to the NAND chip
- Functions:
    - reset() will reset the chip, MUST be called before powerup
    - readID() will read the chip ID and ONFI ID of the chip, should print "ONFI" for the ONFI ID
    - readParameterPage(length) will return the data from the parameter page, a length is specified as the chip can theoretically return 999+ lines
    - setFeatures(featureAddress, value) will set the value to the feature, this is used to enable or disable ECC, WARMING: reserved bits are not verified in this command
    - getFeatrues(featureAddress) will return the value of the feature
    - status() will return 8 bits corresponding to the status of the chip
- Column address operations (Not really sure when to use these, however they are implemented)
    - columnRandReadC(columnAddress, length) will read data from the columnAddress at the current page and block, potentially used to quickly switch columns when reading multiple times
    - columnRandReadP(columnAddress, pageAddress, blockAddress, length) will read data from the specified addresses, not sure what the advantage is
    - columnRandInput(columnAddress, data) is supposed to write data to a columnAddress, NOT TESTED TO WORK
- Read operations
    - readMode() will set the chip to readMode(), sometimes required after going into status mode
    - readPage(columnAddress, pageAddress, blockAddress, length, ecc=False), will read data of length from a specified address, set ecc to True if ecc is turned on
    - readPageCacheSequential(columnAddress, pageAddress, blockAddress, length, pages, ecc=False, end=True) used to sequentially read data, it should be used to improve read speeds, by default, it will assume to only read the specified number of pages, but when end=False, can be used with readPageCacheSequentialContinued() to read larger data sequentially without running out of memory
    - readPageCacheSequentialContinued(length, pages, ecc=False, end=False) is used to continue to read sequential data
    - readPageCacheLast(length) will end the readSequential function, and also return data of the last specified page
- Program operations
    - programPage(columnAddress, pageAddress, blockAddress, data) will program a page at the address with the data in binary
    - programPageString(columnAddress, pageAddress, blockAddress, data) will achieve the same as programPage(), however it will accept a string, converting it to a binary value before writing it, this can be quicker than using programPage()
- Erase operations
    - eraseBlock(pageAddress, blockAddress) will erase the block specified by the blockAddress, pageAddress will be ignored by the NAND chip
- initialise() will enable the chip, resetting it, and set all control values to default ones

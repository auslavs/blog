---
title: "Code 128 Barcode Properties"
date: 2017-03-29
draft: true
---

### Determining x-value and quiet zones
You can use the following formula to determine the x Value of a barcode.

![formula for determining x-value](/images/formula.png "Formula for determining x-value")


| Variable | Description                                     | Value   |
| ---------| ------------------------------------------------| -------:|
| a        | Constant                                        |    25.4 |
| b        | The number of dots that make up a bar (integer) |    1..n |
| n        | The printer's dots per inch (dpi)               |  203/300|

When scanning a barcode using the barcode verifier you can deduce the dpi of the printer by using the above formula.

If the barcode verifier gives me an x-value of 334μm I can determine the true x-value by trialling different values of n

b = (25.4 * n) / 334 

b = (25.4 * 5) / 0.334 = 380.24 - 5 is not likely as it is not close to either 203 or 300 dpi.

b = (25.4 * 4) / 0.334 = 304.19 - Our number of dots is likely to be 4 as this is close to 300 dpi.

 

Therefore, the true x-value will be:

(25.4 / 300) * 4 = 0.339

#### Calculating the quiet zone
The quiet zone for the barcode is to be 10 times the x-value, so 10(0.339) = 3.39mm


### Calculating the width of a Code 128 Barcode

The following formulas can be used to calculate the width of a Code 128 barcode excluding quiet zones.

**Code128 Subset A/B**

![XXXXX](/images/formula3.png "XXXXX")

**Code128 Subset C**

![XXXXX](/images/formula3.png "XXXXX")


| Variable | Description                                     |
| ---------| ------------------------------------------------|
| w        | Overall barcode width (in mm)                   |
| c        | Number of characters being encoded              |
| x        | x-value (in mm)                                 |

**Note: The above formulas do not apply to barcodes that comprise of multiple subsets.**
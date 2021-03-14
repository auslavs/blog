---
title: "Code 128 Barcode Properties"
date: 2017-03-29
draft: false
---

### Determining the X-Dimension and quiet zones

The X-Dimension is the narrowest bar or whitespace in a barcode.
You can use the following formula to determine the X-Dimension of a barcode.

![Formula for determining X-Dimension](/images/formula.png "Formula for determining X-Dimension")


| Variable | Description                                     | Value   |
| ---------| ------------------------------------------------| -------:|
| a        | Constant                                        |    25.4 |
| b        | The number of dots that make up a bar (integer) |    1..n |
| n        | The printer's dots per inch (dpi)               |  203/300|

When scanning a barcode using a barcode verifier, you can deduce the label printer's dpi using the above formula.

For example, if the barcode verifier gives me an X-Dimension of 334μm, I can determine the what was the intended X-Dimension by trialing different values of n:

b = (25.4 * n) / 334 

b = (25.4 * 5) / 0.334 = 380.24 - 5 is not likely as it is not close to either 203 or 300 dpi.

b = (25.4 * 4) / 0.334 = 304.19 - Our number of dots is likely to be 4 as this is close to 300 dpi.

 
Therefore, the true X-Dimension will be:

(25.4 / 300) * 4 = 0.339

#### Calculating the quiet zone
The quiet zone for the barcode is to be 10 times the X-Dimension, so 10(0.339) = 3.39mm


### Calculating the width of a Code 128 Barcode

The following formulas can be used to calculate the width of a Code 128 barcode excluding quiet zones.

**Code128 Subset A/B**

![Formula to calculate the width of a Code128 Subset A/B barcode](/images/formula3.png "Formula to calculate the width of a Code128 Subset A/B barcode")

**Code128 Subset C**

![Formula to calculate the width of a Code128 Subset C barcode](/images/formula3.png "Formula to calculate the width of a Code128 Subset A/B barcode")


| Variable | Description                                     |
| ---------| ------------------------------------------------|
| w        | Overall barcode width (in mm)                   |
| c        | Number of characters being encoded              |
| x        | X-Dimension (in mm)                                 |

**Note: The above formulas do not apply to barcodes that comprise of multiple subsets.**
---
title: "ABN Generator"
date: 2023-12-30
draft: true
tags: ['dotnet','fsharp']
---

Following on from my [Tax File Number (TFN) Generator](./tfn-generator/) post, the [Australian Business Number (ABN)](https://en.wikipedia.org/wiki/Australian_Business_Number) is another number that I need to generate when writing and testing financial software.

The ABN is a unique identifier issued by the Australian Business Register (ABR), operated by the Australian Taxation Office (ATO). It consists of an eleven-digit number; like the TFN, it contains a checksum.
However, this time, the checksum is in the first two digits is the checksum.

The below tool generates random ABN with a valid checksum.
Once you have clicked on the *Generate ABN* button, click the TFN to copy it to your clipboard.

Here is also a link to a standalone version of the [ABN Generator](/abn-generator.html).

{{< abn-generator >}}

The algorithm for generating the checksum of a TFN involves multiplying each of the first eight digits by a specific weight and then summing these products. The weights for the eight digits are 1, 4, 3, 7, 5, 8, 6, and 9, respectively. The sum is then divided by 11, and the remainder determines the final digit.

There are more details on [wikipedia](https://en.wikipedia.org/wiki/Tax_file_number) if you are interested.

## Implementing the TFN Generator Algorithm in F#

1. Start by Creating a Function to Generate Random Digits:

```fsharp
let randomDigits count =
  let rnd = System.Random()
  List.init count (fun _ -> rnd.Next(0, 10))

```

2. Define the Weights and the Checksum Calculation:

```fsharp
let weights = [1; 4; 3; 7; 5; 8; 6; 9]

let calculateChecksum digits =
  let sum = List.map2 (*) digits weights |> List.sum
  sum % 11

```

3. Generate the TFN:

```fsharp
// Warning there is a bug here
let generateTFN () =
  let digits = randomDigits 8
  let checksum = calculateChecksum digits
  digits @ [checksum]
  |> List.map string
  |> String.concat ""
```

I ruined the surprise, but I didn't want to trip anyone up. The above code will generate a TFN most of the time, however the checksum could potentially be equal to 10, making our 9 digit TFN 10 digits, and therefore not valid.

This is what prompted me to build my own tool and write this post as I found more than one online tool with this issue.

So to work around this we can simply keep generating random a digits until we find one thats checksum is only one digit.
The final code looks like this.

```fsharp
let randomDigits count =
  let rnd = System.Random()
  List.init count (fun _ -> rnd.Next(0, 10))

let weights = [1; 4; 3; 7; 5; 8; 6; 9]

let calculateChecksum digits =
  let sum = List.map2 (*) digits weights |> List.sum
  sum % 11

let generateTfn () =
  let rec generate () =
    let digits = randomDigits 8
    match calculateChecksum digits with
    | cs when cs <= 9 -> digits @ [cs]
    | _ -> generate ()
  generate () |> List.map string |> String.concat ""

generateTfn ()
```

Note that this is just a tool for testing, the ATO have never released there methodology for generating a TFN.
Real TFNs are issued exclusively by the ATO and should not be generated or used for any official purposes.

Just as a side note, the TFN Generator code embedded in this page was completely written in F# using [Feliz](https://zaid-ajaj.github.io/Feliz/).

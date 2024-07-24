Here’s a comprehensive GitHub README for your Pine Script. This README provides a detailed overview of the script’s purpose, functionality, inputs, and usage instructions.

---

# Flag Pattern Detection Indicator

## Overview

The "Flag Pattern Detection Indicator" is a Pine Script designed for use with TradingView. This script identifies flag patterns in trading charts and calculates potential target prices based on detected patterns. Flag patterns are chart formations that suggest the continuation of the current trend. This script is particularly useful for traders looking to automate the identification of such patterns.

## Features

- **Pivot Detection**: Identifies high and low pivots in the price chart.
- **Pattern Matching**: Validates flag patterns based on length ratios and coefficients.
- **Target Calculation**: Computes and displays potential price targets when a flag pattern is detected.
- **Customizable Inputs**: Allows users to adjust parameters to suit their analysis needs.

## Inputs

The script includes several user-configurable inputs:

- `period` (integer): Number of candles to consider for detecting pivots. Default is `45`.
- `k1` (float): Coefficient to compare the length of the flagpole and flag. Default is `1.5`.
- `k2` (float): Coefficient for checking the length of the flag relative to the flagpole. Default is `0.25`.

## How It Works

### 1. **Pivot Detection**

The script detects significant high and low pivots within a specified period. These pivots are used to assess potential flag patterns.

### 2. **Pattern Matching**

The script compares the lengths of various segments (flagpole and flag) to determine if they match the criteria for a flag pattern. It uses coefficients `k1` and `k2` to validate the pattern.

### 3. **Target Calculation**

When a valid flag pattern is detected, the script calculates a potential target price based on the flagpole length and the detected pattern. This target price is displayed on the chart.

### 4. **Visualization**

The script plots detected pivots and potential target levels on the chart. It uses labels and lines to indicate the identified patterns and target prices.

## Pine Script Code

```pinescript
//@version=5
indicator("final project flag pattern mahrad mozafari", overlay=true)

// Inputs
period = input.int(45, "Period Check of Last Candles", 1)
k1 = input.float(1.5, "Coefficient (#0=>#1).len > k1 * (#1=>#2).len")
k2 = input.float(0.25, "Coefficient (#1=>#2)-(#3=>#4).len < k2 * (#1=>#2).len")

// Variables
countup = 0  // Number of high pivots
countdown = 0  // Number of low pivots
temp = close * 0  // Placeholder variable
maxs = array.new_float(0)  // Array for high pivots
maxsindex = array.new_int(0)  // Index array for high pivots
mins = array.new_float(0)  // Array for low pivots
minsindex = array.new_int(0)  // Index array for low pivots
minmax = array.new_int(1, 0)  // Array to track pivot types

// Check Pattern Function
CheckPattern(temp1, temp2, leg, target, dir, tempindex) =>
    if (array.size(temp1) == 3) and (array.size(temp2) == 4)
        array.push(leg, math.abs(array.get(temp1, 0) - array.get(temp2, 0)))
        array.push(leg, math.abs(array.get(temp1, 0) - array.get(temp2, 1)))
        if array.get(leg, 0) >= (1 / k1) * array.get(leg, 1)
            array.push(leg, math.abs(array.get(temp1, 1) - array.get(temp2, 2)))
            if math.abs(array.get(leg, 1) - array.get(leg, 2)) <= (k2) * array.get(leg, 1)
                array.push(leg, math.abs(array.get(temp1, 2) - array.get(temp2, 3)))
                if math.abs(array.get(leg, 1) - array.get(leg, 3)) <= (k2) * array.get(leg, 1)
                    target_price = array.get(temp2, 3) + (-1 * dir) * math.abs(array.get(temp2, 0) - array.get(temp1, 0))
                    array.push(target, target_price)
                    label.new(bar_index[array.get(tempindex, 2)], y=target_price, textcolor=color.white, color=(dir == 1 ? color.maroon : color.green), style=(dir == 1 ? label.style_label_upper_right : label.style_label_lower_left), text="Target: " + str.tostring(target_price))
                    line.new(bar_index[array.get(tempindex, 2)], dir == 1 ? high[array.get(tempindex, 2)] : low[array.get(tempindex, 2)], bar_index[array.get(tempindex, 2) - 10], target_price, extend=extend.none, width=3, style=line.style_arrow_right, color=(dir == 1 ? color.rgb(1226, 121, 135, 50) : color.rgb(38, 168, 150, 50)))
                    array.pop(leg, 3)
            array.pop(leg, 2)
        array.pop(leg, 1)
        array.pop(leg, 0)

// Find Pivots Function
FindPivots(whatExterm, whatIndexExterm, whatHighLow, i, d) =>
    if array.get(minmax, array.size(minmax) - 1) == d
        array.pop(whatIndexExterm)
        last_value = array.get(whatExterm, array.size(whatExterm) - 1)
        new_value = (last_value > whatHighLow * d ? last_value : whatHighLow)
        array.push(whatExterm, new_value)
        array.push(whatIndexExterm, period - i)
        if (d == 1 ? array.size(mins) : array.size(maxs)) == 3 and (d == 1 ? array.size(maxs) : array.size(mins)) == 4
            CheckPattern((d == 1 ? maxs : mins), (d == 1 ? mins : maxs), leg, target, d, (d == 1 ? minsindex : maxsindex))
    else
        array.push(whatExterm, whatHighLow)
        array.push(whatIndexExterm, period - i)
        array.push(minmax, d)
        if (d == 1 ? array.size(mins) : array.size(maxs)) == 3 and (d == 1 ? array.size(maxs) : array.size(mins)) == 4
            CheckPattern((d == -1 ? maxs : mins), (d == -1 ? mins : maxs), leg, target, d, (d == -1 ? minsindex : maxsindex))
            array.reverse(d == 1 ? mins : maxs)
            array.pop(d == 1 ? mins : maxs)
            array.reverse(d == 1 ? mins : maxs)

// Main Logic
if bar_index >= period
    for i = 0 to period
        if high[period - i] <= high[period - i + 1] and high[period - i + 1] >= high[period - i + 2]
            countup := countup + 1
            FindPivots(maxs, maxsindex, high[period - i + 1], i, 1)
        if low[period - i] >= low[period - i + 1] and low[period - i + 1] <= low[period - i + 2]
            countdown := countdown + 1
            FindPivots(mins, minsindex, low[period - i + 1], i, -1)

plot(countup, title="High Pivots Count", color=color.blue)
plot(countdown, title="Low Pivots Count", color=color.red)
```

## Usage Instructions

1. **Add to TradingView**:
   - Copy the Pine Script code provided above.
   - Open TradingView, go to the Pine Editor, and create a new indicator.
   - Paste the code into the editor and save the indicator.

2. **Adjust Parameters**:
   - Modify the `period`, `k1`, and `k2` inputs to fit your trading strategy and analysis needs.

3. **Apply to Chart**:
   - Add the indicator to your chart to visualize detected flag patterns and potential target prices.

## License

This script is licensed under the [Mozilla Public License 2.0](https://mozilla.org/MPL/2.0/).

---

This README provides a detailed description of the script, including its purpose, how it works, and instructions for usage. You can further customize the text according to your specific needs or preferences.

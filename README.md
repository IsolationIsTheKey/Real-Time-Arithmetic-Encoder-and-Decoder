# Project Overview: Real-Time Arithmetic Encoder and Decoder

## Objective
This project implements a real-time arithmetic encoder and decoder on the Blackfin BF533 processor. The goal is to translate a baseline Python code into C and then optimize it to improve performance. The project compares performance in terms of clock cycles required for encoding and decoding a specified input sequence, **"ABBACDDCABAB"**.

---

## Introduction to Real-Time Arithmetic Coding and Decoding

A real-time arithmetic encoder and decoder compresses and decompresses sequences of symbols based on predefined probabilities for each symbol. The encoding calculates a single value representing the entire sequence, which is decoded to retrieve the original data. Performance optimization is critical in real-time applications to reduce clock cycles without compromising accuracy.

---

## Implementation

### Original Python Code for Arithmetic Encoding and Decoding

This Python code is used as a baseline for the arithmetic encoding and decoding process. It is translated to C for performance comparison and optimization.

```python
class Probability:
    def __init__(self, symbol, low, high):
        self.symbol = symbol
        self.low = low
        self.high = high

def arithmetic_encode(symbols, probabilities):
    low = 0.0
    high = 1.0

    for symbol in symbols:
        range_ = high - low
        for prob in probabilities:
            if prob.symbol == symbol:
                high = low + range_ * prob.high
                low = low + range_ * prob.low
                break

    return low

def arithmetic_decode(value, probabilities, length):
    decoded_symbols = []
    for _ in range(length):
        range_ = 1.0
        for prob in probabilities:
            mid = range_ * prob.high
            if value < mid:
                decoded_symbols.append(prob.symbol)
                value = (value - range_ * prob.low) / (mid - range_ * prob.low)
                break
    return ''.join(decoded_symbols)

# Example Usage
probabilities = [
    Probability('A', 0.0, 0.3),
    Probability('B', 0.3, 0.5),
    Probability('C', 0.5, 0.7),
    Probability('D', 0.7, 1.0)
]

encoded_value = arithmetic_encode("ABBACDDCABAB", probabilities)
decoded_sequence = arithmetic_decode(encoded_value, probabilities, 12)

```C

#include <stdio.h>

typedef struct {
    char symbol;
    float low;
    float high;
} Probability;

float arithmetic_encode_optimized(const char* symbols, Probability* probabilities, int length) {
    float low = 0.0f;
    float high = 1.0f;
    for (int i = 0; i < length; i++) {
        float range = high - low;
        switch (symbols[i]) {
            case 'A':
                high = low + range * probabilities[0].high;
                low = low + range * probabilities[0].low;
                break;
            case 'B':
                high = low + range * probabilities[1].high;
                low = low + range * probabilities[1].low;
                break;
            case 'C':
                high = low + range * probabilities[2].high;
                low = low + range * probabilities[2].low;
                break;
            case 'D':
                high = low + range * probabilities[3].high;
                low = low + range * probabilities[3].low;
                break;
        }
    }
    return low;
}

void arithmetic_decode_optimized(float value, Probability* probabilities, int length, char* decoded_symbols) {
    for (int i = 0; i < length; i++) {
        float range = 1.0f;
        switch (value < probabilities[1].high ? 'A' : value < probabilities[2].high ? 'B' : value < probabilities[3].high ? 'C' : 'D') {
            case 'A':
                decoded_symbols[i] = 'A';
                value = (value - range * probabilities[0].low) / (range * (probabilities[0].high - probabilities[0].low));
                break;
            case 'B':
                decoded_symbols[i] = 'B';
                value = (value - range * probabilities[1].low) / (range * (probabilities[1].high - probabilities[1].low));
                break;
            case 'C':
                decoded_symbols[i] = 'C';
                value = (value - range * probabilities[2].low) / (range * (probabilities[2].high - probabilities[2].low));
                break;
            case 'D':
                decoded_symbols[i] = 'D';
                value = (value - range * probabilities[3].low) / (range * (probabilities[3].high - probabilities[3].low));
                break;
        }
    }
    decoded_symbols[length] = '\0';
}


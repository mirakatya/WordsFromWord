# Words from Word Generator

## Project Description
This Java application implements the "Words from Word" algorithm. Based on a given dictionary and a list of main words, it generates a structured JSON of levels. Each level contains the main word, a set of valid subwords, the calculated difficulty, and the minimum number of subwords required to pass the level.

## Features
*   Reads input data from files.
*   Filters subwords based on `minSubWordLength` from the config.
*   Handles letter multiplicity (e.g., if a word has one 'Я', you cannot make a subword with two).
*   Handles the letter 'Ё' by folding it into 'Е'.
*   Automatically calculates the level difficulty (`EASY`, `MEDIUM`, `HARD`).
*   Validates input (skips levels if there are not enough valid subwords).

## Division of Work
The code is divided into logical blocks using author comments.

**Mira**
*   Implemented the `Config` parsing logic.
*   Developed File I/O operations (`readWords` method) and the `main` execution flow.

**Katya**
*   Developed the core generation algorithm (`generate` method).
*   Implemented word normalization and letter count filtering.

**Veronika**
*   Created data models (`Level` record).
*   Implemented manual JSON serialization (`toJson` and `esc` methods).

## How to Run
1. Open the project in IntelliJ IDEA.
2. Run the `WordsFromWord` class.
3. Provide arguments: `dict.txt mains.txt config.json`
23:20
Сегодня

# AI Usage & Prompt History

For this project, our team used the Claude AI model as an assistant to generate the initial boilerplate code based on the assignment requirements. Below is the history of our main prompts and the summary of the responses.

## Interaction 1: Initial Code Generation

**Prompt:**
> "Напиши программу на Java (с использованием Java Collections Framework). Нужно реализовать алгоритм "Words from word". На вход подается словарь (массив русских слов) и список главных слов. Нужно сгенерировать структуру уровней в JSON. Уровень состоит из главного слова, набора подслов, сложности и минимального количества подслов для прохождения.
> Учти конфиг: minSubWordLength, minWordsToPass, difficultyThresholds (easyMaxMainWordLength, mediumMaxMainWordLength, easyMaxSubWords, mediumMaxSubWords).
> Правила: подслово можно составить из букв главного слова с учетом кратности (если в главном слове одна 'Я', то две нельзя). Буква 'Ё' приравнивается к 'Е'. Словарь может содержать дубликаты."

**Response Summary:**
The AI generated the initial `WordsFromWord.java` class. It included:
*   A `Config` record for parsing JSON thresholds.
*   A method `norm()` to handle case-insensitivity and the 'Ё' to 'Е' replacement.
*   A method `counts()` to calculate letter frequencies.
*   The core `generate()` method with logic to filter subwords.
*   A manual `toJson()` method to format the output.

## Interaction 2: Refining the Code and Splitting Work

**Prompt:**
> "Как нам разделить этот код между тремя участниками команды, чтобы каждый отвечал за свою логическую часть (чтение файлов, алгоритм, генерация JSON)? Добавь комментарии для разделения."

**Response Summary:**
The AI suggested dividing the code into three logical blocks:
1. Configuration and File I/O.
2. Core Algorithm and Filtering.
3. Data Models and JSON Serialization.
We then manually added author comments (`// Author: Name`) to these blocks as suggested.

## Interaction 3: Fixing Edge Cases

**Prompt:**
> "Как обработать ситуацию, если в словаре нет подходящих подслов? Программа не должна падать."

**Response Summary:**
The AI suggested adding a check using `cfg.minWordsToPass()`. If the `subs` list size is smaller than the threshold, the program prints a warning and skips adding this level, preventing runtime errors.

*Note: The code provided by the AI was manually reviewed, formatted, and integrated by the team.*

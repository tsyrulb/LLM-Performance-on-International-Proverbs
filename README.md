# Understanding Large Language Models' Performance on International Proverbs

## Introduction
Large Language Models (LLMs) have shown impressive capabilities in language understanding, transformation, and generation. However, they still exhibit limitations. This project explores LLMs' performance on interpreting international proverbs within dialogues, a linguistic task where nuanced understanding is critical. I aim to identify the extent to which LLMs struggle with this task, quantify their performance, and suggest potential mitigation strategies.

## Linguistic Phenomenon

### Description and Definition
The phenomenon investigated is the interpretation of international proverbs within dialogues. A proverb (from Latin: proverbium) or an adage is a simple, traditional saying that expresses a perceived truth based on common sense or experience. Proverbs are often metaphorical and are an example of formulaic language. A proverbial phrase or a proverbial expression is a type of conventional saying similar to proverbs and transmitted by oral tradition. The difference is that a proverb is a fixed expression, while a proverbial phrase permits alterations to fit the grammar of the context. Collectively, they form a genre of folklore.

Proverbs pose unique challenges for LLMs due to their metaphorical nature and reliance on cultural context. Understanding proverbs requires not only linguistic competence but also cultural knowledge, as these sayings often reflect values, beliefs, and common experiences specific to a particular culture. LLMs may struggle with:
- **Metaphorical Interpretation**: Proverbs often convey meanings beyond their literal words, which can confuse models that rely heavily on word-to-word translation.
- **Cultural Context**: Many proverbs are deeply rooted in specific cultural contexts, and LLMs may lack the cultural knowledge needed to interpret them correctly.
- **Formulaic Language**: The fixed nature of proverbs and their variations in proverbial phrases can lead to difficulties in understanding and generating appropriate responses.

These challenges highlight the need for advanced language understanding and cultural embedding within LLMs to accurately interpret and respond to proverbs within dialogues.

## Data Description
The data file contains 200 samples of dialogues with different proverbs, 4 possible answers for each dialogue, and the correct answer. For example:

| Dialogue | Answers | Correct Answer |
| -------- | ------- | -------------- |
| • “Why did you pack the flashlight? We’re only going during the day.” • “When you are full, bring food. When it is sunny, bring an umbrella.” | "a) The speaker believes it will get dark during their trip, so they brought a flashlight. b) The speaker brought a flashlight because they expect to need it in case of an emergency. c) The speaker is prepared for any unexpected situations that may arise. d) The speaker didn’t bring a flashlight because they will be traveling during the day." | c |
| • “Why did you bring extra money? We have enough for the tickets.” • “When you are full, bring food. When it is sunny, bring an umbrella.” | "a) The speaker expects to find extra expenses on the trip, so they brought extra money. b) The speaker brought extra money because they plan to buy souvenirs. c) The speaker is prepared for any unexpected expenses. d) The speaker didn’t bring extra money because they have enough for the tickets." | c |

## Notebook Description

The included Jupyter Notebook contains the following steps and code:

1. **Library Installation**:
    ```python
    !pip install pandas openpyxl
    !pip install requests
    ```

2. **Imports and Data Loading**:
    ```python
    import pandas as pd
    import requests
    import time
    import re

    file_path = '/content/data.xlsx'
    df = pd.read_excel(file_path)
    data_list = [tuple(row) for row in df.values]
    ```

3. **Data Formatting**:
    ```python
    formatted_list_with_answers = []
    for dialogue, answers, correct_answer in data_list:
        formatted_string = (
            "Part of the dialogue is written here. You need to analyze it and choose one of the four options listed below, "
            "which you consider to be the most correct based on the dialogue.\n\n"
            f"{dialogue}\n\n"
            "Choose one correct option out of four:\n"
            f"{answers}"
        )
        formatted_list_with_answers.append((formatted_string, correct_answer))
    ```

4. **API Request Setup**:
    ```python
    TOGETHER_API_KEY = ""
    def send_prompt_to_together(prompt):
        url = "https://api.together.xyz/v1/chat/completions"
        headers = {
            "Authorization": f"Bearer {TOGETHER_API_KEY}",
            "Content-Type": "application/json"
        }
        data = {
            "model": "meta-llama/Llama-2-13b-chat-hf",
            "messages": [{"role": "user", "content": prompt}],
            "temperature": 0.7,
            "max_tokens": 50
        }

        response = requests.post(url, headers=headers, json=data)
        if response.status_code == 200:
            return response.json()
        else:
            print(f"Error: {response.status_code}, {response.text}")

        return None
    ```

5. **Sending Prompts and Collecting Responses**:
    ```python
    response_list = []
    delay_between_requests = 1.5

    for formatted_string, correct_answer in formatted_list_with_answers:
        response = send_prompt_to_together(formatted_string)
        if response:
            choices = response.get('choices', [])
            if choices:
                response_message = choices[0]['message']['content']
                response_list.append((formatted_string, response_message, correct_answer))
            else:
                response_list.append((formatted_string, None, correct_answer))
        else:
            response_list.append((formatted_string, None, correct_answer))

        time.sleep(delay_between_requests)
    ```

6. **Response Cleaning**:
    ```python
    def clean_response_characters(response_list):
        cleaned_response_list = []
        for formatted_string, response, correct_answer in response_list:
            if response and ')' in response:
                last_char_before_parenthesis = re.findall(r'.(?=\))', response)[-1]
                cleaned_response = last_char_before_parenthesis
            else:
                cleaned_response = 'x'
            cleaned_response_list.append((formatted_string, cleaned_response, correct_answer))
        return cleaned_response_list

    cleaned_response_list = clean_response_characters(response_list)
    ```

7. **Accuracy Calculation**:
    ```python
    def calculate_accuracy(response_list):
        correct_count = 0
        total_count = len(response_list)
        for _, response, correct_answer in response_list:
            if response and response.strip().lower() == correct_answer.strip().lower():
                correct_count += 1
        accuracy = (correct_count / total_count) * 100
        return accuracy

    accuracy = calculate_accuracy(cleaned_response_list)
    print(f"Accuracy: {accuracy:.2f}%")
    ```

8. **Adding Responses to Excel**:
    ```python
    def add_responses_to_excel(file_path, response_list):
        df = pd.read_excel(file_path)
        if len(df) != len(response_list):
            raise ValueError("The length of the DataFrame does not match the length of the response list.")
        df['LLaMA-2 Chat (13B)'] = [response[1] for response in response_list]
        df.to_excel(file_path, index=False)

    add_responses_to_excel(file_path, cleaned_response_list)
    updated_df = pd.read_excel(file_path)
    print(updated_df.head())
    ```
    ## Using LLMs from Together AI API
This project uses Large Language Models from the Together AI API to interpret and respond to the dialogues containing international proverbs. The Together AI API provides access to powerful LLMs, which are essential for this linguistic analysis.







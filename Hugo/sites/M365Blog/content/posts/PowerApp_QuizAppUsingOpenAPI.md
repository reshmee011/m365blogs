---
title: "Building a Quiz App with OpenAI GPT in PowerApps"
date: 2023-10-12T06:50:11+01:00
tags: ["PowerApps", "OpenAI GPT","ParseJson"]
featured_image: '/posts/images/PowerApp_QuizAppUsingOpenAPI/Assets/QuizApp.png'
draft: false
---

# Building a Quiz App with OpenAI GPT in PowerApps 

Discover how to create your very own quiz app in Power Apps using the [OpenAI GPT (Independent Publisher)](https://learn.microsoft.com/en-us/connectors/openaigpt4ip/). This app was inspired by [Add OpenAI Capabilities to your Power Platform solutions](https://www.youtube.com/watch?v=AVK7BUmTGvs&t=1062s) [Robin Rosengrün](https://twitter.com/power_r2).

## Prerequisites

### API Key

Before integrating ChatGPT into PowerShell, you'll need an OpenAI API key. You can obtain one by visiting [OpenAI's API key page](https://platform.openai.com/account/api-keys) and creating a new secret key. Keep in mind that if you're using a free account, you might encounter a quota error ("You exceeded your current quota, please check your plan and billing details").

Bear in mind that the [OpenAI's pricing](https://openai.com/pricing) is per token. Generating extensive data can potentially be costly. I recommend using the browser to experiment with different prompts to determine which one yields the best data. 

### Prompt

Experiment with different prompts in the app to find the one that works best for your scenario. In the app, the prompt used to send to ChatGPT is:
 
"Generate a JSON dataset of 10 pub quiz questions. Provide each question entry in proper JSON format with a 'question' field, an 'options' field consisting of an array of 4 possible answers. Additionally, ensure that each option includes an 'answer' field with the text of the option, a 'selected' field initialized to false indicating whether the option has been chosen, and a 'correct' field indicating whether the answer is correct"

## Design the Power Apps

Create a new Power Apps and follow the steps below

### Setting Up Controls
 
1.  Add the OpenAIGPT (Independent Publisher connector). Please note that a premium license is required. 
    ![OpenAI GPT](../images/PowerApp_QuizAppUsingOpenAPI/assets/OpenAIGPTDataSource.png)
2. Add the following to the App OnStart to declare some variables that will be used in the app:

```JSON
    Set(varIndex, 1);
Set(varPromt,"Generate a JSON dataset of 10 pub quiz questions on the topic specified below. Provide each question entry in proper JSON format with a 'question' field, an 'options' field consisting of an array of 4 possible answers. Additionally, ensure that each option includes an 'answer' field with the text of the option, a 'selected' field initialized to false indicating whether the option has been chosen, and a 'correct' field indicating whether the answer is correct."
);

Set(varResponse, "[
    {
        ""question"": ""None"",
        ""options"": [
            {""Answer"": ""None"", ""Selected"": false, ""Correct"": false},
            {""Answer"": ""None"", ""Selected"": false, ""Correct"": false},
            {""Answer"": ""None"", ""Selected"": false, ""Correct"": false},
            {""Answer"": ""None"", ""Selected"": false, ""Correct"": true}
        ]
    }]");

ClearCollect(
  colQuestions,
  Table(ParseJSON(varResponse));
);

Set(varIndex,1);

ClearCollect(
    colOptions,
    AddColumns(
        Table(Index(colQuestions, varIndex).Value.options),
        "answer", Value.Answer,
        "selected", Value.Selected = true,
        "correct",Value.Correct = true
    )
);
```

3. Add the following controls

![OpenAI GPT](../images/PowerApp_QuizAppUsingOpenAPI/assets/Controls.png)

- Text Inputs for prompt (txtPrompt) and topic (txtTopic)

- Button to generate the quiz from OpenAI GPT (btnGenerate)

Set the OnSelect property to 

```JSON

Set(varResponse, 'OpenAIGPT(IndependentPublisher)'.CompletionPost("text-davinci-003",{prompt:txtPrompt.Text & Char(10) & "Topic: " & txtTopic.Text,max_tokens:4000,temperature:0.7,top_p:1,n:1}).first_completion); 
  
ClearCollect(
  colQuestions,
  Table(ParseJSON(varResponse));
);

Set(varIndex,1);

ClearCollect(
    colOptions,
    AddColumns(
        Table(Index(colQuestions, varIndex).Value.options),
        "answer", Value.answer,
        "selected", Value.selected = true,
        "correct", Value.correct = true
    )
);
```

ParseJson function has been used to convert the JSON response into a table to bind to a gallery control. 

- Add a button to move to the next question (btnNext) and set its OnSelect property:

```JSON
Set(varIndex, varIndex + 1);

ClearCollect(
    colOptions,
    AddColumns(
        Table(Index(colQuestions, varIndex).Value.options),
        "answer", Value.answer,
        "selected", Value.selected = true,
        "correct", Value.correct = true
    )
);
```

Set visible property to: If(lblSelQuestion.Text = "None" || varIndex >= 10, false, true)

- Add a Label (lblSelQuestion) to display the question with
     Text:  Text(Index(colQuestions,varIndex).Value.question)
     Visible property : If(lblSelQuestion.Text = "None", false, true)

- Add a Gallery control with a label (galQuestionsAnswers) bound to table **colOptions** and 

 Set Visible property of the gallery to:  If(lblSelQuestion.Text = "None", false, true)

- Amend the properties of the label within the gallery to 

     Text : Text(ThisItem.answer)
     OnSelect : Patch(colOptions, ThisItem, {selected: true})
     Fill : If(ThisItem.selected, If(ThisItem.correct, Color.GreenYellow, Color.PaleVioletRed), RGBA(0,0,0,0))

Sample result from OpenAI 

```JSON
[
    {
        "question": "What is the capital of France?",
        "options": [
            {"Answer": "Berlin", "Selected": false, "Correct": false},
            {"Answer": "Madrid", "Selected": false, "Correct": false},
            {"Answer": "Rome", "Selected": false, "Correct": false},
            {"Answer": "Paris", "Selected": false, "Correct": true}
        ]
    },
    {
        "question": "Which planet is known as the 'Red Planet'?",
        "options": [
            {"Answer": "Venus", "Selected": false, "Correct": false},
            {"Answer": "Mars", "Selected": false, "Correct": true},
            {"Answer": "Jupiter", "Selected": false, "Correct": false},
            {"Answer": "Saturn", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "Who wrote the play 'Romeo and Juliet'?",
        "options": [
            {"Answer": "William Shakespeare", "Selected": false, "Correct": true},
            {"Answer": "Charles Dickens", "Selected": false, "Correct": false},
            {"Answer": "Jane Austen", "Selected": false, "Correct": false},
            {"Answer": "Mark Twain", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "What is the chemical symbol for water?",
        "options": [
            {"Answer": "O2", "Selected": false, "Correct": false},
            {"Answer": "H2O", "Selected": false, "Correct": true},
            {"Answer": "CO2", "Selected": false, "Correct": false},
            {"Answer": "CH4", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "In which year did the Titanic sink?",
        "options": [
            {"Answer": "1905", "Selected": false, "Correct": false},
            {"Answer": "1912", "Selected": false, "Correct": true},
            {"Answer": "1920", "Selected": false, "Correct": false},
            {"Answer": "1931", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "What is the largest mammal on Earth?",
        "options": [
            {"Answer": "Elephant", "Selected": false, "Correct": false},
            {"Answer": "Blue Whale", "Selected": false, "Correct": true},
            {"Answer": "Lion", "Selected": false, "Correct": false},
            {"Answer": "Giraffe", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "Who painted the Mona Lisa?",
        "options": [
            {"Answer": "Vincent van Gogh", "Selected": false, "Correct": false},
            {"Answer": "Pablo Picasso", "Selected": false, "Correct": false},
            {"Answer": "Leonardo da Vinci", "Selected": false, "Correct": true},
            {"Answer": "Michelangelo", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "What is the chemical symbol for gold?",
        "options": [
            {"Answer": "Ag", "Selected": false, "Correct": false},
            {"Answer": "Au", "Selected": false, "Correct": true},
            {"Answer": "Fe", "Selected": false, "Correct": false},
            {"Answer": "Cu", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "Which country is known as the Land of the Rising Sun?",
        "options": [
            {"Answer": "China", "Selected": false, "Correct": false},
            {"Answer": "Japan", "Selected": false, "Correct": true},
            {"Answer": "South Korea", "Selected": false, "Correct": false},
            {"Answer": "Thailand", "Selected": false, "Correct": false}
        ]
    },
    {
        "question": "Who was the first man to step on the moon?",
        "options": [
            {"Answer": "Buzz Aldrin", "Selected": false, "Correct": false},
            {"Answer": "Yuri Gagarin", "Selected": false, "Correct": false},
            {"Answer": "Neil Armstrong", "Selected": false, "Correct": true},
            {"Answer": "John Glenn", "Selected": false, "Correct": false}
        ]
    }
]
```

Each time the generate button is clicked, random set of questions is generated based on the specified topic.

Watch the app in action
![QuizApp](../images/PowerApp_QuizAppUsingOpenAPI/assets/QuizApp.gif)

## Minimal Path to Awesome

* [Download](../images/PowerApp_QuizAppUsingOpenAPI/solution/quiz app.msapp) the `.msapp` from the `solution` folder
* Within **Power Apps Studio**, use the `.msapp` file using **File** > **Open** > **Browse** and select the `.msapp` file you just downloaded.
* Select the **Data** tab

### Future Improvements

1. Save generated questions and answers to a data source.
2. Allow users to view all historical quiz data.
3. Display total score at the end of the quiz.
4. Provide links to related materials for users to explore.

## Conclusion

Building a quiz app with OpenAI GPT in PowerApps opens up a world of possibilities for interactive and engaging experiences. By leveraging the power of OpenAI's language model, you can dynamically generate quiz questions tailored to your specific needs.

As you explore and experiment, consider the future improvements outlined in this guide. Saving generated questions, providing historical data, displaying scores, and offering related resources can further enhance the user experience.

With the foundation laid out in this guide, you have the tools to create compelling quiz apps that captivate and educate your audience. Get started today and watch your quizzes come to life!

## References

- [Introduction to Parse JSON in Power Apps | ParseJSON Arrays as Table; Return Array from flow](https://www.youtube.com/watch?v=FqfLiJDdC3Q)
- [Add OpenAI Capabilities to your Power Platform solutions](https://www.youtube.com/watch?v=AVK7BUmTGvs&t=1016s)

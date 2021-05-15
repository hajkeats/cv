---
title: "Revision Quiz Tool"
date: 2021-03-15T15:00:00Z
---

### [Repository](https://github.com/hajkeats/revision_quiz)

## A basic Python Revision Tool

As part of my ongoing efforts to make use of my spare time in lockdown, I have recently started learning a new language: Malay. The Malay language is very interesting because of how simple it is. There are no genders, no tenses (so no verb conjugation), and to pluralise something, rather than add an "s" for example, you would just repeat the word. So "5 words" would be "5 word-word" or "5 word2" in text-speak.

To learn, I've been watching a youtube series on [speaking Malay like a local](https://www.youtube.com/playlist?list=PLdw9ypYjBr27usUnOlpS68lMXgq7a-MZP) and making notes. My problem has been when it comes to using the notes to revise. When testing myself I found that I would tend to remember the order I had made the notes in, rather than learn the translations.

In order to solve this, I put together a quick [Python revision tool](https://github.com/hajkeats/revision_quiz). The way it works is simple:

1. You create a JSON file with a specific format, that contains the information you want to quiz yourself on.
```json
{
    "Topic 1": {
        "Question 1": "Answer 1",
        "Question 2": "Answer 2"
    },
    "Topic 2" : {
    # ...
}
```
2. You call the quiz script from a terminal with the JSON file as the argument.

The script then prompts you with the questions from your JSON file. It will give you a score, corrections etc, but is not very forgiving. Although case is unimportant, if you are a character off the correct answer it will let you know that you got the question wrong. For my purposes however, that is not too much of an issue!

Feel free to give it a go. It should work for any subject matter.

{{< image src="/images/quiz-tool.png" alt="Malay Quiz" >}}
---
title: "AWS Practitioner"
date: 2021-04-12T15:00:00Z
---

### [Repository](https://github.com/hajkeats/revision_quiz)

## Updating my revision tool for AWS Practitioner

As part of my work, I've been allocated some training time to study for the AWS Cloud Practitioner exam, which is the first rung on the ladder of AWS qualifications. After spending some time watching video tutorials, it became clear that the bulk of the studying would involve learning the various services AWS provides. With the exam being multiple choice, I thought it would be a good idea to create a list of AWS services, and adapt my revision quiz script for multiple choice.

### Making the list

The trickiest part was finding a list of services. Online I found many lists that were inadequate, incomplete or outdated, so I looked to AWS itself. Rather than trawl through the [72 page pdf](https://d1.awsstatic.com/whitepapers/aws-overview.pdf) on services that they provide, I thought it would be neater to scrape the navigation bar from the website.

Naturally, I opted for Python and [beautifulsoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) to grab and parse the names and descriptions of the data for storage in the appropriate json format. It soon became apparent that I needed to select the toolbar to get at these names and descriptions, so I was forced into using a [Selenium webdriver](https://www.selenium.dev/documentation/en/webdriver/) also.

The repository for scraping the json file can be found [here](https://github.com/hajkeats/amazon_services_scrape) and the output can be found in the [revision quiz repository](https://github.com/hajkeats/revision_quiz).

### Adapting the quiz script

In order to make the script appropriate for cloud practitioner revision, I added two further optional parameters.

#### - -multiple_choice

Adding the `--multiple_choice` argument required that I split the flow of the quiz function. As before, questions are shuffled, but now an answer pool is collated, with 3 wrong answers and one right. The user must then enter either A, B, C or D to answer the question.

{{< image src="/images/multiple-choice.png" alt="Multiple Choice" >}}

#### - -reversed

Another neat idea I had was to reverse the quiz, so rather than asking the user the keys of the json as the question, it would ask the values, and give the keys as optional answers. This argument was added independently to the `--multiple_choice` argument, so it would also work with the regular quiz mode, where you would have to type out the whole correct answer.

{{< image src="/images/reversed.png" alt="Multiple Choice Reversed" >}}


I hope to pass the exam in the coming weeks. To test your AWS knowledge, run the following from your terminal.

```
git clone git@github.com:hajkeats/revision_quiz.git

cd revision_quiz

./quiz.py aws.json --multiple_choice --reversed
```
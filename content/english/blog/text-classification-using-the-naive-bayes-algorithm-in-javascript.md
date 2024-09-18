---
title: "Text Classification Using the Naive Bayes Algorithm in JavaScript"
meta_title: "Naive Bayes Algorithm for Text Classification with JavaScript"
description: "Learn how to implement the Naive Bayes algorithm for text classification in JavaScript using ES6 classes. This step-by-step guide includes code examples and explanations to help you understand Naive Bayes in a clear and practical way."
date: 2024-09-18
image: "/images/js-naive-bayes.webp"
categories: ["JavaScript",
  "Machine Learning",
  "Algorithms",
  "ES6",
  "Web Development"]
author: "Beta Priyoko"
tags: ["Naive Bayes",
  "Text Classification",
  "JavaScript ES6",
  "Machine Learning",
  "Natural Language Processing",
  "JavaScript Algorithms",]
draft: false
---
#### Introduction:
In this article, we will explore the **Naive Bayes algorithm** for text classification and how to implement it using **JavaScript ES6 classes**. Naive Bayes is a simple yet effective classification algorithm based on probability theory, widely used for natural language processing tasks like **spam detection**, **sentiment analysis**, and **document categorization**.

We will go through the theory behind the algorithm, break it down step by step, and implement it in a class-based structure. By the end of this tutorial, you'll understand how to build a Naive Bayes text classifier in JavaScript.

#### What is the Naive Bayes Algorithm?

Naive Bayes is a **probabilistic classification algorithm** based on **Bayes' Theorem**. It works by calculating the probability of a given input belonging to a particular class. The "naive" part comes from the assumption that all features (words in the case of text) are independent, which simplifies the calculation. Even though this assumption may not always hold, Naive Bayes often performs well in practice.

##### Bayes' Theorem:
The core formula behind Naive Bayes is **Bayes' Theorem**:

\[
P(C | X) = \frac{P(X | C) \cdot P(C)}{P(X)}
\]

Where:
- **P(C | X)**: The probability of class C given the input X.
- **P(X | C)**: The probability of input X occurring given the class C.
- **P(C)**: The prior probability of class C.
- **P(X)**: The total probability of input X across all classes.

The goal of the algorithm is to find the class with the highest probability for a given input.

#### Step-by-Step Breakdown of Naive Bayes for Text Classification:

##### 1. **Tokenization:**
   The first step is to split the input text into individual words, known as **tokens**. This helps the algorithm understand the composition of the text.

##### 2. **Calculate Priors (P(C)):**
   We calculate the **prior probability** for each class. The prior is simply the frequency of a class in the training dataset divided by the total number of documents.

##### 3. **Calculate Likelihood (P(X | C)):**
   For each word in the input text, we calculate the **likelihood**, which is the probability of the word occurring in a particular class. We also apply **Laplace smoothing** to handle words that may not appear in the training data.

##### 4. **Multiply the Probabilities:**
   Using **logarithms** to avoid underflow issues with very small probabilities, we multiply the likelihoods of each word by the prior for that class.

##### 5. **Classify the Document:**
   Finally, we choose the class with the highest computed probability.

#### Naive Bayes Classifier Implementation in JavaScript (ES6 Classes)

Here’s how you can implement the Naive Bayes classifier using ES6 classes:

```javascript
class NaiveBayesClassifier {
  constructor() {
    this.wordCount = {};
    this.categoryCount = {};
    this.vocabulary = new Set();
    this.totalDocuments = 0;
  }

  // Tokenizes the text into words
  tokenize(text) {
    return text
      .toLowerCase()
      .replace(/[^\w\s]/g, '') // Remove punctuation
      .split(/\s+/); // Split by spaces
  }

  // Trains the classifier with a document and its category
  train(document, category) {
    const tokens = this.tokenize(document);

    if (!this.categoryCount[category]) {
      this.categoryCount[category] = 0;
      this.wordCount[category] = {};
    }

    // Increment category count
    this.categoryCount[category]++;
    this.totalDocuments++;

    // Count words for the category
    tokens.forEach((word) => {
      this.vocabulary.add(word);
      if (!this.wordCount[category][word]) {
        this.wordCount[category][word] = 0;
      }
      this.wordCount[category][word]++;
    });
  }

  // Calculate the probability of a word given a category
  wordProbability(word, category) {
    const wordFrequencyInCategory = this.wordCount[category][word] || 0;
    const totalWordsInCategory = Object.values(this.wordCount[category]).reduce(
      (acc, count) => acc + count,
      0
    );

    // Laplace Smoothing to avoid zero probabilities
    return (wordFrequencyInCategory + 1) / (totalWordsInCategory + this.vocabulary.size);
  }

  // Classify a new document
  classify(document) {
    const tokens = this.tokenize(document);
    let categoryScores = {};

    for (const category in this.categoryCount) {
      // Calculate the prior probability
      categoryScores[category] = Math.log(this.categoryCount[category] / this.totalDocuments);

      // Calculate the likelihood for each word
      tokens.forEach((word) => {
        categoryScores[category] += Math.log(this.wordProbability(word, category));
      });
    }

    // Return the category with the highest score
    return Object.keys(categoryScores).reduce((a, b) =>
      categoryScores[a] > categoryScores[b] ? a : b
    );
  }
}
```

#### Explanation of the Code:

1. **`tokenize(text)`**:
   - This method processes the input text by converting it to lowercase, removing punctuation, and splitting it into individual words. Tokenization is essential for extracting features from the text.

2. **`train(document, category)`**:
   - This method updates the model by adding the document’s tokens to the word count for the given category. It tracks the number of documents per category and the vocabulary (unique words encountered).

3. **`wordProbability(word, category)`**:
   - This method calculates the probability of a word occurring in a specific category using **Laplace smoothing** to handle words that may not appear in the training data.

4. **`classify(document)`**:
   - This method classifies a new document by calculating the probabilities for each category. It uses the **logarithm of probabilities** to avoid issues with very small numbers. The category with the highest score is chosen as the prediction.

#### Example Usage:

```javascript
const classifier = new NaiveBayesClassifier();

// Training the classifier
classifier.train('The weather is sunny and bright today', 'Positive');
classifier.train('It is raining heavily outside', 'Negative');
classifier.train('What a wonderful and sunny day', 'Positive');
classifier.train('The storm is really bad', 'Negative');

// Classifying a new document
const result = classifier.classify('It is a sunny day');
console.log(`The document is classified as: ${result}`);
// Output: The document is classified as: Positive
```

#### Conclusion:
The **Naive Bayes algorithm** is an effective and easy-to-implement classification technique, especially useful for text classification tasks. By using a class-based approach with **ES6 features**, we can create a flexible and reusable classifier in JavaScript. Whether you're building a spam filter, sentiment analyzer, or any other text-based classifier, Naive Bayes can be a great starting point.

Feel free to experiment with larger datasets and add more complex preprocessing steps, such as **stop word removal** or **stemming**, to improve the performance of your classifier!
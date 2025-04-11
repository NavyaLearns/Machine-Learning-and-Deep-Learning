---

# **Documentation on Implementation of Book Review Sentiment Analysis Application**

## **1. Introduction**
The goal of this project is to build a sentiment analysis application that evaluates Goodreads book reviews. The application fetches reviews using web scraping, processes the text through custom sentiment analysis, and then aggregates the sentiment scores. We aim to create a recommendation system based on sentiment analysis and key terms from reviews.

## **2. Web Scraping**
### **Overview of Web Scraping**
Web scraping is the process of extracting data from websites, in this case, from Goodreads to collect book reviews. We used Python's `requests` and `BeautifulSoup` libraries to scrape the book reviews.

### **Key Implementation Steps**
1. **Fetching the Web Page:**
   ```python
   response = requests.get(url)
   ```
   The `requests` library sends an HTTP request to the provided Goodreads book URL. The page's HTML content is retrieved for further parsing.

2. **Parsing the HTML Content:**
   ```python
   doc = BeautifulSoup(response.text, 'html.parser')
   ```
   After receiving the page content, we use `BeautifulSoup` to parse the HTML and navigate its structure. This allows us to extract specific parts of the page (like reviews, likes, and comments).

3. **Extracting Review Data:**
   ```python
   reviews = doc.find_all('article', class_='ReviewCard')
   ```
   The reviews are encapsulated within `article` tags with the class `ReviewCard`. This is the section we focus on to collect individual reviews.

4. **Extracting Likes, Comments, and Review Text:**
   ```python
   likes = extract_likes_and_comments(review, 'likes')
   comments = extract_likes_and_comments(review, 'comments')
   review_text = extract_review_text(review_text_section)
   ```
   We extract the review text, number of likes, and comments by finding specific HTML elements that contain these values. The review text is further cleaned for formatting, such as bold or italic text.

### **Reasoning Behind Choices**
- **`requests` and `BeautifulSoup`** are chosen because they are efficient and widely used libraries for web scraping.
- **Extracting likes and comments**: This allows us to weigh the sentiment of reviews based on their popularity, i.e., reviews with more likes and comments are more influential.
- **Formatting review text**: By extracting bold, italicized, and blockquoted text, we aim to preserve important nuances in reviews, making them richer for sentiment analysis.

## **3. Preprocessing and Sentiment Analysis**
### **Preprocessing Steps**
Before performing sentiment analysis, it's crucial to preprocess the review text. The following steps were implemented:

1. **Tokenization:**
   ```python
   tokens = re.findall(r'\b\w+\b|[^\w\s]', review)
   ```
   Tokenization breaks down the review text into words (tokens). This includes punctuation as separate tokens, which is important for understanding sentiment in context (e.g., exclamation marks can amplify sentiment).

2. **Negation Handling:**
   ```python
   if token.lower() in NEGATION_WORDS:
   ```
   Words like "not," "never," or "can't" are handled separately. When encountered, they invert the sentiment of the following word. We account for double negations to avoid errors (e.g., "not bad" is positive).

3. **Intensity Modifiers:**
   ```python
   sentiment_value *= INTENSITY_MODIFIERS[tokens[i - 1]]
   ```
   Words like "very" or "extremely" amplify or reduce the sentiment score. This is handled by multiplying the sentiment value of the word by an intensity factor.

4. **Stopword Removal:**
   ```python
   elif token.lower() not in STOPWORDS:
   ```
   Common words like "the," "is," and "and" are removed because they don’t contribute meaningful sentiment information. Negation words and intensity modifiers are preserved, as they are essential for sentiment analysis.

### **Sentiment Scoring**
- **Positive and Negative Scores:**
  Sentiment is determined by looking up each token in a lexicon (`SENTIMENT_LEXICON`). If a token matches a word in the lexicon, its corresponding sentiment score (positive or negative) is added.
  
- **Exclamation Handling:**
  ```python
  sentiment_value *= (1 + 0.25 * token.count("!"))
  ```
  Words with exclamation marks (e.g., "fantastic!!!") are given extra weight, as multiple exclamation marks amplify the sentiment.

### **Reasoning Behind Preprocessing Choices**
- **Negation handling** ensures that phrases like "not good" or "can't stand" are properly understood as negative, which is crucial for the accuracy of sentiment analysis.
- **Intensity modifiers** allow the sentiment to be more nuanced, providing a better representation of how strongly a reviewer feels about the book.
- **Stopword removal** helps eliminate noise, allowing the model to focus on words that contribute to sentiment.

## **4. Sentiment Analysis Models**
The sentiment analysis model was inspired by the **VADER sentiment analysis** model, part of the **NLTK library**, but with modifications specific to our dataset.

### **Key Components**
| **Feature**                     | **VADER**                            | **Custom Implementation**                               |
|----------------------------------|--------------------------------------|---------------------------------------------------------|
| **Lexicon-based approach**      | Yes, uses a predefined lexicon        | Yes, custom lexicon tailored to book reviews             |
| **Negation handling**           | Inverted polarity for negation words | Explicit handling of negation words, including double negations |
| **Intensity Modifiers**         | Yes, applies intensity to sentiment   | Yes, custom intensity factors (e.g., "very", "extremely") |
| **Exclamation Handling**        | Amplifies sentiment for exclamations | Custom amplification for multiple exclamation marks    |
| **Stopwords**                   | Removes common stopwords             | Removes stopwords but retains negations and intensifiers |

### **Key Differences**
- **Custom Lexicon**: Unlike VADER, which uses a generic lexicon, we tailored our lexicon specifically for book reviews, making it more sensitive to language commonly used in reviews.
- **Double Negation Handling**: VADER doesn't handle double negations explicitly. In our approach, if two negations are encountered in sequence (e.g., "not bad"), we negate the negation, which improves accuracy.

## **5. Performance Evaluation**
The performance of the sentiment analysis model is evaluated by aggregating sentiment scores (positive, negative, neutral) across multiple reviews, considering the number of likes and comments as weights.

### **Next Steps for Evaluation:**
- Train the model on a larger dataset to improve accuracy.
- Implement cross-validation with real labeled datasets to evaluate precision, recall, and F1-score.
- Analyze the impact of different parameters, such as the inclusion of exclamation marks and the intensity modifier thresholds.

## **6. Conclusion and Future Work**
### **Conclusion**
This project successfully implements a custom sentiment analysis model inspired by VADER, tailored specifically for book reviews. The application performs web scraping to collect reviews, preprocesses them, and computes sentiment scores. The sentiment distribution is visualized to provide an overview of how readers feel about a book.

### **Future Work**
- **Improving Data Coverage**: To optimize performance, we will train the model with a larger, more diverse dataset of book reviews, allowing the lexicon and sentiment rules to be fine-tuned.
- **Interactive Web Application**: We aim to build an interactive webpage where users can input a Goodreads link, and the application will provide a sentiment summary along with key review tags. This could eventually evolve into a **book recommendation system**, where similar books with positive sentiment and shared keywords are recommended to users.

By continuously refining the model and expanding its capabilities, we plan to make the system robust enough to assist in book recommendations based on user preferences.

--- 


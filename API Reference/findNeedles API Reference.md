

# **findNeedles API Reference**

# Method Signature

| Method | findNeedles(String haystack, String\[\] needles) |
| :---- | :---- |
| **Description** | Counts occurrences of specific words (needles) within a larger string (haystack) and prints the count for each word. Suitable for text analysis and word frequency tasks. |

# Parameters and Returns

| Parameter | Type | Description |
| :---- | :---- | :---- |
| haystack | String | The text to search within. |
| needles | String\[\] | An array of words to count within the haystack. Maximum of 5 words. |

| Returns | Type | Description |
| :---- | :---- | :---- |
| void | void | Prints the count of each word in needles to the console. |

# Usage Example

String text \= "The cat sat on the mat. The mat was flat.";  
String\[\] words \= {"mat", "cat", "dog"};  
findNeedles(text, words);

#### **Output**

mat: 2  
cat: 1  
dog: 0

# Test Cases

// Basic Test Case  
String text \= "The quick brown fox jumps over the lazy dog";  
String\[\] needles \= {"fox", "dog", "cat"};  
findNeedles(text, needles);

// Edge Case \- No Matching Words  
String emptyHaystack \= "";  
findNeedles(emptyHaystack, needles);

// Exceeding Word Limit  
String\[\] tooManyNeedles \= {"fox", "dog", "cat", "rabbit", "wolf", "bear"};  
findNeedles(text, tooManyNeedles);

# Notes

* The haystack string is tokenized using split() with spaces and common delimiters (for example, tabs, newlines).  
* The needles array allows a maximum of 5 words. If exceeded, the method outputs "Too many words\!" to the error stream.
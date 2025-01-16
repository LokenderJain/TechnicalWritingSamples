# **Cover Sheet**

# Title

findNeedles API Reference

# Type

API Reference

# Code

public static void findNeedles(String haystack, String\[\] needles) {  
if (needles.length \> 5\) {  
System.err.println("Too many words\!");  
} else {  
int\[\] countArray \= new int\[needles.length\];  
for (int i \= 0; i \< needles.length; i++) {  
String\[\] words \= haystack.split("\[ \\"\\'\\t\\n\\b\\f\\r\]", 0);  
for (int j \= 0; j \< words.length; j++) {  
if (words\[j\].compareTo(needles\[i\]) \== 0\) {  
countArray\[i\]++;  
}  
}  
}  
for (int j \= 0; j \< needles.length; j++) {  
System.out.println(needles\[j\] \+ ": " \+ countArray\[j\]);  
}  
}  
}

# Purpose

This document is divided into two parts:

**API Reference**: Explains how to use the `findNeedles` API, which identifies and counts occurrences of specific words (needles) within a larger string (haystack). Aimed at experienced Java programmers, it includes the method signature, parameter descriptions, return types, usage examples, and test cases to support API integration.

**Code Improvement Suggestions**: Provides feedback to enhance the API’s performance, reduce memory usage, and improve usability, based on technical analysis and real-world use cases.

# Questions

*Whether the content was written solely by you, or if you were part of a team that worked on the content. If you were part of a team, please identify as exactly as possible which portions were your responsibility.*

‣ I authored over 90% of the content, referencing industry-standard API documentation, online resources, and code review practices.

*If you used generative AI during the writing process, please describe how the tool was used.*

‣ I used generative AI to assist with writing and validating the syntax for `HashMap` and related operations. AI helped me research concepts such as optimizing performance, implementing proper error handling, and improving code readability and usability.


# **Feedback on the \`findNeedles\` Method**

Hello \[Engineer Name\],

I looked at the **findNeedles** method and found a few areas that might help improve performance and usability.

1. We could explore using a HashMap to store the needle counts and lookup, which would prevent repetitive iterations.  
2. Splitting the haystack outside the loop could avoid redundant operations.  
3. The code currently uses compareTo to check for string equality. However, switching to equals() could simplify the comparison without altering the logic.  
4. For error handling, how do you feel about returning an error message when the needles array exceeds five elements instead of logging to System.err? This could make error handling more consistent for users.  
     
   if (needles.length \> 5) {  
     return new String\[\]{"Error: Too many words\!"};  
   }

   Map\<String, Integer\> needleCounts \= new HashMap\<\>();

   for (String needle : needles) {

     needleCounts.put(needle, 0);

   }

   String\[\] words \= haystack.split("\[ \\"'\\t\\n\\b\\f\\r\]"); 

   for (String word : words) {

     if (needleCounts.containsKey(word)) { 

       needleCounts.put(word, needleCounts.get(word) \+ 1);

     }

   }

   for (int i \= 0; i \< needles.length; i++) {

   System.out.println(needles\[i\] \+ ": " \+ needleCounts.getOrDefault(needles\[i\], 0);

   }

   `return "success!";`

I would like to hear your thoughts on this approach. Thank you.

Best,  
Lokender Jain
# Fuzzymatching 


Objective: The broad intent is to develop a fuzzy matching algorithm to match strings from two sources. In my porject, I use data from Patent View and CRSP data to match company names from both the data bases.
Coding language used: Python 
Packages: Pandas, numpy, fuzzywuzzy, regex
Memory: requires large RAM memory fuzzy matching invloves a matrix of size m by n, where m and n are the sizes of datasets invloved

Advantages: Can match names with 
 - spelling errors: e.g. International business machines, Internatnal business machines.
 - names with slight vairations: 
 e.g KONINKLIJKE PHILIPS ELEC N V with Koninklijke Philips Electronics N.V.
 e.g. Sony electronics with Sony
 - names with difference in order of strings e.g. W.R.Grace & Co with Grace WR and company
 
 Limitations:
 - Cannot match names with short forms/ trade acronyms
 e.g. “general refractories" to "grefco" will not be matched
 e.g. "internet pictures” to "ipix"
 e.g. "Internatnal business machines corp" to "ibm"

Files included





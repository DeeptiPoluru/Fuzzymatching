# Fuzzymatching 


Objective: The broad intent is to develop a fuzzy matching algorithm to match strings from two sources. In my porject, I use data from Patent View and CRSP data to match company names from both the data bases and use KPSS data to test the matches.
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

Raw Files:  

1. rawassignee.tsv : Patent View data file with patents and company names: source: http://s3.amazonaws.com/data.patentsview.org/20191231/download/assignee.tsv.zip
2. CRSP_Names_Data_Monthly.txt : CRSP data file with CRSP company names, year, permno
https://drive.google.com/open?id=1QbHpWCb1VrreadGuupkO9O4VTGzv4JAG

3. KPSSdata_DirectFromWeb_GroundTruth.csv: KPSS data file with PERMNO and year
https://paper.dropbox.com/ep/redirect/external-link?url=https%3A%2F%2Fwww.dropbox.com%2Fs%2Fe7oopkmv032cz6j%2Fpatents_xi.zip%3Fdl%3D0&hmac=gqntXBaplUJL%2FaJXtkBim%2F%2FdeLV5u8PVlwvI4PTRgFI%3D

ipython notebooks: 
1. KPSS_clean_11Feb2020.ipynb: cleaning of KPSS data file
2. CRSP_clean_23Apr2020.ipynb: cleaning of CRSP data file
3. PV_rawassignee_23Apr2020.ipynb : cleaning of Patent View data file
4. kpss_crsp_gt_28Apr20.ipynb : ground truth from 3-way matching between CRSP-Patent View-KPSS
5. hi_freq_pv_crsp_30Apr20.ipynb: remove high frequency last words from patent view nad CRSP company names after cleaning
6. Fuzzy_calculations_KPSS_CRSP_30Apr20.ipynb: fuzzy score computation
7. Precion_Recall_CRSP_PV_30Apr20.ipynb: precision-recall caluclations

Results: 

threshold score >71 gives 99.2 matches based only on spellings(Excluding short forms/ abbreviations/trade names from ground truth) wiht precision of 94.54% (upper limit)  and recall 99.93%.



# Generic fuzzy matching Steps: 
1. Convert datasets into pandas dataframes 
   - data1 = pd.read_csv("data1.csv")
   - data2 = pd.read_csv("data2.csv")
2. Establish base matches: Compute direct matches without any algorithm using pandas merge between the two datasets. This gives the base matches, which can be compared to the matches from fuzzy matching algorithm to determine its efficiency.
 - convert strings to lower case
   
   - for company_name in data:
        clower = company_name.lower()
 
 - remove spaces in strings
   - clower_nospace = clower.replace(" ","")
   
 - inner join
   - direct_merge = pd.merge(data1, data2, how = "inner", left_on = [clower_nospace_data1], right_on = [clower_nospace_data2])
   
 - get number of matches
   - direct_merge.nunique()
   
 3. Fuzzy matching: This is divided into three sub-steps steps:
 
    i. standardize names and compute fuzzy matching scores of standardized names.
 
    ii. remove high frequency words from the names without altering the names and compute fuzzy matching score of these new names.
 
    iii. Use a combination of scores from a,b to create a threshold for matching.
 
 - standardize names: This needs to be customized based on the nature of data. In the datasets I was using, the company names 
   had common suffies such as corporation, inc etc which could be standardized.
   
      a. Convert to lower case
      
      b. Remove common company suffix based on frequency of last words such as corporation, inc, gmbh etc.
      
      c. Remove geographic locations at the end of company names such that it does not change the company name. e.g. ASML   
         Netherlands is saved as asml in cleaned version.
         
      d.	Remove punctuation
      
      e.	Join single characters e.g. W I C O will be wico
      
      f.	Remove text from braces where it is possible: e.g. Trailmor [Proprietary] Limited == Trailmor Limited
           e.g. Oxford Instruments (UK) Limited == Oxford Instruments Limited
           
      g.	Remove common words of in the names without altering teh name e.g. the
      
      h.	Other rules for standardization
      
            o	Convert united states to us
            o	Convert telefonaktiebolaget to telephone
            o	Lab tp laboratory
            o	Labs to laboratories
            o	Join  “tele communications” to form a single word “telecommunications”
            o	Convert “telecom” to “telecommunications”
            o	Convert “comm” at the end of company names to “communications” e.g. “General Data Comm” is saved as “general  
              data communications”
              
- fuzzy matching based on levenshtein distance (https://www.datacamp.com/community/tutorials/fuzzy-string-python) 
      a.	Fuzz.ratio computes standard levenshtein score between two strings
          fuzz_ratio =  fuzz.ratio(data1_std_name, data2_std_name) 

          Example:
          data1_name: “GOOGLE INC”
          data1_std_name : “google”
          data2_name: “Googole, Inc.”
          data2_std_name: “googole”
          fuzz _ratio =  fuzz.ratio (“google”, “googole”)==92

     b.	Fuzz.token_set_ratio computes levenshtein score between two strings after tokenizeing the strings, removing the common strings, sorting the remaining strings in alphabetical order and then using fuzz.ratio() pairwise comparisons between the  new strings
          
          Example:
          data1_name: “W. R. Grace & Co., -Conn.”
          data1_std_name : “wr grace”
          data2_name: “GRACE W R & CO”
          data2_std_name: “grace wr”
          fuzz _token =  fuzz.token_set_ratio (“wr grace”, “grace wr”)==100
          
          In fuzz matching, both fuzz.ratio and fuzz.token_set_ratio are useful for the datasets I was using. There are a few other fuzzy functions available, whcih can be explored (https://www.datacamp.com/community/tutorials/fuzzy-string-python) 
          
For fuzzy matching, we need to create a matrix with names from both the datasets to compute fuzzy scores for every possible pair of names. For this I use meshgrid function from Numpy.

    x = np.array(np.meshgrid(data1.name_std.values, data2.name_std.values)).T.reshape(-1,2) . # create matrix with both datasets
    df.columns = ['data1_names','data2_name'] # create dataframe 
    
    df['fuzz_ratio'] = [fuzz.ratio(*i) for i in map(tuple, x)] # compute fuzz.ratio scores for both the datasets
    df['fuzz_token'] = [fuzz.token_set_ratio(*i) for i in map(tuple, x)] # compute fuzz.token_set_ratio scores for both the datasets
    
    
          
- remove high frequency words occuring at the end of strings without altering the names.
  data1_unq =  data1.drop_duplicates(['name_clean'], keep = 'first') ##only unique names
  data1_unq['last'] = data1_unq['name_clean'].apply(lambda x: x.split()[-1]) ## last words in the unique names
  
  data2_unq =  data2.drop_duplicates(['name_clean'], keep = 'first') ##only unique names
  data2_unq['last'] = data2_unq['name_clean'].apply(lambda x: x.split()[-1]) # last words in the unique names
  
  #get frequency of last words in the names
  l1 =  list(data1_unq['last'])
  l2 = list(data2_unq['last'])
  lst = list(l1 + l2)
  hfq = pd.DataFrame({'lst':lst})
  z = hfq['lst'].value_counts() 
  check high frequent words which do not alter company names. removing these will minimize false positives.
  
  e.g. sony corp after removing high freq last words will be sony
  
- compute fuzz.ratio and fuzz.token for the strings using mesh grid and fuzzywuzzy.

- use a combination of fuzzy scores to determine threshold for matching. All scores == 100, would be a definite match. 
        


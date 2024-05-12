## Matching and identifying terms in the text

### Introduction
 This project involves developing a tool for matching and identifying terms in text documents efficiently. The tool operates on a document containing several pages and a sizable database (dictionary) of terms, which may consist of single or multiple words. The primary objectives are to identify and tag all terms present in the document.

### Tool Selection
For this project we had three candidate tools to use, these are; PostgreSQL, MongoDB, and Apache Lucene. All these tools support full-text search and all of them are capable of working efficiently with the data in our project scale. After doing some research, we decided to work on MongoDB because it is easy to use and suitable for use in the Python environment thanks to the Pymongo library.

### Our Approach
Firstly, in the data preprocessing stage we just made it easier to look up the words in the database, but also made sure that the preprocessing does not affect the matching results somehow. We strip the sentences, remove punctuation, and tokenize the words. It is important to note that we do this step for all the text but then also follow other steps for multiwords. For multiwords, we are considering 2 different variables. One is the length of the sentences that will take place on the full-text search that will be extracted from the text, and two is the number of words that will be skipped between these sentences.

#### Demonstration of the approach
Let's say we have such a text;
Word1 word2, word3 word4 word5 word6 word7 word8. Word9 word10 word11 word12, word13 word14...

The extraction for the basic method that we found to give the best results looks like this;
'word1 word2 word3 word4 word5 word6' 'word5 word6 word7 word8 word9 word10'

In this case, we decided the length of the sentence as 6, and the number of words to skip as 4. With these numbers, it is possible to detect up to 3 multiwords instead of grouping the texts by 6-word sentences that would cause separation of some multiwords in the process. It is possible to implement later an application in such a way that the user could give the length of sentences (an increase of this number would decrease the time the process takes, but also decrease the accuracy of the results), and the number of words to skip (an increase of this number would decrease the time the process takes, but also decrease the number of multi-words that could be found).

#### First search on the database

In the next step, we just perform a simple query that could find a huge number of single words in the text using the "$in" operator. Then, we perform the same query on the multi-words to find some quick results of those as well. It is important to note that initially there were no multiwords in the database, we added some for testing purposes. So initially, there were too little number of multiwords that could be found right away with this query.

We have created a text-search index on the 'word' column before the next step.

On the NEW CODE section, firstly we perform our base algorithm on the single words that couldn't be found with the initial query then finally perform the same algorithm on the multiwords.

#### Explanation of the base algorithm

Since we use the Levenshtein distance, the cell starts with the selection of the threshold for the distance. For each word in the list, we create a cursor using a full-text search and save all the words found in the database using this query into a list. Then, we perform the same steps but with cleaning the words from numbers and '-' dashes using regex. The reason we have decided to do that is that most of the words that we didn't find in the database had these characters. So this step was making the results much more accurate and didn't increase the time too much. Finally, we compare all the candidates with the word using the Levenshtein distance and create a dictionary taking the closest candidate word.

There are very few differences in the algorithm for the multiwords. Firstly, since we are considering the multiwords, we are not taking any candidates that are single words. Then, when we are calculating the Levenshtein distance we add the length of the candidate word with the distance and then subtract the length of the word. The reason we do that is since the length of the sentence we are checking is very long (because it has 6 words by default), we are considering that the Levenshtein results might be affected by this.

### Deployment Instruction

**MongoDB Registration & Depyolment**

- Create a MongoDB account via the "https://www.mongodb.com/" site.
- Create a new project by clicking on the “Projects” tab in MongoDB.
- Create a deployment for the specified project by clicking “Create” on “DEPLOYMENT” tab.
- When creating a deployment, the desired cloud server to be used can be specified. By default, a certain fee is charged for the service. It is recommended to choose the free "M0" cloud service and AWS as provider.
- In the next step, you will be asked to customize the information required to connect to the service. Here, the information required for authentication is selected as username and password. Then, you need to specify the username and the password and create the user. Finally, you need to enter the IP addresses of the devices you want to use to access the service in the list at the bottom of the page. This step is important because the service cannot be accessed from a device whose IP address is not on the list.
- Then it reroute you to overview. Here is an interface for us to connect to the deployment we just created. We proceed to the connection steps by clicking the "Connect" button under the name of the deployment we created.
- Here we are given various alternatives to connect. We preferred to use the "MongoDB Compass" application.
- After selecting the required option for MongoDB Compass, it directs us to the next page. Here are instructions for installing the application if it is not installed on your system. If you have the app, you can select the second option. Since we are working with the most current version of the application, we selected the "1.12 or later" option. Below is the connection string provided to access our deployment when using the MongoDB Compass application. The connection string is in the format "mongodb+srv://username:<password>@cluster0.jwl5k2o.mongodb.net/". You must fill in the required fields with your username and password.

**MongoDB Compass**

- Open MongoDB Compass application.
- Paste the connection string in the “New Connection” section and click the “Connect” button.
- We can create a new database by pressing the “+” sign next to the “Databases” option in the menu on the left.
- We can add data to our database by pressing the "Add data" button. Since the data set we use is in csv format, we select the first option, "Import JSON or CSV file". We select the csv file we will use from the window that opens. And the data inside the file starts to be scanned. When the scanning process is finished, we can see the file content below. Here we continue by clicking the "Import" button without changing any settings.
- The import process can take approximately 2-3 minutes. After importing your dataset is ready for use.
- As the last step, we go to the "DEPLOYMENT" section on the MongoDB site, click the "Connect" button and select the "Drivers" option. We select "Python" as the driver and "3.12 or later" as the version. Then, we activate the "view full sample code" option and copy the resulting code snippet. The snipped should look like this:
    - from pymongo.mongo_client import MongoClient
        
        from pymongo.server_api import ServerApi
        
        uri = "mongodb+srv://username:<password>@cluster0.jwl5k2o.mongodb.net/?retryWrites=true&w=majority"
        
        # Create a new client and connect to the server
        
        client = MongoClient(uri, server_api=ServerApi('1'))
        
        # Send a ping to confirm a successful connection
        
        try:
        
        client.admin.command('ping')
        
        print("Pinged your deployment. You successfully connected to MongoDB!")
        
        except Exception as e:
        
        print(e)
        
- By pasting this snippet into the code, you can access the database.

#### User Manual | Description of the methods

After following the steps of deployment, and after you install the required Python libraries, you should be able to ping your deployment and successfully connect to MongoDB on the connection check cell inside the notebook.

Requirements for libraries are included in requirements.txt

Following steps, instructions and information:

**Connection**

- ‘client.list_database_name()’ code should give you the information database names, after selecting the database you uploaded ex_words in, you should also put the name of the collection inside give variable
    - ex_words = client.database_name.colletion_name
- If you are running the code for the first time you should uncomment the adding indexes and multiwords and run the cells because these information does not exist in the initial database. If the user wants to add more words or multiwords inside the database, it is possible just to extend the array with those words.

**Data Preprocess**

- First we have the functions required for the data preprocessing. After running these functions, user have following options to choose:
    - Name of the txt file that will be read should be determined with ‘file_name’ variable.
    - Max number of multi words (the numbers of words that will be skipped) should be determined by the ‘max_multi_words’ variable. It is initially 3. In this version it is not possible to not use this variable so it is not possible to select the variable as 4 and use gel_multiwords4.
    - The length of the sentences that will be used inside the query should be determined while using the get_multiwords function. While returning the results of this function to the multiwords variable, the user should determine which functions should be used. Options are: get_multiwords4, get_multiwords5, get_multiwords6, get_multiwords8, get_multiwords10. The number at the end of the function determines the length of the sentences.
    - The user can see some information about the uploaded data after preprocessing.

**Query Testing**

Firstly we check all the single words inside the database since we can get these results very quick. The user can see amount of unique words from text found in the database, and the the words themselves. 

On the next step, it is possible to find the words that are not found inside the database, so we save the inside an array to find them later. User can see the number of single words and the words themselves that are not found  in the database.

On the next step, we use the same query to try to find the multi-words directly inside the database. The results can be observed in the same way. But since multi-words require more through search, it is possible to see that initially we can’t find many words. Also chosen length for the sentences affects these results, if the length is selected as 5, it would only find the multi-words that has the length of 5. For shorter multi-words, final runs are required.

**Explain**

It is possible to see how much time previous queries took in this section.

**Single Words Couldn’t Be Found**

In this section, user can find the single words that couldn’t be found previously.

How does it run:

- We create a dictionary to save the results
- It is possible for user to select Levenshtein distance threshold but it is highly recommended to choose 3 or 2.
- We search each not found word in the database again using full-text search, and save the candidate words.
- We do the search again but after cleaning the words even more from numbers and dashes.
- We put this candidates uniquely inside a list and compare the Levenshtein distance with the word that we are searching for.
- If the word found has a shorter distance than the threshold, we register it to the results dictionary. This result might change if later on there is a better result with a shorter distance.

After this the user can see the time it took for this step (which should be quite short). On the next cell user can see the results in such format 

{word that is initially not found: best candidate word that the algorithm found}

**Multiword Search**

At this step, we are performing the final search with a little bit altered version of the previous step on multi-words. 

Differences:

- Levenshtein distance threshold is still up to the user but in this step we highly recommend a smaller value such as 2.
- We have additional variable to show the user time it took for this process.
- We are not including the candidates we get which are not multi words since we are only looking for multi words.
- Since the sentence length could go up to 10, it is possible to get more than one multi word from a sentence, so it is possible for the user to see which multi words coming from which sentence.
- The most important difference in this step is how we consider the Levenshtein distance. We have a new calculation where:
    
    Check_Distance = Length of the candidate - Levenshtein distance - Length of the sentence
    
    The reason we are doing this instead of using Levenshtein distance directly is that since the length of the whole sentence is very long, the algorithm would be more vulnerable, and would make more mistake.
    

Those results could be seen on the next steps in this format:

- {sentence: multi-word/multi-words found}
- Time it took for whole algorithm to run
- Time cursor took

**Find the words inside the text**
Finally, after adjusting the results, the user can see all the unique words identified from the text using the variable 'final_results'.

And on the final cell, the user can see the positions and how many matches the words have.

### Future Improvements

- This could be made into an app and the important values I have mentioned before that could affect the accuracy, speed, and length of multiwords of the search could be taken as inputs from the user.

- Punctuations on the preprocessing could be used much more effectively for the selection of sentence lengths.

- There could be a better way to determine the similarities between the sentences and candidates than Levenshtein distance.

- There could be a way to determine if the candidate is a multiword or not BEFORE not including them as a candidate. Especially while using the cursor. We believe that would decrease the time dramatically.

- The results where which words are found can be find at the last cell along with the positions. But there is a problem matching the multiwords in the text because the results found in the dataset are inflected. Instead of this we could use the sentences we used to match those multiwords to get estimation of the location of the words, but it still wouldn't be the exact location. There are future improvements required to get better results for this feature.

### **Evaluation of Results**

**Words number: 4 | Skipped words: 3**

- Multi-words that are found with our algorithm.

Query Time: `35.329113721847534` `seconds`

Number of words: 56

Results:

```
'that sustains life on': ['urban life'],
 'for future generations the': ['future generation'],
 'from deforestation to pollution': ['Soil pollution'],
 'of environmental protection lies': ['environmental policy'],
 'burning of fossil fuels': ['fossil fuels'],
 'fossil fuels releases greenhouse': ['fossil fuels'],
 'releases greenhouse gases such': ['greenhouse gas'],
 'to sustainable energy sources': ['sustainable energy'],
 'the environment air pollution': ['Soil pollution'],
 'water sources soil pollution': ['Soil pollution'],
 'soil pollution often caused': ['Soil pollution'],
 'comprehensive environmental education programs': ['environmental education'],
 'enforce stringent environmental regulations': ['environmental education'],
 'environmental regulations that limit': ['environmental education'],
 'deforestation and pollution incentives': ['Soil pollution'],
 'environmentally friendly alternatives international': ['environmental education'],
 'alternatives international cooperation is': ['international cooperation'],
 'environmental issues transcend national': ['environmental education'],
 'ability of future generations': ['future generation'],
 'future generations to meet': ['future generation'],
 'sustainable agriculture and forestry': ['sustainable energy'],
 'can spur economic growth': ['economic growth'],
 'economic growth while minimizing': ['economic growth'],
 'such as national parks': ['national parks', 'national park'],
 'national parks and wildlife': ['national parks', 'national park'],
 'adopting sustainable lifestyle choices': ['sustainable lifestyle'],
 'environmental protection overexploitation of': ['environmental policy'],
 'and collective responsibility that': ['collective responsibility'],
 'current and future generations': ['future generation'],
 'future generations by embracing': ['future generation'],
 'enforcing environmental policies and': ['environmental policy'],
 'of critical thinking creativity': ['critical thinking'],
 'growth of urban life': ['urban life'],
 'urban life during the': ['urban life'],
 'in urban life this': ['urban life'],
 'the printing press which': ['printing press'],
 'of new scientific methods': ['scientific method', 'scientific methods'],
 'scientific methods during the': ['scientific methods', 'scientific method'],
 'the printing press the': ['printing press'],
 'press the printing press': ['printing press'],
 'printing press was one': ['printing press'],
 'the parliamentary system in': ['parliamentary system'],
 'on european culture and': ['European culture'],
 'back to ancient civilizations': ['ancient civilizations',
  'ancient civilization'],
 'ancient civilizations where various': ['ancient civilizations',
  'ancient civilization'],
 'national pride that accompany': ['national park'],
 'sports popularity to drive': ['hard drive'],
 'to drive positive change': ['positive change'],
 'positive change emphasizing values': ['positive change'],
 'on the economic landscape': ['economic landscape'],
 'economic landscape is also': ['economic landscape'],
 'stimulating economic growth despite': ['economic growth'],
 'car a lot of': ['a lot of'],
 'lot of hard drive': ['hard drive'],
 'hard drive i want': ['hard drive'],
 'a lot of hard': ['a lot of']
```

**Words number: 5 | Skipped words: 3**

- Multi-words that are found with our algorithm.

Query Time: `23.40343976020813` `seconds`

Number of words: 55

Results:

```
'fragile entity that sustains life': ['urban life'],
 'sustains life on earth as': ['urban life'],
 'it for future generations the': ['future generation'],
 'worldwide from deforestation to pollution': ['Soil pollution'],
 'of environmental protection lies the': ['environmental policy'],
 'burning of fossil fuels releases': ['fossil fuels'],
 'fuels releases greenhouse gases such': ['greenhouse gas'],
 'transitioning to sustainable energy sources': ['sustainable energy'],
 'the environment air pollution primarily': ['Soil pollution'],
 'water sources soil pollution often': ['Soil pollution'],
 'of responsibility and instigating change': ['positive change'],
 'implement comprehensive environmental education programs': ['environmental education'],
 'enforce stringent environmental regulations that': ['environmental education'],
 'as deforestation and pollution incentives': ['Soil pollution'],
 'friendly alternatives international cooperation is': ['international cooperation'],
 'as environmental issues transcend national': ['environmental education'],
 'transcend national borders and require': ['national park'],
 'environmental protection it entails meeting': ['environmental policy'],
 'the ability of future generations': ['future generation'],
 'future generations to meet their': ['future generation'],
 'from sustainable agriculture and forestry': ['sustainable energy'],
 'can spur economic growth while': ['economic growth'],
 'areas such as national parks': ['national parks', 'national park'],
 'national parks and wildlife reserves': ['national parks', 'national park'],
 'environmental protection adopting sustainable lifestyle': ['environmental policy',
  'sustainable lifestyle'],
 'sustainable lifestyle choices such as': ['sustainable lifestyle'],
 'to environmental protection overexploitation of': ['environmental policy'],
 'environmental degradation adopting circular economy': ['environmental policy',
  'environmental education'],
 'urgent and collective responsibility that': ['collective responsibility'],
 'current and future generations by': ['future generation'],
 'enforcing environmental policies and fostering': ['environmental policy'],
 'the development of critical thinking': ['critical thinking'],
 'critical thinking creativity and self-improvement': ['critical thinking'],
 'the growth of urban life': ['urban life'],
 'urban life during the renaissance': ['urban life'],
 'growth in urban life this': ['urban life'],
 'such as the printing press': ['printing press'],
 'printing press which made it': ['printing press'],
 'new scientific methods during the': ['scientific method',
  'scientific methods'],
 'invention of the printing press': ['printing press'],
 'printing press the printing press': ['printing press'],
 'printing press was one of': ['printing press'],
 'the parliamentary system in england': ['parliamentary system'],
 'impact on european culture and': ['European culture'],
 'to ancient civilizations where various': ['ancient civilizations',
  'ancient civilization'],
 'national pride that accompany these': ['national parks', 'national park'],
 'the sports popularity to drive': ['hard drive'],
 'to drive positive change emphasizing': ['positive change'],
 'football on the economic landscape': ['economic landscape'],
 'economic landscape is also noteworthy': ['economic landscape',
  'economic growth'],
 'creation and stimulating economic growth': ['economic growth'],
 'economic growth despite its widespread': ['economic growth'],
 'car a lot of hard': ['a lot of'],
 'of hard drive i want': ['hard drive'],
 'a lot of hard drives': ['hard drive', 'a lot of']
```

**Words number: 6 | Skipped words: 3**

- Multi-words that are found with our algorithm.

Query Time: `19.406460762023926` `seconds`

Number of words: 48

Results:

```
'that sustains life on earth as': ['urban life'],
 'for future generations the alarming rate': ['future generation'],
 'concerns worldwide from deforestation to pollution': ['Soil pollution'],
 'the heart of environmental protection lies': ['environmental policy'],
 'change the burning of fossil fuels': ['fossil fuels'],
 'fossil fuels releases greenhouse gases such': ['greenhouse gas',
  'fossil fuels'],
 'to sustainable energy sources pollution in': ['Soil pollution',
  'sustainable energy'],
 'threat to the environment air pollution': ['Soil pollution'],
 'on these water sources soil pollution': ['Soil pollution'],
 'soil pollution often caused by improper': ['Soil pollution'],
 'to implement comprehensive environmental education programs': ['environmental education'],
 'enforce stringent environmental regulations that limit': ['environmental education'],
 'such as deforestation and pollution incentives': ['Soil pollution'],
 'environmentally friendly alternatives international cooperation is': ['environmental education',
  'international cooperation'],
 'environmental issues transcend national borders and': ['environmental education'],
 'ability of future generations to meet': ['future generation'],
 'activities from sustainable agriculture and forestry': ['sustainable energy'],
 'eco-friendly solutions can spur economic growth': ['economic growth'],
 'economic growth while minimizing the ecological': ['economic growth'],
 'protected areas such as national parks': ['national parks', 'national park'],
 'national parks and wildlife reserves provides': ['national parks',
  'national park'],
 'environmental protection adopting sustainable lifestyle choices': ['environmental policy',
  'sustainable lifestyle'],
 'substantial positive outcomes when multiplied across': ['positive change'],
 'fundamental to environmental protection overexploitation of': ['environmental policy'],
 'an urgent and collective responsibility that': ['collective responsibility'],
 'current and future generations by embracing': ['future generation'],
 'enforcing environmental policies and fostering a': ['environmental policy'],
 'of critical thinking creativity and self-improvement': ['critical thinking'],
 'today the growth of urban life': ['urban life'],
 'urban life during the renaissance there': ['urban life'],
 'significant growth in urban life this': ['urban life'],
 'such as the printing press which': ['printing press'],
 'of new scientific methods during the': ['scientific method',
  'scientific methods'],
 'invention of the printing press the': ['printing press'],
 'press the printing press was one': ['printing press'],
 'the parliamentary system in england the': ['parliamentary system'],
 'on european culture and society the': ['European culture'],
 'be traced back to ancient civilizations': ['ancient civilizations',
  'ancient civilization'],
 'ancient civilizations where various forms of': ['ancient civilizations',
  'ancient civilization'],
 'fervor and national pride that accompany': ['national park'],
 'leverage the sports popularity to drive': ['hard drive'],
 'to drive positive change emphasizing values': ['positive change'],
 'on the economic landscape is also': ['economic landscape'],
 'creation and stimulating economic growth despite': ['economic growth'],
 'positive impact on societies football continues': ['positive change'],
 'a nice car a lot of': ['a lot of'],
 'lot of hard drive i want': ['hard drive'],
 'mount everest a lot of hard': ['a lot of']
```

**Words number: 8 | Skipped words: 3**

- Multi-words that are found with our algorithm.

Query Time: `11.64547848701477` `seconds`

Number of words: 54

Results:

```
'fragile entity that sustains life on earth as': ['urban life'],
 'protect and preserve it for future generations the': ['future generation'],
 'has raised concerns worldwide from deforestation to pollution': ['Soil pollution'],
 'of environmental protection lies the understanding that the': ['environmental policy'],
 'species disrupting the intricate web of life that': ['urban life'],
 'burning of fossil fuels releases greenhouse gases such': ['greenhouse gas',
  'fossil fuels'],
 'climate change and transitioning to sustainable energy sources': ['sustainable energy'],
 'the environment air pollution primarily caused by industrial': ['Soil pollution'],
 'water sources soil pollution often caused by improper': ['Soil pollution'],
 'of responsibility and instigating change governments non-governmental organizations': ['positive change'],
 'should collaborate to implement comprehensive environmental education programs': ['environmental education'],
 'promoting environmental protection on a larger scale governments': ['environmental policy'],
 'enforce stringent environmental regulations that limit harmful practices': ['environmental policy',
  'environmental education'],
 'harmful practices such as deforestation and pollution incentives': ['Soil pollution'],
 'to embrace environmentally friendly alternatives international cooperation is': ['environmental education',
  'international cooperation'],
 'cooperation is vital as environmental issues transcend national': ['environmental education'],
 'transcend national borders and require a collective effort': ['national park'],
 'the quest for environmental protection it entails meeting': ['environmental policy'],
 'present without compromising the ability of future generations': ['future generation'],
 'future generations to meet their own needs sustainable': ['future generation'],
 'needs sustainable practices encompass a wide range of': ['sustainable energy'],
 'range of activities from sustainable agriculture and forestry': ['sustainable energy'],
 'can spur economic growth while minimizing the ecological': ['economic growth'],
 'and maintaining protected areas such as national parks': ['national parks',
  'national park'],
 'national parks and wildlife reserves provides safe havens': ['national parks',
  'national park'],
 'environmental protection adopting sustainable lifestyle choices such as': ['environmental policy',
  'sustainable lifestyle'],
 'resources is fundamental to environmental protection overexploitation of': ['environmental policy'],
 'resources and accelerates environmental degradation adopting circular economy': ['environmental policy',
  'environmental education'],
 'the environmental impact of resource extraction and consumption': ['environmental policy',
  'environmental education'],
 'environment is an urgent and collective responsibility that': ['collective responsibility'],
 'current and future generations by embracing sustainable practices': ['future generation'],
 'sustainable practices raising awareness enacting and enforcing environmental': ['sustainable energy'],
 'enforcing environmental policies and fostering a sense of': ['environmental policy'],
 'the development of critical thinking creativity and self-improvement': ['critical thinking'],
 'the world today the growth of urban life': ['urban life'],
 'urban life during the renaissance there was a': ['urban life'],
 'was a significant growth in urban life this': ['urban life'],
 'such as the printing press which made it': ['printing press'],
 'profound impact on the development of western civilization': ['ancient civilization'],
 'the development of new scientific methods during the': ['scientific method',
  'scientific methods'],
 'invention of the printing press the printing press': ['printing press'],
 'printing press was one of the most important': ['printing press'],
 'the parliamentary system in england the exploration of': ['parliamentary system'],
 'had a profound impact on european culture and': ['European culture'],
 'impact on the development of western civilization its': ['ancient civilizations',
  'ancient civilization'],
 'be traced back to ancient civilizations where various': ['ancient civilizations',
  'ancient civilization'],
 'national pride that accompany these events are unparalleled': ['national parks',
  'national park'],
 'and charities leverage the sports popularity to drive': ['hard drive'],
 'to drive positive change emphasizing values such as': ['positive change'],
 'the impact of football on the economic landscape': ['economic landscape'],
 'economic landscape is also noteworthy the sport has': ['economic growth',
  'economic landscape'],
 'creation and stimulating economic growth despite its widespread': ['economic growth'],
 'car a lot of hard drive i want': ['a lot of', 'hard drive'],
 'i want to climb mount everest a lot': ['a lot of']
```

**Words number: 10 | Skipped words: 3**

- Multi-words that are found with our algorithm.

Query Time: `8.959564208984375` `seconds`

Number of words: 51

Results:

```
'that sustains life on earth as the custodians of this ': ['urban life'],
 'protect and preserve it for future generations the alarming rate ': ['future generation'],
 'environmental degradation has raised concerns worldwide from deforestation to pollution ': ['Soil pollution',
  'environmental policy',
  'environmental education'],
 'the heart of environmental protection lies the understanding that the ': ['environmental policy'],
 'of biodiversity and contributes to climate change the destruction of ': ['positive change'],
 'the environment is climate change the burning of fossil fuels ': ['fossil fuels'],
 'fossil fuels releases greenhouse gases such as carbon dioxide and ': ['greenhouse gas',
  'fossil fuels'],
 'to sustainable energy sources pollution in its various forms poses ': ['Soil pollution',
  'sustainable energy'],
 'forms poses another significant threat to the environment air pollution ': ['Soil pollution'],
 'human populations that rely on these water sources soil pollution ': ['Soil pollution'],
 'soil pollution often caused by improper disposal of hazardous waste ': ['Soil pollution'],
 'and instigating change governments non-governmental organizations ngos and educational institutions ': ['environmental education'],
 'educational institutions should collaborate to implement comprehensive environmental education programs ': ['environmental education'],
 'promoting environmental protection on a larger scale governments worldwide should ': ['environmental policy'],
 'worldwide should enact and enforce stringent environmental regulations that limit ': ['environmental education'],
 'that limit harmful practices such as deforestation and pollution incentives ': ['Soil pollution'],
 'pollution incentives for sustainable practices such as renewable energy adoption ': ['sustainable energy'],
 'and industries to embrace environmentally friendly alternatives international cooperation is ': ['environmental education',
  'international cooperation'],
 'cooperation is vital as environmental issues transcend national borders and ': ['environmental education'],
 'the quest for environmental protection it entails meeting the needs ': ['environmental policy'],
 'ability of future generations to meet their own needs sustainable ': ['future generation'],
 'needs sustainable practices encompass a wide range of activities from ': ['sustainable energy'],
 'activities from sustainable agriculture and forestry to the promotion of ': ['sustainable lifestyle',
  'sustainable energy'],
 'in green technologies and eco-friendly solutions can spur economic growth ': ['economic growth'],
 'economic growth while minimizing the ecological footprint of human activities ': ['economic growth'],
 'protected areas such as national parks and wildlife reserves provides ': ['national parks',
  'national park'],
 'a crucial role in environmental protection adopting sustainable lifestyle choices ': ['environmental policy',
  'sustainable lifestyle'],
 'behavior can lead to substantial positive outcomes when multiplied across ': ['positive change'],
 'of natural resources is fundamental to environmental protection overexploitation of ': ['environmental policy'],
 'protecting the environment is an urgent and collective responsibility that ': ['collective responsibility'],
 'current and future generations by embracing sustainable practices raising awareness ': ['future generation'],
 'raising awareness enacting and enforcing environmental policies and fostering a ': ['environmental policy'],
 'of critical thinking creativity and self-improvement the rise of individualism ': ['critical thinking'],
 'think about the world today the growth of urban life ': ['urban life'],
 'urban life during the renaissance there was a significant growth ': ['urban life'],
 'significant growth in urban life this was due in part ': ['urban life'],
 'such as the printing press which made it possible to ': ['printing press'],
 'of new scientific methods during the renaissance scientists began to ': ['scientific method',
  'scientific methods'],
 'invention of the printing press the printing press was one ': ['printing press'],
 'the parliamentary system in england the exploration of the new ': ['parliamentary system'],
 'on european culture and society the renaissance was a period ': ['European culture'],
 'be traced back to ancient civilizations where various forms of ': ['ancient civilizations',
  'ancient civilization'],
 'forms of the sport were played in different cultures however ': ['European culture'],
 'their cultural diversity the fervor and national pride that accompany ': ['national park'],
 'discrimination organizations and charities leverage the sports popularity to drive ': ['hard drive'],
 'to drive positive change emphasizing values such as teamwork discipline ': ['positive change'],
 'on the economic landscape is also noteworthy the sport has ': ['economic growth',
  'economic landscape'],
 'creation and stimulating economic growth despite its widespread popularity football ': ['economic growth'],
 'positive impact on societies football continues to capture the hearts ': ['positive change'],
 'model ferrari it is a nice car a lot of ': ['a lot of'],
 'lot of hard drive i want to climb mount everest ': ['a lot of',
  'hard drive']
```

- When we compare the results we obtained, we can see that the query times get longer and shorter depending on the number of words we choose. For example, when the word length is 4, our query time is approximately 35 seconds. As the number of words increases, the query time begins to decrease. For example, we can observe that it takes 23 seconds for 5 words, 19 seconds for 6 words, 12 seconds for 8 words and 9 seconds for 10 words.
- In all results, we got 0 matches using the $in query with the multi-words. The reason for that is we have the options 4, 5, 6, 8 and 10, and inside the dataset we only have multi-words with the length of 2 and 3. This query would be useful if we added some multi-words with the same length of sentence length we choose, or if we added sentence length options 2 and 3.
- When we look at the number of correct matches, despite the long query time, the queries we make with the fewest words result in more accurate matches. For example, when we choose the number of words as 4, we can see that 48 of 56 words are matched correctly. For word count 5, this results in correct matching of 38 out of 55 words. For 6 words, 38 out of 48 words were matched correctly. For 8 words, 30 out of 54 words were matched correctly. For 10 words, only 28 out of 51 words were matched correctly.
- Query times and correct result rates are given in the table below.

| Number of words | Query Time (in seconds) | Words Found | Correct Matches | Ratio(%) |
| --- | --- | --- | --- | --- |
| 4 | 35 | 56 | 48 | 86 |
| 5 | 23 | 55 | 38 | 69 |
| 6 | 19 | 48 | 38 | 79 |
| 8 | 11 | 54 | 30 | 56 |
| 10 | 9 | 51 | 28 | 55 |

### Workload Distribution 
Mehmet Akif Arican performed research on datasets and tools to use on the project. He did some basic tests and made improvements to the project. He performed manual corrections on the data used on the project.

Emirhan Aykan performed literature research to figure out which path we would follow during the project. He created the most essential parts of the projects such as determining the methods and writing the algorithms. He did tests on different data and performed a comparison of the results. He prepared the documentation and made the presentations.

> "We shall not cease from exploration, and the end of all our exploring will be to arrive where we started and know the place for the first time." - T.S Eliot

This quote means a lot to me and in different ways while working on building my first official 'data science' prediction mode a.k.a - Project Luther. To me, data science is an art of smart exploration, and we all need to come back to where we started to update our knowledge about the place.

Project Luther is a project where we use Regression model to predict something we choose, and also collect our own data by scraping website(s). It is actually a pretty exciting project to me that I finally get to build a 'forecast' model by not using Excel but by programming!

I don't know much about Luther, but I certainly wish I could be Sherlock, that way I can solve my case easily with 100% accuracy. Now comes the project story...

## Scraping and Cleaning Data
When the power of BeautifulSoup and Selenium was first demonstrated in class, I secrectly thought it must be a lot of fun to be a hacker. This thought only survied till I started doing the real scraping myself. I had to give up my initial project idea because the lack of available data. For someone like me who was used to being drowned by massive amount of data, it's a momment of realization of the unbalanced public data availability in different areas. I settled on diving into the popular movie topic, and picked my own angle of prediction - IMDB rating prediction.

If sniffing through a website's html data structure to get the needed data is hard (the site structure changed on the 2 days I was sraping my data), wait till you have to prepare your data for modeling. There are many unexpected problems to be solved, and of course I learned quite few useful tricks. For example, using 'eval' to evaluate data type in Pandas DataFrame. And the way to unpack data cell that has multiple values to rows and merge the unpacked DataFrame back with the main DataFrame. Code example:

```python
# unpack director column
director = pd.DataFrame([[i,x] for i,y in df_train['director'].apply(list).items() for x in y],columns=['I','director'])
director.set_index('I')
df_director = data_df.merge(director,left_index=True,right_on='I')
```
## Feature Engineering
This is the most important stage for building the models. It is a challenge for my project because most of the features I have are categorical, as directors, actors, writers, production compaines. Conceptually, I know these would be features can be used to predict IMDB rating, but I need to figure out a way of converting them to features I can use for a regression model. I believe I made the right decision of using historical IMDB rating in some way to re-engineer the features, and I even thought about the common issue of data leakage by carefully holding out a portion of most recent data. However, I still faced data leakage issue, which I didn't realize until after I fit my models. My RandomForrest model got me a R^2 of 0.998. I knew right away this is too good to be true. With some investigation, it was found out that I made some mistakes in the process of re-engineering features:
1. I included all the movies into the rating assignment calcuation. Included the to-be-predicted data point to training data
2. I used the rating data to generate features even in holdout data set, so I'm using future data to predict future data
3. Because I unpacked directors, actors, writers etc. where there were multiple of them for the same movie to different rows, my movie title is no longer unqiue in the data set. When I did the train, test split, training data can also leak into test data

Now I learned few different ways of causing **data leakage**! How about know it all, so here goes a longer list from Kaggle:
* Leaking test data into the training data.
* Leaking the correct prediction or ground truth into the test data.
* Leaking of information from the future into the past.
* Retaining proxies for removed variables a model is restricted from knowing.
* Reversing of intentional obfuscation, randomization or anonymization.
* Inclusion of data not present in the model's operational environment.
* Distorting information from samples outside of scope of the model's intended use.
* Any of the above present in third party data joined to the training set.

Also, there is another inherent problem with the data set:
Only about 20-25% directors, actors, and writers had 3 or above movies in the data set, so by using this data set to predict rating for specific movie, the model pretty much remembered most movies.

## Modeling Results
Due to the time limitation, I did not have time to fix the mistakes I made to the dataset. I had to limit the few numerical features I have, as Year, Month, Day, Weekday, runtime, Domestic Gross, and Budget. And the Oscar goes to..........no one. I tried different models by using different combinations of these features. The best one is a 3 degree Polynomial model with Year, Month, Weekday, and Budget, and the test R^2 is only 0.06.

Even though I did not find a model that can predict IMDB rating, I enjoyed the learning of different ways of doing feature seletion and engineering.

## Fun Facts about Movies
1. Worst movies are usually pushed out in January
2. Most movies are released on Fridays (you might have already known)
3. IMDB rating is slightly negative correlated with budget
4. The director has the highest gross per movie is John Dahl (from 1970 to 2017)

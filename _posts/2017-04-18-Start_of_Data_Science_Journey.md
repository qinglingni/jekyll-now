# Climbing the Data Science Mountain
> "It’s not that I’m so smart, it’s just that I stay with problems longer." -Albert Einstein

Choosing to seriously take on data science journey is no small decision for me, not only because I have been on a good career path, but also because I know I almost have to start from scratch on every technical skills required by data science discipline. It's like I am going to climb the Hymalayas when I am not adequately equipped. But here I am, starting the climb because I believe there will be a lot of rewards are waiting for me along the road and on the top.

![alt tag](https://github.com/qinglingni/qinglingni.github.io/blob/master/IMG_3507.jpg)

Can you predict how the week would look like when you see this face on Monday morning? It's going to be great! Well, that's when you don't have enough observations to make the prediction (BTW, he is our puppy - Epsilon, who just joined the family on Sunday - April 9th). As the week went on, there were two words dominated the theme: Sleepless and Overwhelmed. But... did I say there will be rewards on the way up? It was very rewarding to put the presentation together with my teammates for Project Benson. The process allowed us to have a peek on data science project in real life. It is about collecting data, understanding data, forming hypothesis or business ideas, cleaning and analyzing data, presenting results, and repeating the process to fine tune models. The goal of Project Benson is to use NYC MTA data as basis to come up with an business idea to pitch, and of course pitch with the data insights. I will cover the data science learning below

And Epsilon brought so much joy into our life even though he still pees and poops all over the place. He is pretty much my decompression therapist, and what makes me really proud is he can now sit and stay on command!

## Some Data Science

Okay, what data science skills did I learn in the first week through Project Benson
### 1. Data Collection
Project Benson requires us to download multiple .txt files from mta website. In this case, I wrote a Python script that allows me to download multiple files at once and combine them to one dataframe to save time. The key part of the learning is how to utilize Python datetime package to convert datetime to string, and does datetime calculation. It's important just because datetime operations are very different in different languages.

For example in below code, **datetime.strptime** converts datestring to desired date format, and **.date()** extracts the date portion. **timedelta** in datetime package performs the date calculation, and **date.weekday()** returns the day of week value from 0 to 6 with 0 being Monday, and 6 being Sunday.
```
start_date = datetime.strptime(start_date, '%Y-%m-%d').date()
start_date = start_date + timedelta(days = (5 - start_date.weekday()))
```
### 2. Data Transformation
MTA entry and exit data is presented in cumulative form in 4-hour time interval, meaning calculation needs to be done in order to get number of entries and exits for a particular day. In order to do the calculation, we need to reorganize the data in the dataframe so that we can use second day first entry number minus first day first entry number to get the net entries for first day. The code for doing this part is as below:
```
#Get only the first entry data for the combined key of ["C/A", "UNIT", "SCP", "STATION", "DATE"]
turnstiles_daily = df.groupby(["C/A", "UNIT", "SCP", "STATION", "DATE"]).ENTRIES.first().reset_index()

#Shift previous-day data up to be ready for dataframe calculation
turnstiles_daily[["PREV_DATE", "PREV_ENTRIES"]] = (turnstiles_daily
                                                       .groupby(["C/A", "UNIT", "SCP", "STATION"])["DATE", "ENTRIES"]
                                                       .transform(lambda grp: grp.shift(1)))
turnstiles_daily.dropna(subset=["PREV_DATE"], axis=0, inplace=True)

#Define a function to do the calculation and also manage data exceptions
def get_daily_counts(row, max_counter):
    counter = row["ENTRIES"] - row["PREV_ENTRIES"]
    if counter < 0:
        # Maybe counter is reversed?
        counter = -counter
    if counter > max_counter:
        print(row["ENTRIES"], row["PREV_ENTRIES"])
        counter = min(row["ENTRIES"], row["PREV_ENTRIES"])
    if counter > max_counter:
        # Check it again to make sure we are not giving a counter that's too big
        return 0
    return counter

# If counter is > 1Million, then the counter might have been reset.  
# Just set it to zero as different counters have different cycle limits
turnstiles_daily["DAILY_ENTRIES"] = turnstiles_daily.apply(get_daily_counts, axis=1, max_counter=1000000)
```
### 3. Data Visulization
With any data analytics, visualization is always key either to do data discovery or to communicate results to your audience. For the business proposal our team came up with, time series plot of subway station data is very important for us to identify where opportunities lie. We want to use recent trend to predict if it's a good time for a gym company or a food truck company to enter certain neighborhood(s). I created a function to allow me to plot time series (trend) for any given station:
```
data = pd.DataFrame(turnstiles_daily[['STATION','DATE','DAILY_ENTRIES']])
def plot_date(stationname):
    plot_data = data.set_index('STATION').loc[stationname]
    plot_data = plot_data.groupby('DATE').DAILY_ENTRIES.sum().reset_index()
    plt.figure(figsize=(10,2))
    x = plot_data['DATE'].map(lambda x: dateutil.parser.parse(x))
    y = plot_data.DAILY_ENTRIES    
    return plt.plot(x,y)
plot_date('59 ST')
```
**_ Below is the sample output of one station's entry trend of 2 weeks_**
![alt_tag]https://github.com/qinglingni/qinglingni.github.io/blob/master/mta_timeseries.png

## Climbing Countinues
Time went by fast and slow for the first week, there is a huge amount of new knowledge poured in. I am not claiming I understand all, but as Einstein said: "It's not that I am so smart, it’s just that I stay with problems longer". I really enjoys what you can uncover from the data out in the wild world, and I will enjoy all the awards along my climb up to the data science peak. And hopefully I can cracking down data in my sleep like:
![alt_tag]https://github.com/qinglingni/qinglingni.github.io/blob/master/IMG_3520.JPG

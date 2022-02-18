# Brazilian Football League (2003-2021) EDA

![alt text](http://s.glbimg.com/es/ge/f/original/2012/11/18/festa_fred_fluminense3_andur.jpg)

This is an article about the history of the Brazilian Football League's first division between the year of 2003 and 2021. 

In 2003 the tournament changed its system to a _**round-robin**_ tournament (or all-play-all tournament), which is a competition in which each team meets every other participant. Before the round-robin system, the Brazilian Football League was played in a cup system, with a group stage - where only a few teams would get qualified to the next round - followed by a knock-out stage - where the teams that lost would get eliminated.

In the current system, at the end of a match, the winning team is rewarded with three points and the losing team with zero. If the match ends up as a tie, both teams are rewarded with one point.

The team that accumulates the highest sum of points will automatically be the champion. The bottom four teams are automatically relegated to the second division.

## Cleaning and Formatting Data

For this analysis, two datasets were utilized. The first provided data from the seasons of 2003-2020. For the most up-to-date analysis, the second dataset was scraped from the web, with the data of the season of 2021.



```
# Checking names of columns
head(br)

# Dropping unused columns
br = br[-c(3:4,7:8,11:13)]

# Changing (translating) column names
colnames(br) = c('Round','Season','Home','Away','HG','AG')

# Extracting the year of the date
br$Season = as.numeric(str_sub(br$Season, -4, -1))

# Part of the 2020 Season was played in 2021, due to COVID. So it is necessary
# to fix the season year
br$Season[br$Season == 2021] = 2020

# Cleaning HOME names

br$Home = gsub('Ã£', 'a', br$Home)
br$Home = gsub('Ã©', 'e', br$Home)
br$Home = gsub('Ã¡', 'a', br$Home)
br$Home = gsub('Ãº', 'u', br$Home)
br$Home = gsub('Ãª', 'e', br$Home)
br$Home = gsub('Ã³', 'o', br$Home)
br$Home = gsub('Ã', 'i', br$Home)

br$Home = str_to_title(tolower(br$Home))


# Cleaning HOME names

br$Away = gsub('Ã£', 'a', br$Away)
br$Away = gsub('Ã©', 'e', br$Away)
br$Away = gsub('Ã¡', 'a', br$Away)
br$Away = gsub('Ãº', 'u', br$Away)
br$Away = gsub('Ãª', 'e', br$Away)
br$Away = gsub('Ã³', 'o', br$Away)
br$Away = gsub('Ã', 'i', br$Away)

br$Away = str_to_title(tolower(br$Away))

# Function to determine if Home Team or Away Team won the match
resFunc = function (HG, AG){
    if (HG > AG)
        br$Result = 'H'
    else if (HG < AG)
        br$Result = 'A'
    else
        br$Result = 'D'
}

# Assingning results to new column
br$Result = mapply(resFunc, br$HG, br$AG)

# Exporting clean table
# write.csv(br, 'BRA-03-20_clean.csv')

# Since this dataset only covers the seasons of 2003 until 2020, we will add
# the data for the season of 2021 now, which is already formatted
br = rbind(br,br21)

```
{
   "cell_type": "code",
   "execution_count": 288,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Unnamed: 0</th>\n",
       "      <th>Unnamed: 1</th>\n",
       "      <th>Unnamed: 2</th>\n",
       "      <th>Unnamed: 3</th>\n",
       "      <th>Puntos</th>\n",
       "      <th>PJ</th>\n",
       "      <th>PG</th>\n",
       "      <th>PE</th>\n",
       "      <th>PP</th>\n",
       "      <th>GF</th>\n",
       "      <th>GC</th>\n",
       "      <th>TA</th>\n",
       "      <th>TR</th>\n",
       "      <th>season</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>NaN</td>\n",
       "      <td>1</td>\n",
       "      <td>NaN</td>\n",
       "      <td>Bayern München</td>\n",
       "      <td>82</td>\n",
       "      <td>34</td>\n",
       "      <td>26</td>\n",
       "      <td>4</td>\n",
       "      <td>4</td>\n",
       "      <td>100</td>\n",
       "      <td>32</td>\n",
       "      <td>52</td>\n",
       "      <td>3</td>\n",
       "      <td>2019-20</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>NaN</td>\n",
       "      <td>2</td>\n",
       "      <td>NaN</td>\n",
       "      <td>Borussia Dortmund</td>\n",
       "      <td>69</td>\n",
       "      <td>34</td>\n",
       "      <td>21</td>\n",
       "      <td>6</td>\n",
       "      <td>7</td>\n",
       "      <td>84</td>\n",
       "      <td>41</td>\n",
       "      <td>48</td>\n",
       "      <td>1</td>\n",
       "      <td>2019-20</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>NaN</td>\n",
       "      <td>3</td>\n",
       "      <td>NaN</td>\n",
       "      <td>RB Leipzig</td>\n",
       "      <td>66</td>\n",
       "      <td>34</td>\n",
       "      <td>18</td>\n",
       "      <td>12</td>\n",
       "      <td>4</td>\n",
       "      <td>81</td>\n",
       "      <td>37</td>\n",
       "      <td>50</td>\n",
       "      <td>3</td>\n",
       "      <td>2019-20</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>NaN</td>\n",
       "      <td>4</td>\n",
       "      <td>NaN</td>\n",
       "      <td>Borussia Mönchengladbach</td>\n",
       "      <td>65</td>\n",
       "      <td>34</td>\n",
       "      <td>20</td>\n",
       "      <td>5</td>\n",
       "      <td>9</td>\n",
       "      <td>66</td>\n",
       "      <td>40</td>\n",
       "      <td>74</td>\n",
       "      <td>3</td>\n",
       "      <td>2019-20</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>NaN</td>\n",
       "      <td>5</td>\n",
       "      <td>NaN</td>\n",
       "      <td>Bayer Leverkusen</td>\n",
       "      <td>63</td>\n",
       "      <td>34</td>\n",
       "      <td>19</td>\n",
       "      <td>6</td>\n",
       "      <td>9</td>\n",
       "      <td>61</td>\n",
       "      <td>44</td>\n",
       "      <td>58</td>\n",
       "      <td>6</td>\n",
       "      <td>2019-20</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   Unnamed: 0  Unnamed: 1  Unnamed: 2                Unnamed: 3 Puntos  PJ  \\\n",
       "0         NaN           1         NaN            Bayern München     82  34   \n",
       "1         NaN           2         NaN         Borussia Dortmund     69  34   \n",
       "2         NaN           3         NaN                RB Leipzig     66  34   \n",
       "3         NaN           4         NaN  Borussia Mönchengladbach     65  34   \n",
       "4         NaN           5         NaN          Bayer Leverkusen     63  34   \n",
       "\n",
       "   PG  PE  PP   GF  GC  TA  TR   season  \n",
       "0  26   4   4  100  32  52   3  2019-20  \n",
       "1  21   6   7   84  41  48   1  2019-20  \n",
       "2  18  12   4   81  37  50   3  2019-20  \n",
       "3  20   5   9   66  40  74   3  2019-20  \n",
       "4  19   6   9   61  44  58   6  2019-20  "
      ]
    

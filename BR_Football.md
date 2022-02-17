# Brazilian Football League (2003-2021) EDA

## Exploratory Data Analysis

## Cleaning and Formatting Data

The dataset needs cleaning and translating

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

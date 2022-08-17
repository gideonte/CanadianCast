# Where do Canadian Film Actors and Actresses Come from?


Concept & Design by Gideon Msambwa

April 7, 2021

## Topic

This report will present data visualization of where the Canadian film casts are coming from. This report will help individuals seeking to meet with different film actors for mentorship and learn from them. The word casts is used in this visualization to present actors and actresses.

<img width="1158" alt="Casts" src="https://user-images.githubusercontent.com/8546504/183376867-5cadfe8c-6bd5-4372-9a90-c3c17de66724.png">

## Sources 
This report will use the Internet Movie Database (IMDB) for film casts (https://www.kaggle.com/stefanoleone992/imdb-extensive-dataset) while geographical names data delivered from the Canadian Geographical Names Database. (https://www.nrcan.gc.ca/earth-sciences/geography/download-geographical-names-data/9245). and Shapefile comes from Natural Earth Data http://naciscdn.org/naturalearth/packages/natural_earth_vector.zip

## Obtaining  Data
First, we will obtain data of Casts Names and Locations.

```{r load libraries and obtain data, message=FALSE}
library(tidyverse)

IMDB_names <- read_csv("IMDb names.csv")
locations <- read_csv("cgn_canada_csv_eng.csv")
```

### Formatting the Cast and Location Data

The only columns I am interested in from the IMDB_names data frames are

* Cast Name
* Date of Birth
* Place of Birth

```{r Clean up and summarize data}
cast <- select(IMDB_names,
                name,
                date_of_birth,
                place_of_birth)
```

The goal is to have Canadian casts, so I will filter the _place_of_birth_ column to contain only Canada value.

```{r Filter for Canadian Cast}

canadian_cast <- filter(cast, str_detect(place_of_birth, "Canada")) %>%
  separate(place_of_birth, c("Geographical_Name", "Province", "Country"), ",")%>%
  mutate(Province = trimws(Province))%>%
  mutate(Province = gsub("C)","e",iconv(Province, 'ASCII//TRANSLIT'), fixed = TRUE))

```

Also, we will summarize the location data frame.

The column names are inconvenient, so I replace them here with names that are simpler and do not have spaces (which makes the syntax much simpler).

```{r change colnames, message=FALSE, results=FALSE, warning=FALSE}
colnames(locations) <- c("CGNDB_ID",
                      "Geographical_Name",
                      "Language",
                      "Syllabic_Form",
                      "Generic_Term",
                      "Generic_Category",
                      "Concise_Code",
                      "Toponymic",
                      "lat",
                      "long",
                      "Location",
                      "Province",
                      "Relevance_Scale",
                      "Decision_Date",
                      "Source")
```

The only columns I am interested in from the Canada_locations data frames are

* Geographical Name
* Latitude
* Longitude
* Province

```{r Clean up and summarize data of Canada Locations and Provinces}
canada_locations<- select(locations,
                Geographical_Name,
                lat,
                long,
                Province)

ca_provinces <- select(canada_locations,
                       Province,
                       long,
                       lat) %>%
  distinct(Province, .keep_all = TRUE)
```

Then Joining the _canadian_cast_ data with _canada_locations_ data

```{r final combination}
cast_and_location <- left_join(canadian_cast,
                                canada_locations,
                                by = c("Geographical_Name","Province"))%>%
  distinct(name, .keep_all = TRUE)%>%
  drop_na()
```

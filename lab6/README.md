Lab 06 - Regular Expressions and Web Scraping
================

# Learning goals

- Use a real world API to make queries and process the data.
- Use regular expressions to parse the information.
- Practice your GitHub skills.

# Lab description

In this lab, we will be working with the [NCBI
API](https://www.ncbi.nlm.nih.gov/home/develop/api/) to make queries and
extract information using XML and regular expressions. For this lab, we
will be using the `httr`, `xml2`, and `stringr` R packages.

This markdown document should be rendered using `github_document`
document ONLY and pushed to your *JSC370-labs* repository in
`lab06/README.md`.

## Question 1: How many sars-cov-2 papers?

Build an automatic counter of sars-cov-2 papers using PubMed. You will
need to apply XPath as we did during the lecture to extract the number
of results returned by PubMed in the following web address:

    https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2

Complete the lines of code:

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/div[1]/span")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "192,677"

``` r
stringr::str_extract(counts, "[\\d,]+")
```

    ## [1] "192,677"

- How many sars-cov-2 papers are there?

*There are 192677 sars-cov-2 papers.*

Don’t forget to commit your work!

## Question 2: Academic publications on COVID19 and Hawaii

Use the function `httr::GET()` to make the following query:

1.  Baseline URL:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

2.  Query parameters:

    - db: pubmed
    - term: covid19 hawaii
    - retmax: 1000

The parameters passed to the query are documented
[here](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

``` r
library(httr)
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(
    db = "pubmed",
    term = "covid19 hawaii",
    retmax = 1000
  )
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
#ids <- content(query_ids)
```

The query will return an XML object, we can turn it into a character
list to analyze the text directly with `as.character()`. Another way of
processing the data could be using lists with the function
`xml2::as_list()`. We will skip the latter for now.

Take a look at the data, and continue with the next question (don’t
forget to commit and push your results to your GitHub repo!).

## Question 3: Get details about the articles

The Ids are wrapped around text in the following way:
`<Id>... id number ...</Id>`. we can use a regular expression that
extract that information. Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>\\d+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
```

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

1.  Baseline url:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

2.  Query parameters:

    - db: pubmed
    - id: A character with all the ids separated by comma, e.g.,
      “1232131,546464,13131”
    - retmax: 1000
    - rettype: abstract

**Pro-tip**: If you want `GET()` to take some element literal, wrap it
around `I()` (as you would do in a formula in R). For example, the text
`"123,456"` is replaced with `"123%2C456"`. If you don’t want that
behavior, you would need to do the following `I("123,456")`.

``` r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    retmax = 1000,
    rettype = "abstract",
    id = paste(ids, collapse = ",")
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

With this in hand, we can now analyze the data. This is also a good time
for committing and pushing your work!

## Question 4: Distribution of universities, schools, and departments

Using the function `stringr::str_extract_all()` applied on
`publications_txt`, capture all the terms of the form:

1.  University of …
2.  … Institute of …

Write a regular expression that captures all such instances

``` r
institution <- stringr::str_extract_all(
  publications_txt,
  "University of [\\w\\-\\.\\s]+|[\\w]+ Institute of [\\w\\-\\.\\s]+"
  ) 
institution <- unlist(institution)
as.data.frame(table(institution))
```

    ##                                                                                                                                                                                          institution
    ## 1                                                                                                                                                                         Army Institute of Research
    ## 2                                                                                                                                               Australian Institute of Tropical Health and Medicine
    ## 3                                                                                                                                                   Beijing Institute of Pharmacology and Toxicology
    ## 4                                                                                                                                                                         Berlin Institute of Health
    ## 5                                                                                                                                     Breadfruit Institute of the National Tropical Botanical Garden
    ## 6                                                                                                                                                                 Broad Institute of Harvard and MIT
    ## 7                                                                                                                                                               Cancer Institute of Emory University
    ## 8                                                                                                                                                                     Cancer Institute of New Jersey
    ## 9                                                                                                                                                                Davis Institute of Health Economics
    ## 10                                                                                                                                                                     Genome Institute of Singapore
    ## 11                                                                                                                                                      Graduate Institute of Rehabilitation Science
    ## 12                                                                                                                                                                  Health Institute of Montpellier 
    ## 13                                                                                                                                                             Heidelberg Institute of Global Health
    ## 14                                                                                                                                              Higher Institute of Health. COVID-19 epidemic. https
    ## 15                                                                                                                                                                     i Institute of Marine Biology
    ## 16                                                                                                                                                          Indian Institute of Tropical Meteorology
    ## 17                                                                                                                                         Leeds Institute of Rheumatic and Musculoskeletal Medicine
    ## 18                                                                                                              Massachusetts Institute of Technology Koch Institute for Integrative Cancer Research
    ## 19                                                                                                                                                                      Mayo Institute of Technology
    ## 20                                                                                                                                                       Medanta Institute of Education and Research
    ## 21                                                                                                                                                           Mediterranean Institute of Oceanography
    ## 22                                                                                                                                                                  MGM Institute of Health Sciences
    ## 23                                                                                                                                            Monterrey Institute of Technology and Higher Education
    ## 24                                                                                                                                             National Institute of Allergy and Infectious Disease 
    ## 25                                                                                                                                             National Institute of Allergy and Infectious Diseases
    ## 26                                                                                               National Institute of Allergy and Infectious Diseases. U.S. Department of Health and Human Services
    ## 27                                                                                                                                       National Institute of Biomedical Imaging and Bioengineering
    ## 28                                                                                                                                                National Institute of Biostructures and Biosystems
    ## 29                                                                                                                                               National Institute of Environmental Health Sciences
    ## 30                                                                                                                                                   National Institute of General Medical Sciences 
    ## 31                                                                                                                                National Institute of Infectious Diseases website.  Available from
    ## 32              National Institute of Neurological Disorders and Stroke - National Institutes of Health Low back pain fact sheet for patients and the public. J Pain Palliat Care Pharmacother. 2004
    ## 33                                                                                                                                                               National Institute of Public Health
    ## 34                                                                                                                                                      National Institute of Technology Gumi Korea.
    ## 35                                                                                                                                        Nordic Institute of Chiropractic and Clinical Biomechanics
    ## 36                                                                                                                                                          Prophylactic Institute of Southern Italy
    ## 37                                                                                                                                                                 Research Institute of New Zealand
    ## 38                                                                                                                                             Research Institute of Tuberculosis and Lung Diseases 
    ## 39                                                                                                                                                                 Swiss Institute of Bioinformatics
    ## 40                                                                                                                                  the Institute of Biomedical Sciences and School of Life Sciences
    ## 41                                                                                                                                                                     the Institute of Medicine as 
    ## 42                                                                                                                                                                             University of Alabama
    ## 43                                                                                                                                                                             University of Alberta
    ## 44                                                                                                                                                                    University of Applied Sciences
    ## 45                                                                                                                                                     University of Applied Sciences Mainz Germany.
    ## 46                                                                                                                                                 University of Applied Sciences Rosenheim Germany.
    ## 47                                                                                                                                                                             University of Arizona
    ## 48                                                                                                                                                                       University of Arizona Press
    ## 49                                                                                                                                                                            University of Arkansas
    ## 50                                                                                                                                                       University of Arkansas for Medical Sciences
    ## 51                                                                                                                                             University of Arkansas for Medical Sciences Northwest
    ## 52                                                                                                                                                                               University of Basel
    ## 53                                                                                                                                                           University of Benin Benin City Nigeria.
    ## 54                                                                                                                                                                            University of Botswana
    ## 55                                                                                                                                                                            University of Bradford
    ## 56                                                                                                                                                                             University of Bristol
    ## 57  University of Bristol. Dr Berry reported receiving grants from Berry Consultants. Dr Derde reported being a member of the COVID-19 guideline committee for the Society of Critical Care Medicine
    ## 58                                                                                                                                                                    University of British Columbia
    ## 59                                                                                                                                                 University of British Columbia School of Medicine
    ## 60                                                                                                                                                                             University of Calgary
    ## 61                                                                                                                                                                          University of California
    ## 62                                                                                                                         University of California Cancer Consortium experience. J. Clin. Oncol. 38
    ## 63                                                                                                                                                                    University of California Davis
    ## 64                                                                                                                                        University of California Davis Comprehensive Cancer Center
    ## 65                                                                                                                                                     University of California Davis Medical Center
    ## 66                                                                                                                                                             University of California Los Angeles 
    ## 67                                                                                                                                    University of California Los Angeles Geffen School of Medicine
    ## 68                                                                                                                                                                    University of California Press
    ## 69                                                                                                                                                                University of California Riverside
    ## 70                                                                                                                                                                University of California San Diego
    ## 71                                                                                                                                             University of California San Diego School of Medicine
    ## 72                                                                                                                                                            University of California San Francisco
    ## 73                                                                                                                                         University of California San Francisco School of Medicine
    ## 74                                                                                                                                                           University of California San Francisco.
    ## 75                                                                                                                                                                           University of Cambridge
    ## 76                                                                                                                                                      University of Campinas Piracicaba SP Brazil.
    ## 77                                                                                                                                                                             University of Chicago
    ## 78                                                                                                                                                              University of Chicago Medical Center
    ## 79                                                                                                                                                                    University of Chicago Medicine
    ## 80                                                                                                                                                                       University of Chicago Press
    ## 81                                                                                                                                                 University of Chicago Pritzker School of Medicine
    ## 82                                                                                                                                                         University of Chinese Academy of Sciences
    ## 83                                                                                                                                                                          University of Cincinnati
    ## 84                                                                                                                                                            University of Cincinnati Cancer Center
    ## 85                                                                                                                                                                            University of Colorado
    ## 86                                                                                                                                                                     University of Colorado Denver
    ## 87                                                                                                                                                         University of Colorado School of Medicine
    ## 88                                                                                                                                                                         University of Connecticut
    ## 89                                                                                                                                                                          University of Copenhagen
    ## 90                                                                                                                                                                             University of Córdoba
    ## 91                                                                                                                                                             University of Dayton Dayton Ohio USA.
    ## 92                                                                                                                                                                  University of Education Freiburg
    ## 93                                                                                                                                                                              University of Exeter
    ## 94                                                                                                                                                                            University of Florence
    ## 95                                                                                                                                                                             University of Florida
    ## 96                                                                                                                                                                             University of Georgia
    ## 97                                                                                                                                                                               University of Ghana
    ## 98                                                                                                                                                                             University of Granada
    ## 99                                                                                                                                                                              University of Guilan
    ## 100                                                                                                                                                                              University of Haifa
    ## 101                                                                                                                                                                              University of Hawai
    ## 102                                                                                                                                                                   University of Hawai i at Mānoa
    ## 103                                                                                                                        University of Hawaiʻi Cancer Center and Hawaiʻi Tumor Registry. 2016. Dec
    ## 104                                                                                                                                                                             University of Hawaii
    ## 105                                                                                                                                                                            University of Hawaii 
    ## 106                                                                                                                                                                   University of Hawaii - System.
    ## 107                                                                                                                                       University of Hawaii Actions to Address COVID-19 Pandemic 
    ## 108                                                                                                                                                            University of Hawaii and Z Consulting
    ## 109                                                                                                                                                                     University of Hawaii at Hilo
    ## 110                                                                                                                                                                    University of Hawaii at Manoa
    ## 111                                                                                                                                                                    University of Hawaii At Manoa
    ## 112                                                                                                                                                                    University of Hawaii at Mānoa
    ## 113                                                                                                                                                                    University of Hawaìi at Mānoa
    ## 114                                                                                                                                                                   University of Hawaii at Mānoa 
    ## 115                                                                                                                                                   University of Hawaii at Manoa Honolulu HI USA.
    ## 116                                                                                                                                    University of Hawaii at Manoa John A Burns School of Medicine
    ## 117                                                                                                                                   University of Hawaii at Manoa John A. Burns School of Medicine
    ## 118                                                                                                                                    University of Hawaii at Manoa Office of Public Health Studies
    ## 119                                                                                                                                                                   University of Hawaii at Manoa.
    ## 120                                                                                                                                                                    University of Hawaii at Manon
    ## 121                                                                                                                                                               University of Hawaii Cancer Center
    ## 122                                                                                                                        University of Hawaii Cancer Center \nThe multiethnic cohort. Available at
    ## 123                                                                                                                                   University of Hawaii Cancer Center and Department of Pathology
    ## 124                                                                                                                                              University of Hawaii Economic Research Organization
    ## 125                                                                                                                                             University of Hawaii Economic Research Organization 
    ## 126                                                                                                                                                                  University of Hawaiï i at Mãnoa
    ## 127                                                                                                                                            University of Hawaii John A. Burns School of Medicine
    ## 128                                                                                                                                                            University of Hawaii Law Review. 2015
    ## 129                                                                                                                                                  University of Hawaii Manoa Honolulu Hawaii USA.
    ## 130                                                                                                                                                                       University of Hawaii News 
    ## 131                                                                                                                                                                       University of Hawaii Press
    ## 132                                                                                                                                                                University of Hawaii Press.\nhttp
    ## 133                                                                                                                                                          University of Hawaii School of Medicine
    ## 134                                                                                                                                University of Hawaii System. 2019. \naccessed 2020 Jan\n28. https
    ## 135                                                                                                                                                                       University of Hawaii-Manoa
    ## 136                                                                                                                                                                       University of Hawaii-Mānoa
    ## 137                                                                                                                                                                            University of Hawaii.
    ## 138                                                                                                                                    University of Health and Welfare Graduate School Tokyo Japan.
    ## 139                                                                                                                     University of Health and Welfare School of Medicine  Guidebook 2023.   https
    ## 140                                                                                                                                                                    University of Health Sciences
    ## 141                                                                                                                                                        University of Health Sciences in Bethesda
    ## 142                                                                                                                                                                          University of Hong Kong
    ## 143                                                                                                                                                                           University of Honolulu
    ## 144                                                                                                                                                                  University of Honolulu at Manoa
    ## 145                                                                                                                                                                             University of Ibadan
    ## 146                                                                                                                                                                           University of Illinois
    ## 147                                                                                                                                                                   University of Illinois Chicago
    ## 148                                                                                                                                                         University of Illinois Press. Amazon.com
    ## 149                                                                                                                                                          University of Illinois Urbana-Champaign
    ## 150                                                                                                                                                 University of Information Science and Technology
    ## 151                                                                                                                                                                               University of Iowa
    ## 152                                                                                                                                                                       University of Juiz de Fora
    ## 153                                                                                                                                                                             University of Kansas
    ## 154                                                                                                                                                                   University of Kansas Alzheimer
    ## 155                                                                                                                                                              University of Kansas Medical Center
    ## 156                                                                                                                                                          University of Kansas. Academic Medicine
    ## 157                                                                                                                                                                           University of Kentucky
    ## 158                                                                                                                                                                University of Kentucky UKnowledge
    ## 159                                                                                                                                                                             University of Korea 
    ## 160                                                                                                                                                                      University of KwaZulu-Natal
    ## 161                                                                                                                                                                           University of Lausanne
    ## 162                                                                                                                                                                              University of Leeds
    ## 163                                                                                                                                                                             University of Leuven
    ## 164                                                                                                                                                                         University of Louisville
    ## 165                                                                                                                                                                              University of Maine
    ## 166                                                                                                                                                                             University of Malaya
    ## 167                               University of Malaya. Dr Garcia-Vicuna reported receiving grants and personal fees from Sanofi and Lilly. Dr Gonzalez-Alvaro reported receiving grants from Sanofi
    ## 168                                                                                                                                                                           University of Maryland
    ## 169                                                                                                                                                       University of Maryland - Baltimore Country
    ## 170                                                                                                                                         University of Maryland COVID-19 Impact Analysis Platform
    ## 171                                                                                                                                  University of Maryland COVID-19 Impact Analysis Platform. https
    ## 172                                                                                                                                                        University of Maryland School of Medicine
    ## 173                                                                                                                                                                          University of Maryland.
    ## 174                                                                                                                                                               University of Massachusetts Boston
    ## 175                                                                                                                                                  University of Massachusetts Chan Medical School
    ## 176                                                                                                                                                 University of Massachusetts Medical School. 2007
    ## 177                                                                                                                                                                   University of Medical Sciences
    ## 178                                                                                                                                                                           University of Medicine
    ## 179                                                                                                                                                       University of Medicine and Health Sciences
    ## 180                                                                                                                                                                          University of Melbourne
    ## 181                                                                                                                                                                              University of Miami
    ## 182                                                                                                                                                                University of Miami Health System
    ## 183                                                                                                                                                                           University of Michigan
    ## 184                                                                                                                                                       University of Michigan Rogel Cancer Center
    ## 185                                                                                                                                                                          University of Michigan.
    ## 186                                                                                                                                                                      University of Minas Gerais 
    ## 187                                                                                                                                                                          University of Minnesota
    ## 188                                                                                                                      University of Minnesota Center for Infectious Disease Research and Policy. 
    ## 189                                                                                                                                                                    University of Minnesota Press
    ## 190                                                                                                                                             University of Minnesota Rural Health Research Center
    ## 191                                                                                                                                                                           University of Missouri
    ## 192                                                                                                                                                                            University of Montana
    ## 193                                                                                                                                                                        University of Mount Union
    ## 194                                                                                                                                                                             University of Murcia
    ## 195                                                                                                                                                            University of Nebraska Medical Center
    ## 196                                                                                                                                                           University of Nebraska Medical Center 
    ## 197                                                                                                                                                                             University of Nevada
    ## 198                                                                                                                                                                        University of New England
    ## 199                                                                                                                                                                      University of New Hampshire
    ## 200                                                                                                                                                                        University of New Mexico.
    ## 201                                                                                                                                                                    University of New South Wales
    ## 202                                                                                                                                                             University of New South Wales Sydney
    ## 203                                                                                                                                                                           University of New York
    ## 204                                                                                                                                                                          University of New York 
    ## 205                                                                                                                             University of New York College of Environmental Science and Forestry
    ## 206                                                                                                                                                    University of New York New York New York USA.
    ## 207                                                                                                                                           University of New York-University Hospital of Brooklyn
    ## 208                                                                                                                                                                     University of North Carolina
    ## 209                                                                                                                                                       University of North Carolina at Greensboro
    ## 210                                                                                                                                                         University of North Carolina Chapel Hill
    ## 211                                                                                                                                                  University of North Carolina School of Medicine
    ## 212                                                                                                                                                                       University of North Dakota
    ## 213                                                                                                                                                                        University of North Texas
    ## 214                                                                                                                                                          University of Ohio Global Field Program
    ## 215                                                                                                                                                    University of Ontario Institute of Technology
    ## 216                                                                                                                                                                               University of Oslo
    ## 217                                                                                                                                                                             University of Oxford
    ## 218                                                                  University of Oxford. Digital contact tracing can slow or even stop coronavirus transmission and ease us out of lockdown. https
    ## 219                                                                                                                                                                            University of Palermo
    ## 220                                                                                                                                                                              University of Paris
    ## 221                                                                                                                                                                       University of Pennsylvania
    ## 222                                                                                                                                                         University of Pennsylvania Health System
    ## 223                                                                                                                                           University of Pennsylvania Perelman School of Medicine
    ## 224                                                                                                                                                                 University of Pennsylvania Press
    ## 225                                                                                                                                                    University of Pennsylvania School of Medicine
    ## 226                                                                                                                                                                      University of Pennsylvania.
    ## 227                                                                                                                                                                         University of Pittsburgh
    ## 228                                                                                                                                                          University of Pittsburgh Medical Center
    ## 229                                                                                                                                                         University of Pittsburgh Medical Center 
    ## 230                                                                                                                                                          University of Pittsburgh Medical Centre
    ## 231                                                                                                                                                      University of Pittsburgh School of Medicine
    ## 232                                                                                  University of Pittsburgh. Y.M.A. reports that he is the principal investigator on a clinical trial of lopinavir
    ## 233                                                                                                                                                                              University of Porto
    ## 234                                                                                                                                                            University of Puerto Rico at Mayagüez
    ## 235                                                                                                                                            University of Puerto Rico Comprehensive Cancer Center
    ## 236                                                                                                                                                                         University of Queensland
    ## 237                                                                                                                                                    University of Queensland Mayne Medical School
    ## 238                                                                                                                                                                       University of Rhode Island
    ## 239                                                                                                                                                                           University of Richmond
    ## 240                                                                                                                                                                  University of Rio Grande do Sul
    ## 241                                                                                                                                                           University of Rochester Medical Center
    ## 242                                                                                                                                                                             University of Rwanda
    ## 243                                                                                                                                                                          University of Sao Paulo
    ## 244                                                                                                                                                                         University of São Paulo 
    ## 245                                                                                                                                                             University of Science and Technology
    ## 246                                                                                                                                                         University of Sergipe Aracaju SE Brazil.
    ## 247                                                                                                                                                                          University of Singapore
    ## 248                                                                                                                                                                     University of South Carolina
    ## 249                                                                                                                                                                      University of South Florida
    ## 250                                                                                                                                                                University of Southern California
    ## 251                                                                                                                                    University of Southern California Los Angeles California USA.
    ## 252                                                                                                                                                               University of Southern California.
    ## 253                                                                                                                                                                   University of Southern Denmark
    ## 254                                                                                                                                                                             University of Sydney
    ## 255                                                                                                                                                                            University of Syndney
    ## 256                                                                                                                                                                         University of Technology
    ## 257                                                                                                                                                  University of Technology Owerri Owerri Nigeria.
    ## 258                                                                                                                                                               University of Texas at San Antonio
    ## 259                                                                                                                                                       University of Texas Health Sciences Center
    ## 260                                                                                                                                           University of Texas Health Sciences Center at Houston 
    ## 261                                                                                                                                                    University of Texas MD Anderson Cancer Center
    ## 262                                                                                                                                                                 University of Texas Southwestern
    ## 263                                                                                                                                                  University of Texas Southwestern Medical Center
    ## 264                                                                                                                                              University of Texas-Houston School of Public Health
    ## 265                                                                                                                                                                University of the Health Sciences
    ## 266                                                                                                                 University of the Health Sciences Infectious Diseases Clinical Research Program 
    ## 267                                                                                                                                                                    University of the Philippines
    ## 268                                                                                                                                      University of the Philippines Los Baños. J. Hum. Ecol. 2020
    ## 269                                                                                                                                                                      University of Thessalonica 
    ## 270                                                                                                                                                                            University of Toronto
    ## 271                                                                                                                                                                             University of Toulon
    ## 272                                                                                                                                                                           University of Tübingen
    ## 273                                                                                                                                                                               University of Utah
    ## 274                                                                                                                                                            University of Utah School of Medicine
    ## 275                                                                                                                                                                                University of Uyo
    ## 276                                                                                                                                                                         University of Washington
    ## 277                                                                                                                                                      University of Washington School of Medicine
    ## 278                                                                                                                                                 University of Washington Seattle Washington USA.
    ## 279                                                                                                                                                           University of Washington. 2018.   http
    ## 280                                                                                                                                                                       University of West Georgia
    ## 281                                                                                                                                                                  University of Western Australia
    ## 282                                                                                                                                                                          University of Wisconsin
    ## 283                                                                                                                                                                  University of Wisconsin Madison
    ## 284                                                                                                                                                                    University of Wisconsin Press
    ## 285                                                                                                                                                                   University of Wisconsin System
    ## 286                                                                                                                             University of Wisconsin-Madison School of Medicine and Public Health
    ## 287                                                                                                                                                                University of Wisconsin-Milwaukee
    ## 288                                                                                                                                                               University of Wisconsin-Whitewater
    ## 289                                                                                                                                                                            University of Wyoming
    ## 290                                                                                                                                                                               University of York
    ##     Freq
    ## 1      1
    ## 2     15
    ## 3      2
    ## 4      4
    ## 5      1
    ## 6      2
    ## 7      2
    ## 8      1
    ## 9      8
    ## 10     1
    ## 11     3
    ## 12     1
    ## 13     1
    ## 14     1
    ## 15     2
    ## 16     5
    ## 17     2
    ## 18     1
    ## 19     1
    ## 20     1
    ## 21     2
    ## 22     1
    ## 23     1
    ## 24     1
    ## 25     2
    ## 26     1
    ## 27     1
    ## 28     1
    ## 29     3
    ## 30     1
    ## 31     1
    ## 32     1
    ## 33     1
    ## 34     1
    ## 35     1
    ## 36     2
    ## 37     4
    ## 38     2
    ## 39     1
    ## 40     1
    ## 41     1
    ## 42     3
    ## 43     2
    ## 44     2
    ## 45     1
    ## 46     1
    ## 47     5
    ## 48     2
    ## 49     2
    ## 50     4
    ## 51    20
    ## 52     8
    ## 53     1
    ## 54     1
    ## 55     1
    ## 56     4
    ## 57     1
    ## 58     2
    ## 59     2
    ## 60    23
    ## 61    69
    ## 62     1
    ## 63     1
    ## 64     1
    ## 65     1
    ## 66     1
    ## 67     1
    ## 68     4
    ## 69     1
    ## 70     4
    ## 71     4
    ## 72     6
    ## 73     2
    ## 74     1
    ## 75     1
    ## 76     3
    ## 77    11
    ## 78     1
    ## 79     1
    ## 80     2
    ## 81     1
    ## 82     1
    ## 83    12
    ## 84     4
    ## 85     1
    ## 86     1
    ## 87     3
    ## 88     3
    ## 89     8
    ## 90     1
    ## 91     1
    ## 92     1
    ## 93     1
    ## 94     1
    ## 95     9
    ## 96     2
    ## 97     1
    ## 98     2
    ## 99     1
    ## 100    1
    ## 101  359
    ## 102    8
    ## 103    1
    ## 104  106
    ## 105    3
    ## 106    1
    ## 107    1
    ## 108    1
    ## 109    5
    ## 110  210
    ## 111    1
    ## 112    9
    ## 113    1
    ## 114    2
    ## 115    1
    ## 116    1
    ## 117    1
    ## 118    1
    ## 119    2
    ## 120    1
    ## 121   26
    ## 122    1
    ## 123    1
    ## 124    3
    ## 125    2
    ## 126    2
    ## 127   11
    ## 128    1
    ## 129    1
    ## 130    1
    ## 131    3
    ## 132    1
    ## 133    3
    ## 134    1
    ## 135    2
    ## 136    2
    ## 137    1
    ## 138    2
    ## 139    1
    ## 140    6
    ## 141    3
    ## 142    4
    ## 143    3
    ## 144    3
    ## 145    1
    ## 146    1
    ## 147    1
    ## 148    1
    ## 149    4
    ## 150    2
    ## 151    4
    ## 152    4
    ## 153    1
    ## 154    1
    ## 155    1
    ## 156    1
    ## 157    1
    ## 158    1
    ## 159    1
    ## 160    3
    ## 161    1
    ## 162    2
    ## 163    1
    ## 164    1
    ## 165    2
    ## 166    2
    ## 167    1
    ## 168    9
    ## 169    6
    ## 170    1
    ## 171    1
    ## 172    5
    ## 173    1
    ## 174    1
    ## 175   21
    ## 176    2
    ## 177    3
    ## 178    4
    ## 179    2
    ## 180    2
    ## 181    2
    ## 182    1
    ## 183    7
    ## 184    2
    ## 185    1
    ## 186    1
    ## 187    3
    ## 188    1
    ## 189    1
    ## 190    1
    ## 191    2
    ## 192    3
    ## 193    1
    ## 194    1
    ## 195    4
    ## 196    1
    ## 197    3
    ## 198    1
    ## 199    1
    ## 200    6
    ## 201    3
    ## 202    1
    ## 203    1
    ## 204    2
    ## 205    3
    ## 206    1
    ## 207    1
    ## 208    2
    ## 209    2
    ## 210    1
    ## 211    1
    ## 212    1
    ## 213    2
    ## 214    1
    ## 215    1
    ## 216    6
    ## 217   10
    ## 218    1
    ## 219    1
    ## 220    1
    ## 221   91
    ## 222    6
    ## 223   13
    ## 224    1
    ## 225    7
    ## 226    1
    ## 227    7
    ## 228    1
    ## 229    1
    ## 230    2
    ## 231    3
    ## 232    1
    ## 233    3
    ## 234    1
    ## 235    2
    ## 236    2
    ## 237    1
    ## 238    3
    ## 239    1
    ## 240    1
    ## 241    4
    ## 242    1
    ## 243    2
    ## 244    4
    ## 245   34
    ## 246    1
    ## 247    1
    ## 248    3
    ## 249    1
    ## 250   23
    ## 251    2
    ## 252    1
    ## 253    1
    ## 254    2
    ## 255    1
    ## 256    2
    ## 257    3
    ## 258    1
    ## 259    1
    ## 260    2
    ## 261    4
    ## 262    1
    ## 263    2
    ## 264    1
    ## 265  241
    ## 266    1
    ## 267    1
    ## 268    1
    ## 269    1
    ## 270   15
    ## 271    1
    ## 272    7
    ## 273    5
    ## 274    3
    ## 275    1
    ## 276    7
    ## 277    3
    ## 278    1
    ## 279    1
    ## 280    1
    ## 281    1
    ## 282    3
    ## 283    2
    ## 284    1
    ## 285    1
    ## 286    1
    ## 287    1
    ## 288    4
    ## 289    1
    ## 290    1

Repeat the exercise and this time focus on schools and departments in
the form of

1.  School of …
2.  Department of …

And tabulate the results

``` r
schools_and_deps <- stringr::str_extract_all(
  publications_txt,
  "School of [\\w\\-\\s]+|Department of [\\w\\-\\s]+"
  )
as.data.frame(table(schools_and_deps))
```

    ##                                                                                                                                                                                     schools_and_deps
    ## 1                                                                                                                                                                    Department of Ageing and Health
    ## 2                                                                                                                                                  Department of Agricultural and Resource Economics
    ## 3                                                                                                                                                                          Department of Agriculture
    ## 4                                                                                                                                                    Department of Agriculture and Consumer Services
    ## 5                                                                                                                                                         Department of Agriculture Research Service
    ## 6                                                                                                                                                                              Department of Anatomy
    ## 7                                                                                                                                                        Department of Anesthesia and Intensive Care
    ## 8                                                                                                                                                         Department of Anesthesia and Pain Medicine
    ## 9                                                                                                                                        Department of Anesthesilogy Critical Care and Pain Medicine
    ## 10                                                                                                                                                                      Department of Anesthesiology
    ## 11                                                                                                                                                  Department of Anesthesiology and Pain Management
    ## 12                                                                                                                                                                        Department of Anthropology
    ## 13                                                         Department of Applied Business Studies in the Robbins College of Business and Entrepreneurship Fort Hays State University Hays Kansas USA
    ## 14                                                                                                                                                              Department of Applied Health Science
    ## 15                                                                                                                                                      Department of Atmospheric and Space Sciences
    ## 16                                                                                                                                                                Department of Atmospheric Sciences
    ## 17                                                                                                                                                                   Department of Behavioral Health
    ## 18                                                                                                                                                                        Department of Biochemistry
    ## 19                                                                                                                                                                          Department of Bioethics 
    ## 20                                                                                                                                                                 Department of Biological Sciences
    ## 21                                                                                                               Department of Biological Sciences and the Advanced Environmental Research Institute
    ## 22                                                                                                                                                                             Department of Biology
    ## 23                                                                                                                             Department of Biology Brigham Young University-Hawaii Laie Hawaii USA
    ## 24                                                                                                                                                              Department of Biomedical Engineering
    ## 25                                                                                                                                                              Department of Biomedical Informatics
    ## 26                                                                                                                                                                          Department of Biophysics
    ## 27                                                                                                                                                                         Department of Biosciences
    ## 28                                                                                                    Department of Biosciences Piracicaba Dental School University of Campinas Piracicaba SP Brazil
    ## 29                                                                                                                                                                       Department of Biostatistics
    ## 30                                                                                                                                                    Department of Biostatistics and Bioinformatics
    ## 31                                                                                                                                               Department of Biostatistics and Medical Informatics
    ## 32                                                                                                                                                          Department of Botany and Plant Pathology
    ## 33                                                                                                                                                                            Department of Business
    ## 34                                                                                                                                               Department of Business Economic Development Tourism
    ## 35                                                                                                                                                                          Department of Cardiology
    ## 36                                                                                                                                                              Department of Cardiovascular Surgery
    ## 37                                                                                                                                                          Department of Cell and Molecular Biology
    ## 38                                                                                                                                                                           Department of Chemistry
    ## 39                                                                                                                                                            Department of Chemistry and Bioscience
    ## 40                                                                                                                                                 Department of Civil and Environmental Engineering
    ## 41                                                                                                             Department of Civil and Environmental Engineering and Water Resources Research Center
    ## 42                                                                                                                                                                  Department of Clinical Education
    ## 43                                                                                                                                                              Department of Clinical Investigation
    ## 44                                                                                                                                                               Department of Clinical Pharmacology
    ## 45                                                                                                                                                                   Department of Clinical Research
    ## 46                                                                                                                                                                            Department of Commerce
    ## 47                                                                                                                                   Department of Commerce  2016 Household income and expenditures 
    ## 48                                                                                                                                                                       Department of Communication
    ## 49                                                                                                                                                Department of Communication Sciences and Disorders
    ## 50                                                                                                                                                                       Department of Communicology
    ## 51                                                                                                                                                       Department of Community and Health Services
    ## 52                                                                                                                                                                    Department of Community Health
    ## 53                                                                                                                                                                    Department of Community Safety
    ## 54                                                                                                                                             Department of Computational and Quantitative Medicine
    ## 55                                                                                                                                                                    Department of Computer Science
    ## 56                                                                                                                                                                       Department of Critical Care
    ## 57                                                                                                                                                              Department of Critical Care Medicine
    ## 58                                                                                                                                                             Department of Critical Care Medicine 
    ## 59                                                                                                                                                                 Department of Cyberinfrastructure
    ## 60                                                                                                                                                                             Department of Defense
    ## 61                                                                                                                                                                            Department of Defense 
    ## 62                                                                                          Department of Defense for clinical trials of convalescent plasma for COVID-19 outside the submitted work
    ## 63                                                                                                                                                                 Department of Defense institution
    ## 64                                                                                                                                 Department of Defense Joint Program Executive Office for Chemical
    ## 65                                                                                                                                                Department of Defense Military Treatment Facility 
    ## 66                                                                                                                                             Department of Defense on Domestic Travel Restrictions
    ## 67                                                                                                                                                       Department of Defense Suicide Event Report 
    ## 68                                                                                                                                                                     Department of Defense website
    ## 69                                                                                                                           Department of Dentistry Federal University of Sergipe Aracaju SE Brazil
    ## 70                                                                                                                                                                         Department of Dermatology
    ## 71                                                                                                                                                                             Department of Ecology
    ## 72                                                                                                                                                    Department of Ecology and Evolutionary Biology
    ## 73                                                                                                                                                         Department of Economic and Social Affairs
    ## 74                                                                                                                                                                           Department of Economics
    ## 75                                                                                                                                                                 Department of Economics and UHERO
    ## 76                                                                                                                                                                           Department of Education
    ## 77                                                                                                                                                                          Department of Education 
    ## 78                                                                                                                                                                 Department of Education data book
    ## 79                                                                                                                                                                   Department of Education website
    ## 80                                                                                                                                                                  Department of Emergency Medicine
    ## 81                                                                                                                      Department of English Studies Universitat Jaume I Castello de la Plana Spain
    ## 82                                                                                                                                                     Department of Environmental and Global Health
    ## 83                                                                                                                                                                Department of Environmental Health
    ## 84                                                                                                                                                       Department of Environmental Health Sciences
    ## 85                                                                                                                                                              Department of Environmental Medicine
    ## 86                                                                                                                                                               Department of Environmental Science
    ## 87                                                                                                Department of Environmental Science and Management Humboldt State University Arcata California USA
    ## 88                                                                                                                                                  Department of Environmental Studies and Sciences
    ## 89                                                                                                                                                                        Department of Epidemiology
    ## 90                                                                                                                                                      Department of Epidemiology and Biostatistics
    ## 91                                                                                                                       Department of Epidemiology and Biostatistics at the School of Public Health
    ## 92                                                                                                                                                  Department of Epidemiology and Population Health
    ## 93                                                                                                                                                           Department of Experimental Therapeutics
    ## 94                                                                                                                                                                             Department of Family 
    ## 95                                                                                                                                                                     Department of Family Medicine
    ## 96                                                                                                                                                Department of Family Medicine and Community Health
    ## 97                                                                                                                                              Department of Family Medicine and Community Medicine
    ## 98                                                                                                                                                   Department of Family Medicine and Public Health
    ## 99                                                                                                                                          Department of Family Medicine and Public Health Medicine
    ## 100                   Department of Family Medicine clinic called patients instructed by our physicians to quarantine for exposure risk or symptoms of potential COVID-19 infection between March 15
    ## 101                                                                                                                                                                      Department of Fish and Game
    ## 102                                                                                                                                                                          Department of Fisheries
    ## 103                                                                                                                                            Department of Forestry and Environmental Conservation
    ## 104                                                                                                                                               Department of Forestry and Environmental Resources
    ## 105                                                                                                                                                     Department of Forestry and Natural Resources
    ## 106                                                                                                                                                                   Department of General Medicine
    ## 107                                                                                                                                                 Department of Genetic and Molecular Epidemiology
    ## 108                                                                                                                                                                          Department of Geography
    ## 109                                                                                                                                                Department of Geography and Environmental Science
    ## 110                                                                                                                                       Department of Geography and Geographic Information Science
    ## 111                                                                                                                                        Department of Geosciences and Natural Resource Management
    ## 112                                                                                                                                                  Department of Geosciences and Natural Resources
    ## 113                                                                                                                                                                 Department of Geriatric Medicine
    ## 114                                                                                                                                                                             Department of Health
    ## 115                                                                                                                                                                           Department of Health\n
    ## 116                                                                                                                                                                            Department of Health 
    ## 117                                                                                                                                                                           Department of Health  
    ## 118                                                                                                                                           Department of Health  Alcohol and Drug Abuse Division 
    ## 119                                                                                                                   Department of Health  Child and Adolescent Mental Health Performance Standards
    ## 120                                                                                                                                                Department of Health  Current Situation in Hawaii
    ## 121                                                                                                                                                        Department of Health  Dikos Ntsaaígíí-19 
    ## 122                                                                                                                                                           Department of Health  Hawaii COVID-19 
    ## 123                                                                                                                                                      Department of Health  Hawaii COVID-19 Cases
    ## 124                                                                                                                                          Department of Health  Hawaii COVID-19 Cases and Testing
    ## 125                                                                                                                                          Department of Health  Hawaii Covid-19 Daily News Digest
    ## 126                                                                                                                                Department of Health  Secondary Hawaii State Department of Health
    ## 127                                                                                                                              Department of Health  What the Hawaii Department of Health is Doing
    ## 128                                                                                                                                                       Department of Health  What You Should Know
    ## 129                                                                                                                             Department of Health - Western Visayas Center for Heath Development 
    ## 130                                                                                                                                                              Department of Health Accessed March
    ## 131                                                                                                                                                            Department of Health Accessed March 4
    ## 132                                                                                                                                                                     Department of Health Affairs
    ## 133                                                                                                                                                         Department of Health and\nHuman Services
    ## 134                                                                                                                                                        Department of Health and Exercise Science
    ## 135                                                                                                                                                         Department of Health and Human Resources
    ## 136                                                                                                                                                          Department of Health and Human Services
    ## 137                                                                                                                                                         Department of Health and Human Services 
    ## 138                                                                                                  Department of Health and Human Services  Supply and demand projections of the nursing workforce
    ## 139                                                                       Department of Health and Human Services  The feasibility of using electronic health data for research on small populations
    ## 140                                                                     Department of Health and Human Services Implements Electronic Death Registration System to Streamline Death Reporting in NC 
    ## 141                                                                                                                                                 Department of Health and Human Services Part 84 
    ## 142                                                                                                                                                             Department of Health and Kinesiology
    ## 143                                                                                                                                                            Department of Health and Social Care 
    ## 144                                                                                                                                                    Department of Health and the New South Wales 
    ## 145 Department of Health Behavioral Health Administration led and contracted a coalition of agencies to plan and implement an isolation and quarantine facility placement service that included food
    ## 146                                                                                                                               Department of Health Chronic Disease Management and Control Branch
    ## 147                                                                                                                                                   Department of Health Coronavirus Disease 2019 
    ## 148                                                                                                                                                                      Department of Health D of H
    ## 149                                                                                                                                           Department of Health Disease Outbreak Control Division
    ## 150                                                                                                                                          Department of Health Disease Outbreak Control Division 
    ## 151                                                                                                                                             Department of Health Family Health Services Division
    ## 152                                                                                                                                                                       Department of Health Hawai
    ## 153                                                                                                                                                                       Department of Health https
    ## 154                                                                                                                                                          Department of Health Organization Chart
    ## 155                                                                                                                                          Department of Health partnered with University of Hawai
    ## 156                                                                                                                                                       Department of Health Policy and Management
    ## 157                                                                                                                          Department of Health receives 2016 Digital Government Achievement Award
    ## 158                                                                                                                                       Department of Health serves a rural island community of 73
    ## 159                                                                                                                                                                    Department of Health Services
    ## 160                                                                                                                                                Department of Health Services Research and Policy
    ## 161                                                                                                                                                             Department of Health Systems Science
    ## 162                                                                                                                                                             Department of Health to bring health
    ## 163                                                                                                 Department of Health to support modelling contributions to Australian COVID-19 responses in 2020
    ## 164                                                                                                                                                           Department of Health webpage for Hawai
    ## 165                                                                                                                                                                     Department of Health website
    ## 166                                                                                                                                                                     Department of Health Website
    ## 167                                                                                                                                                                         Department of Hematology
    ## 168                                                                                                                                                            Department of Hematology and Oncology
    ## 169                                                                                                                                                                       Department of Home Affairs
    ## 170                                                                                                           Department of Homeland Security  Threat and hazard identification and risk assessment 
    ## 171                                                                                                                                                                    Department of Human Nutrition
    ## 172                                                                                                                                                  Department of Immunology and Infectious Disease
    ## 173                                                                                                                                                                 Department of Infectious Control
    ## 174                                                                                                                                                                 Department of Infectious Disease
    ## 175                                                                                                                                                                Department of Infectious Diseases
    ## 176                                                                                                                                              Department of Infectious Diseases and Public Health
    ## 177                                                                                                                                         Department of Information Systems and Business Analytics
    ## 178                                                                                                                                           Department of Information Systems and Computer Science
    ## 179                                                                                                                                                                     Department of Intensive Care
    ## 180                                                                                                                                                            Department of Intensive Care Medicine
    ## 181                                                                                                                                                                   Department of Internal Affairs
    ## 182                                                                                                                                                                  Department of Internal Medicine
    ## 183                                                                                                                                                   Department of Internal Medicine and Pediatrics
    ## 184                                                                                                                                                               Department of International Health
    ## 185                                                                                                                                                   Department of Kinesiology and Exercise Science
    ## 186                                                                                                                                                                              Department of Labor
    ## 187                                                                                                                                                     Department of Labor and Industrial Relations
    ## 188                                                                                                                                                                Department of Laboratory Medicine
    ## 189                                                                                                                                                                        Department of Mathematics
    ## 190                                                                                                                 Department of Mathematics Federal University of Technology Owerri Owerri Nigeria
    ## 191                                                                                                                                 Department of Mathematics University of Benin Benin City Nigeria
    ## 192                                                                                                                         Department of Mathematics University of Hawaii Manoa Honolulu Hawaii USA
    ## 193                                                                                                                                                             Department of Mechanical Engineering
    ## 194                                                                                                                                                   Department of Medical Ethics and Health Policy
    ## 195                                                                                                                                                               Department of Medical Informatics 
    ## 196                                                                                                                                                                   Department of Medical Oncology
    ## 197                                                                                                                                                                   Department of Medical Research
    ## 198                                                                                                                                                                   Department of Medical Sciences
    ## 199                                                                                                                                                                           Department of Medicine
    ## 200                                                                                                                                                                          Department of Medicine 
    ## 201                                                                                                                                                       Department of Metabolism and Endocrinology
    ## 202                                                                                                                                                                       Department of Microbiology
    ## 203                                                                                                                                                        Department of Microbiology and Immunology
    ## 204                                                                                                                                                    Department of Molecular and Cellular Sciences
    ## 205                                                                                                                                           Department of Molecular Biosciences and Bioengineering
    ## 206                                                                                                                                                            Department of Molecular Biotechnology
    ## 207                                                                                                                                                             Department of Native Hawaiian Health
    ## 208                                                                                                                                                                  Department of Natural Resources
    ## 209                                                                                                                                     Department of Natural Resources and Environmental Management
    ## 210                                                                                                                                       Department of Natural Resources and Environmental Sciences
    ## 211                                                                                                                                                          Department of Natural Resources Science
    ## 212                                                                                                                                                        Department of Nephrology and Rheumatology
    ## 213                                                                                                                                                               Department of Neurological Surgery
    ## 214                                                                                                                                                                          Department of Neurology
    ## 215                                                                                                                                                                       Department of Neurosurgery
    ## 216                                                                                                                                                                            Department of Nursing
    ## 217                                                                                                                                                                          Department of Nutrition
    ## 218                                                                                                                                                         Department of Nutrition and Food Studies
    ## 219                                                                                                                                                                                 Department of OB
    ## 220                                                                                                                                                                         Department of Obstetrics
    ## 221                                                                                                                                                                        Department of Obstetrics 
    ## 222                                                                                                                                                               Department of Occupational Therapy
    ## 223                                                                                                                                                                      Department of Ophthalmology
    ## 224                                                                                                                                                                Department of Orthopaedic Surgery
    ## 225                                                                                                                                                                 Department of Orthopedic Surgery
    ## 226                                                                                                                                               Department of Otolaryngology-Head and Neck Surgery
    ## 227                                                                                                                                                            Department of Paediatric Rheumatology
    ## 228                                                                                                                                                             Department of Parliamentary Services
    ## 229                                                                                                                                                                          Department of Pathology
    ## 230                                                                                                                                                  Department of Pathology and Laboratory Medicine
    ## 231                                                                                                                                                              Department of Pediatric Dermatology
    ## 232                                                                                                                                                                Department of Pediatric Neurology
    ## 233                                                                                                                                                                         Department of Pediatrics
    ## 234                                                                                                                                                        Department of Pediatrics and Child Health
    ## 235                                                                                                                                                         Department of Pharmaceutical Biomedicine
    ## 236                                                                                                                                                                       Department of Pharmacology
    ## 237                                                                                                                                                                           Department of Pharmacy
    ## 238                                                                                                                                                        Department of Pharmacy and Biotechnology 
    ## 239                                                                                                                                                        Department of Physical Activity and Sport
    ## 240                                                                                                                                                                 Department of Physical Medicine 
    ## 241                                                                                                                                               Department of Physical Medicine and Rehabilitation
    ## 242                                                                                                                                                                         Department of Physiology
    ## 243                                                                                                                                                                        Department of Physiology 
    ## 244                                                                                                                                                          Department of Physiology and Biophysics
    ## 245                                                                                                                                                                      Department of Physiotherapy
    ## 246                                                                                                                                        Department of Plant and Environmental Protection Sciences
    ## 247                                                                                                                                                                        Department of Population 
    ## 248                                                                                                                                        Department of Population and Quantitative Health Sciences
    ## 249                                                                                                                                                                  Department of Population Health
    ## 250                                                                                                                                                         Department of Population Health Sciences
    ## 251                                                                                                                                                    Department of Prevention and Community Health
    ## 252                                                                                                                                                                Department of Preventive Medicine
    ## 253                                                                                                                                                               Department of Preventive Medicine 
    ## 254                                                                                                                                              Department of Preventive Medicine and Biostatistics
    ## 255                                                                                                                                                                         Department of Psychiatry
    ## 256                                                                                                                                                 Department of Psychiatry and Behavioral Sciences
    ## 257                                                                                                                                              Department of Psychiatry and Department of Medicine
    ## 258                                                                                                                                                      Department of Psychiatry and Human Behavior
    ## 259                                                                                                                                     Department of Psychiatry Emory University School of Medicine
    ## 260                                                                                                                                                                         Department of Psychology
    ## 261                                                                                                                                 Department of Psychology Brigham Young University Provo Utah USA
    ## 262                                                                                                                                                                      Department of Public Health
    ## 263                                                                                                                                                       Department of Public Health - Health Alert
    ## 264                                                                                                                                              Department of Public Health and Infectious Diseases
    ## 265                                                                                                                                                     Department of Public Health and Primary Care
    ## 266                                                                                                                                                  Department of Public Health and Social Services
    ## 267                                                                                                                                                             Department of Public Health Sciences
    ## 268                                                                                                                                                        Department of Pulmonary and Critical Care
    ## 269                                                                                                                                                                 Department of Pulmonary Diseases
    ## 270                                                                                                                                                        Department of Quantitative Health Science
    ## 271                                                                                                                                                       Department of Quantitative Health Sciences
    ## 272                                                                                                                                                Department of Radiotherapy and Radiation Oncology
    ## 273                                                                                                                                                            Department of Rehabilitation Medicine
    ## 274                                                                                                                                                            Department of Rehabilitation Sciences
    ## 275                                                                                                                                                                           Department of Research
    ## 276                                                                                                                                                                          Department of Research 
    ## 277                                                                                                                                                Department of Research and Clinical Investigation
    ## 278                                                                                                                                                            Department of Research and Evaluation
    ## 279                                                                                                                                                                       Department of Rheumatology
    ## 280                                                                                                                                               Department of Rheumatology and Clinical Immunology
    ## 281                                                                                                                                                        Department of Rheumatology and Immunology
    ## 282                           Department of Science and Technology of Guangdong Province and Health Commission of Guangdong Province for chloroquine in the treatment of novel coronavirus pneumonia
    ## 283                                                                                                                                                                   Department of Smoking and COPD
    ## 284                                                                                                  Department of Social and Behavioral Sciences and Lee Kum Sheung Center for Health and Happiness
    ## 285                                                                                                                                                                    Department of Social Medicine
    ## 286                                                                                                                                                                        Department of Social Work
    ## 287                                                                                                      Department of Social Work and Hā Kūpuna National Resource Center for Native Hawaiian Elders
    ## 288                                                                                                                                                                          Department of Sociology
    ## 289                                                                                                                                           Department of Sports Science and Clinical Biomechanics
    ## 290                                                                                                                                                                              Department of State
    ## 291                                                                                                                                                                           Department of State  U
    ## 292                                                                                                                                                                         Department of Statistics
    ## 293                                                                                                                                                                Department of Statistics Malaysia
    ## 294                                                                                                                                                                            Department of Surgery
    ## 295                                                                                                                                                                           Department of Surgery 
    ## 296                                                                                                                                                                 Department of Surgery and Cancer
    ## 297                                                                                                                                                                           Department of Taxation
    ## 298                                                                                                                                                                       Department of the Interior
    ## 299                                                                                                                                                               Department of Traffic Engineering 
    ## 300                                                                                                                                                     Department of Translational Medical Sciences
    ## 301                                                                                                                                                                     Department of Transportation
    ## 302                                                                                                                                                                  Department of Tropical Medicine
    ## 303                                                                                                                        Department of Tropical Medicine and Medical Microbiology and Pharmacology
    ## 304                                                                                                                                                   Department of Tropical Plant and Soil Sciences
    ## 305                                                                                                                                                                     Department of Twin Research 
    ## 306                                                                                                                                             Department of Twin Research and Genetic Epidemiology
    ## 307                                                                                                                                                                            Department of Urology
    ## 308                                                                                                                                                                   Department of Veterans Affairs
    ## 309                                                                                                                                                     Department of Veterans Affairs  COVID Coach 
    ## 310                                                                                                                                      Department of Veterans Affairs during the COVID-19 pandemic
    ## 311                                                                                                                            Department of Veterans Affairs hospitals during the COVID-19 pandemic
    ## 312                                                                                                                                                 Department of Veterinary Integrative Biosciences
    ## 313                                                                                                                                                                Department of Veterinary Medicine
    ## 314                                                                                                                                                                  Department of Veterinary School
    ## 315                                                                                                                                      Department of Virus and Microbiological Special Diagnostics
    ## 316                                                                                                                                                                           Department of Wildlife
    ## 317                                                                                                                                                  Department of Wildlife Ecology and Conservation
    ## 318                                                                                                                                                             Department of Zoology and Physiology
    ## 319                                                                                                                                                                     School of Aerospace Medicine
    ## 320                                                                                                                                                            School of Agriculture and Environment
    ## 321                                                                                                                         School of Applied Economics and Management Cornell University Ithaca USA
    ## 322                                                                                                            School of Aquatic and Fishery Science University of Washington Seattle Washington USA
    ## 323                                                                                                                                                                             School of Biological
    ## 324                                                                                                                                                                    School of Biological Sciences
    ## 325                                                                                                                                                                School of Biomedical Engineering 
    ## 326                                                                                                                                                                       School of Brown University
    ## 327                                                                                                                                                                               School of Business
    ## 328                                                                                                                            School of Business Mainz University of Applied Sciences Mainz Germany
    ## 329                                                                                                          School of Business Rosenheim Technical University of Applied Sciences Rosenheim Germany
    ## 330                                                                                                                  School of Business University of Southern California Los Angeles California USA
    ## 331                                                                                                                                                                   School of Chemical Engineering
    ## 332                                                                                                                                                                      School of Clinical Medicine
    ## 333                                                                                                                                                          School of Communication and Information
    ## 334                                                                                                                                                       School of Economics and Political Science 
    ## 335                                                                                                                                                        School of Education and Human Development
    ## 336                                                                                                                                                                      School of Education Science
    ## 337                                                                                                               School of Electronic Engineering Kumoh National Institute of Technology Gumi Korea
    ## 338                                                                                                                                                                 School of Energy and Environment
    ## 339                                                                                                                                                                            School of Engineering
    ## 340                                                                                                  School of Environmental and Biological Sciences Rutgers University New Brunswick New Jersey USA
    ## 341                                                                                                                                                   School of Epidemiology and Preventive Medicine
    ## 342                                                                                                                                                                                 School of Forest
    ## 343                                                                                                                                                         School of Forestry and Natural Resources
    ## 344                                                                                                                                                         School of Forestry and Wildlife Sciences
    ## 345                                                                                                                                                               School of Government Working Paper
    ## 346                                                                                                                                                                               School of Hawaiian
    ## 347                                                                                                                                             School of Hawaiian Knowledge and University of Hawai
    ## 348                                                                                                                                                                                 School of Health
    ## 349                                                                                                                                                                 School of Health Administration 
    ## 350                                                                                                                                                     School of Health and Rehabilitation Sciences
    ## 351                                                                                                                                                                               School of Hygiene 
    ## 352                                                                                                                                                          School of Hygiene and Tropical Medicine
    ## 353                                                                                                                                                      School of Immunology and Microbial Sciences
    ## 354                                                                                                                                                                                    School of Law
    ## 355                                                                                                                                                                          School of Life Sciences
    ## 356                                                                                                                                                                               School of Medicine
    ## 357                                                                                                                                                               School of Medicine  Guidebook 2023
    ## 358                                                                                                                                              School of Medicine  Office of International Affairs
    ## 359                                                                                                                                                           School of Medicine and Health Sciences
    ## 360                                                                                                                                                          School of Medicine and Medical Sciences
    ## 361                                                                                                                           School of Medicine and New York University Langone Orthopedic Hospital
    ## 362                                                                                                                                                             School of Medicine and Public Health
    ## 363                                                                                                                                        School of Medicine and University of Hawaii Cancer Center
    ## 364                                                                                                                                                                  School of Medicine at Dartmouth
    ## 365                                                                                                                                                                School of Medicine at Mount Sinai
    ## 366                                                                                                                                                                       School of Medicine at UCLA
    ## 367                                                                                                                                                       School of Medicine Biocontainment Facility
    ## 368                                                                                                                                                School of Medicine Dept of Native Hawaiian Health
    ## 369                                                                                                                            School of Medicine have helped elucidate the connections between diet
    ## 370                                                                                                                                                                         School of Medicine in St
    ## 371                                                                                                                                                                   School of Medicine in St Louis
    ## 372                                                                                                                                                                     School of Medicine in the US
    ## 373                                                                                                                                                   School of Medicine of the University of Hawaii
    ## 374                                                                                                                                          School of Medicine of University of Southern California
    ## 375                                                                                                                                                                        School of Medicine of USC
    ## 376                                                                                                                                                School of Medicine Univ of Hawaii Honolulu HI USA
    ## 377                                                                                                                                                      School of Medicine University of California
    ## 378                                                                                                                                                      School of Natural Resources and Environment
    ## 379                                                                                                                                                       School of Natural Sciences and Mathematics
    ## 380                                                                                                                                                                                School of Nursing
    ## 381                                                                                                                                                                               School of Nursing 
    ## 382                                                                                                                                                             School of Nursing and Dental Hygiene
    ## 383                                                                                                                                      School of Nursing and Dental Hygiene at University of Hawai
    ## 384                                                                                                                  School of Nursing at The University of Texas Health Sciences Center at Houston 
    ## 385                                                                                                                                                 School of Ocean and Earth Science and Technology
    ## 386                                                                                                                                                              School of Pacific and Asian Studies
    ## 387                                                                                                                                                                               School of Pharmacy
    ## 388                                                                                                                                                          School of Pharmacy and Medical Sciences
    ## 389                                                                                                                                                                     School of Physical Education
    ## 390                                                                                                                                                           School of Physical Education and Sport
    ## 391                                                                                                                      School of Physical Therapy and Graduate Institute of Rehabilitation Science
    ## 392                                                                                                                                                                          School of Physiotherapy
    ## 393                                                                                                                                               School of Population Health and Community Medicine
    ## 394                                                                                                                                                       School of Public and Environmental Affairs
    ## 395                                                                                                        School of Public and Environmental Affairs and associate vice provost for health sciences
    ## 396                                                                                                                                                                          School of Public Health
    ## 397                                                                                                                                                  School of Public Health and Preventive Medicine
    ## 398                                                                                                                                                                          School of Public Policy
    ## 399                                                                                                                                                                         School of Public Service
    ## 400                                                                                                                                                                               School of Sciences
    ## 401                                                                                                                                                                        School of Social Sciences
    ## 402                                                                                                                                                                            School of Social Work
    ## 403                                                                                                                                                                           School of Social Work 
    ## 404                                                                                                                                                          School of Social Work and Public Health
    ## 405                                                                                                                                                         School of the University of Pennsylvania
    ## 406                                                                                                                                                                         School of Transportation
    ## 407                                                                                                                                                School of Veterinary Medicine at Tufts University
    ##     Freq
    ## 1      1
    ## 2      1
    ## 3      6
    ## 4      1
    ## 5      1
    ## 6     30
    ## 7      2
    ## 8      1
    ## 9      1
    ## 10    14
    ## 11     1
    ## 12     3
    ## 13     1
    ## 14     3
    ## 15     2
    ## 16     1
    ## 17     3
    ## 18     2
    ## 19     1
    ## 20    10
    ## 21     2
    ## 22    30
    ## 23     2
    ## 24     1
    ## 25     3
    ## 26     1
    ## 27     1
    ## 28     3
    ## 29    16
    ## 30     5
    ## 31     1
    ## 32     1
    ## 33    33
    ## 34     1
    ## 35     1
    ## 36     1
    ## 37     4
    ## 38     2
    ## 39     1
    ## 40    12
    ## 41     1
    ## 42     1
    ## 43     3
    ## 44     1
    ## 45     6
    ## 46     4
    ## 47     1
    ## 48     2
    ## 49     1
    ## 50     2
    ## 51     2
    ## 52     2
    ## 53     1
    ## 54     1
    ## 55     1
    ## 56     2
    ## 57     2
    ## 58    11
    ## 59     1
    ## 60     8
    ## 61     3
    ## 62     1
    ## 63     2
    ## 64     2
    ## 65     1
    ## 66     1
    ## 67     1
    ## 68     1
    ## 69     1
    ## 70    27
    ## 71     2
    ## 72     2
    ## 73     5
    ## 74     8
    ## 75    10
    ## 76     5
    ## 77     2
    ## 78     1
    ## 79     1
    ## 80    18
    ## 81     2
    ## 82     2
    ## 83     1
    ## 84     1
    ## 85     1
    ## 86     1
    ## 87     2
    ## 88     2
    ## 89    15
    ## 90     6
    ## 91     1
    ## 92     4
    ## 93     1
    ## 94     1
    ## 95     2
    ## 96    10
    ## 97     1
    ## 98     1
    ## 99     1
    ## 100    1
    ## 101    1
    ## 102    1
    ## 103    2
    ## 104    2
    ## 105    2
    ## 106    7
    ## 107    1
    ## 108    4
    ## 109    2
    ## 110    4
    ## 111    3
    ## 112    1
    ## 113    1
    ## 114  133
    ## 115    2
    ## 116   10
    ## 117    2
    ## 118    1
    ## 119    1
    ## 120    2
    ## 121    1
    ## 122    1
    ## 123    1
    ## 124    1
    ## 125    1
    ## 126    1
    ## 127    1
    ## 128    1
    ## 129    1
    ## 130    1
    ## 131    1
    ## 132    1
    ## 133    1
    ## 134    1
    ## 135    1
    ## 136   29
    ## 137    3
    ## 138    1
    ## 139    1
    ## 140    1
    ## 141    1
    ## 142    3
    ## 143    1
    ## 144    1
    ## 145    1
    ## 146    1
    ## 147    2
    ## 148    1
    ## 149    2
    ## 150    7
    ## 151    1
    ## 152    3
    ## 153    1
    ## 154    1
    ## 155    1
    ## 156    1
    ## 157    1
    ## 158    1
    ## 159    2
    ## 160    1
    ## 161    5
    ## 162    1
    ## 163    1
    ## 164    1
    ## 165    3
    ## 166    1
    ## 167    2
    ## 168    1
    ## 169    1
    ## 170    1
    ## 171    1
    ## 172    1
    ## 173    3
    ## 174    5
    ## 175   20
    ## 176    1
    ## 177    1
    ## 178    1
    ## 179    2
    ## 180    1
    ## 181   22
    ## 182   61
    ## 183    2
    ## 184    3
    ## 185    2
    ## 186    1
    ## 187    2
    ## 188    3
    ## 189   14
    ## 190    3
    ## 191    1
    ## 192    1
    ## 193    8
    ## 194    9
    ## 195    1
    ## 196    5
    ## 197    1
    ## 198    1
    ## 199  202
    ## 200    4
    ## 201    1
    ## 202    3
    ## 203   12
    ## 204    1
    ## 205    8
    ## 206    1
    ## 207    5
    ## 208    5
    ## 209    1
    ## 210    4
    ## 211    3
    ## 212    5
    ## 213   12
    ## 214    4
    ## 215    1
    ## 216    1
    ## 217    5
    ## 218    2
    ## 219    8
    ## 220   18
    ## 221    2
    ## 222    2
    ## 223    1
    ## 224    4
    ## 225    5
    ## 226    4
    ## 227    1
    ## 228    1
    ## 229   10
    ## 230    8
    ## 231    2
    ## 232    2
    ## 233   44
    ## 234    1
    ## 235    1
    ## 236    2
    ## 237    1
    ## 238    1
    ## 239    1
    ## 240    1
    ## 241    3
    ## 242    4
    ## 243    3
    ## 244    9
    ## 245    1
    ## 246    1
    ## 247    2
    ## 248    5
    ## 249    6
    ## 250    4
    ## 251    1
    ## 252   15
    ## 253    5
    ## 254  159
    ## 255   26
    ## 256    6
    ## 257    1
    ## 258    2
    ## 259    1
    ## 260    8
    ## 261    2
    ## 262   10
    ## 263    2
    ## 264    1
    ## 265    2
    ## 266    1
    ## 267    2
    ## 268    1
    ## 269    1
    ## 270    1
    ## 271   33
    ## 272    1
    ## 273    1
    ## 274    6
    ## 275    1
    ## 276   21
    ## 277    1
    ## 278    8
    ## 279    3
    ## 280    2
    ## 281    2
    ## 282    1
    ## 283    8
    ## 284    1
    ## 285    3
    ## 286    3
    ## 287    1
    ## 288    9
    ## 289    1
    ## 290    2
    ## 291    1
    ## 292    5
    ## 293    1
    ## 294   17
    ## 295   22
    ## 296    2
    ## 297    2
    ## 298    1
    ## 299    1
    ## 300    1
    ## 301    1
    ## 302   62
    ## 303    4
    ## 304    1
    ## 305    2
    ## 306    2
    ## 307    1
    ## 308   14
    ## 309    2
    ## 310    1
    ## 311    1
    ## 312    2
    ## 313    4
    ## 314    1
    ## 315    1
    ## 316    2
    ## 317    3
    ## 318    1
    ## 319    7
    ## 320    1
    ## 321    1
    ## 322    1
    ## 323    1
    ## 324    3
    ## 325    3
    ## 326    2
    ## 327    1
    ## 328    1
    ## 329    1
    ## 330    2
    ## 331    1
    ## 332    1
    ## 333    2
    ## 334    1
    ## 335    1
    ## 336    2
    ## 337    1
    ## 338    2
    ## 339    1
    ## 340    2
    ## 341    6
    ## 342    1
    ## 343    2
    ## 344    2
    ## 345    1
    ## 346    1
    ## 347    1
    ## 348    2
    ## 349   11
    ## 350    1
    ## 351    1
    ## 352    1
    ## 353    1
    ## 354    1
    ## 355    3
    ## 356  639
    ## 357    1
    ## 358    1
    ## 359    1
    ## 360    2
    ## 361    2
    ## 362    1
    ## 363    1
    ## 364    2
    ## 365    9
    ## 366    4
    ## 367    2
    ## 368    1
    ## 369    1
    ## 370    1
    ## 371    1
    ## 372    1
    ## 373    1
    ## 374    6
    ## 375    2
    ## 376    2
    ## 377    6
    ## 378    1
    ## 379    3
    ## 380   22
    ## 381   14
    ## 382   32
    ## 383    2
    ## 384    2
    ## 385    1
    ## 386    1
    ## 387    1
    ## 388    1
    ## 389    1
    ## 390    2
    ## 391    3
    ## 392    1
    ## 393    2
    ## 394    1
    ## 395    1
    ## 396   74
    ## 397   18
    ## 398    8
    ## 399    1
    ## 400    4
    ## 401    3
    ## 402   11
    ## 403   33
    ## 404   22
    ## 405    1
    ## 406    1
    ## 407    1

## Question 5: Form a database

We want to build a dataset which includes the title and the abstract of
the paper. The title of all records is enclosed by the HTML tag
`ArticleTitle`, and the abstract by `Abstract`.

Before applying the functions to extract text directly, it will help to
process the XML a bit. We will use the `xml2::xml_children()` function
to keep one element per id. This way, if a paper is missing the
abstract, or something else, we will be able to properly match PUBMED
IDS with their corresponding records.

``` r
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
```

Now, extract the abstract and article title for each one of the elements
of `pub_char_list`. You can either use `sapply()` as we just did, or
simply take advantage of vectorization of `stringr::str_extract`

``` r
abstracts <- stringr::str_extract(pub_char_list, "<Abstract>(\\n|.)+</Abstract>")
abstracts <- stringr::str_remove_all(abstracts, "<Abstract>|</Abstract>")
abstracts <- stringr::str_replace_all(abstracts, "\\s+", " ")
sum(is.na(abstracts))
```

    ## [1] 53

- How many of these don’t have an abstract?

*53 of the publications don’t have an abstract.*

Now, the title

``` r
titles <- stringr::str_extract(pub_char_list, "<ArticleTitle>(\\n|.)+</ArticleTitle>")
titles <- stringr::str_remove_all(titles, "<ArticleTitle>|</ArticleTitle>")
sum(is.na(titles))
```

    ## [1] 0

- How many of these don’t have a title ? *None of the publictions don’t
  have a title.*

Finally, put everything together into a single `data.frame` and use
`knitr::kable` to print the results

``` r
database <- data.frame(
  pub_char_list, abstracts, titles
)
knitr::kable(database)
```

Done! Knit the document, commit, and push.

## Final Pro Tip (optional)

You can still share the HTML document on github. You can include a link
in your `README.md` file as the following:

``` md
View [here](https://cdn.jsdelivr.net/gh/:user/:repo@:tag/:file) 
```

For example, if we wanted to add a direct link the HTML page of lecture
6, we could do something like the following:

``` md
View Week 6 Lecture [here]()
```

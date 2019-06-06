
## Project 5: Recipe Recommender with a Hint of Healthy Bias

### Problem Statement    
Can a recommender be built to recommend similar recipes to a recipe a consumer likes (or craves)?  Can the recommender be biased toward recommending healthy alternatives to a less healthy recipe that a consumer likes (or craves)?

### Repository Structure  
presentation_JuliaTaussig.pdf: This is the presentation that was presented to the class on 17 May 2019.
#### Code:  
- 1_GatheringData.ipynb: Python Jupyter Notebook used to gather data using a function to query the Edamam Recipe Search API
- 2_Cleaning_ExploratoryDataAnalysis.ipynb: Python Jupyter Notebook used to combine data files, to perform some initial exploratory data analysis (EDA), and to clean the dataset prior to further EDA and recommender model building. A nutrition score was created and added to the clean recipes dataframe in this notebook as well.
- 3_ExploratoryDataAnalysis: Python Jupyter Notebook used to perform EDA to get an understanding about the contents of the data and the distributions of nutrients in the dataset as well as distributions of nutrients based on categories such as cuisine type or diets such as the ketogenic diet 
- 4_RecommenderModelBuilding.ipynb: Python Jupyter Notebook used to build the recommender using cosine similarity 
- 5_RecommenderModelEnhancementAndEvaluation.ipynb: Python Jupyter Notebook used to add nutritional bias to the recommender and to evaluate whether the recipe recommender model performs well

#### Contents of the Data Folder:  (NOTE: Data was not included in the repository to respect Edamam and their Terms and Conditions)
What would have been in the data folder:
- 150 csv files (starting with "recipes_dishType") with recipe data that was collected using the code in 1_GatheringData.ipynb
- several csv's (starting with "df_") that were saved during data cleaning and processing
- df_recs_clean_final.csv: The final recipes dataframe which was used to create the recommender
- df_recs_clean_final_nutr_sc.csv: The final recipes dataframe with a nutrition score column added (which was used to add bias toward more nutritious recipes to the recommender)
- recommender_final.csv: The cosine similarity matrix which became the recommender

### Data Collection:    
Recipe data was collected using the Edamam Recipe Search API, and the data dictionary can be found at this link: [https://developer.edamam.com/edamam-docs-recipe-api](https://developer.edamam.com/edamam-docs-recipe-api).  The terms and conditions for the Edamam API can be found here: [https://developer.edamam.com/about/terms](https://developer.edamam.com/about/terms).  Recipe data was collected for education purposes, and to respect Edamam, I did not include detailed data from my repository in my public repository.  I removed my API developer ID and key to ensure people do not use my credentials to attain data from Edamam. I included code people can use if they decide to utilize Edamam's API with their own developer ID and key. 

I decided early on to upgrade from a free Edamam developer account to an Enterprise developer account (which cost me 29 USD). This enabled me to acquire recipes based on dish type, I was able to get more detailed data (including more information such as dish type, meal type, and cuisine type for recipes), and I was no longer limited to 5 calls to the API per minute. I also was able to get up to 1000 recipes per query rather than only 100 recipes per query.  I searched for recipes of the following dish types: breakfast, lunch, dinner, dessert, and nibble.  I also queried the data by systematically running through nutrient content of 0 to a maximum amount for each nutrient that Edamam had information for (there were 28 nutrients included in Edamam's dataset).


### Data Compliling, Cleaning, and Initial EDA:  
I managed to collect over 300,000 recipes (305,542 recipes, to be exact), and I used the glob module to combine all 150 csv files that were created during queries.  I found out that most of the recipes were duplicates, so I removed duplicate recipes (thank goodness the url's were unique!), and I ended up with over 42,777 unique recipes.  I attempted to create a recommender using all 42,777 recipes, but I ran into memory issues (my recommender was over 10 GB in size when it was saved into a csv file).  I then decided to only look at recipes from the top 20 most frequent sources in the dataset because this would reduce the number of sources in the sources dummy matrix, and it reduced the number of recipes.  I ended up with 22,941 recipes which was still a large number of recipes. 

I created ingredient and ingredient category columns using embedded data from the dataframe.  I dropped columns that were full of null values (and they were empty columns because they were percent daily value columns for nutrients that ended up not having daily values set - according to my research, monounsaturated fat, polyunsaturated fat, trans fat, and total sugar (including sugar from natural sources, not just added sugar) do not have daily values).  I also imputed values of 0 for remaining null values because I assumed that if a nutritional value was not included in the query results, it was not present in the dish (for example, tea did not have many nutrients, so its nutrient values were mostly null and then became zero values).

I created amount of nutrient per serving columns.  I also created proportion of daily value per serving columns because these values would be between 0 and 1, and they would be easy to interpret and useful for the recommender.  I decided to drop a few columns which became redundant as well.  

I inspected categorical columns and created dummy matrices for each of the categorical columns.  I found that it was difficult to generate dummy matrices for columns which had two or more potential values rather than only one possible value per recipe.  I used for loops to create dummy columns carefully for the categorical features when it was required.  I ended up creating dummy matrices for the sources of recipes, diet labels of recipes, health labels of recipes, cautions of recipes, cuisine types of recipes, dish types of recipes, and ingredient categories of recipes.  In the future, I would like to find a way to use meal types, more detailed ingredients, the number of ingredients used, and recipe titles in the analysis.

The numerical columns and dummy matrices were cleaned (redundancies were removed by combining columns that were similar, and redundant columns were removed), merged, and a nutrition score was created and added to the recipe dataframe.

The nutrition score was created using a general rule of thumb from the FDA: try to minimize nutrients that you want to avoid such as cholesterol if you have a need to avoid it (have less than 20% of the daily value of these nutrients for each meal), and try to maximize nutrients that you want to get more of (have more than 20% of the daily value of these nutrients for each meal).  I counted the number of desired nutrients (such as iron and folate) to maximize that were above 20% of the DV per serving, the number of nutrients that were desired to minimize (such as cholesterol) that were below 20% of the DV per serving, and I counted the number of nutrients that were in healthy ranges (for example, I wanted to ensure enough protein was in a serving but not too much). Here is a link to the page I used when creating the nutrition score: [https://www.accessdata.fda.gov/scripts/InteractiveNutritionFactsLabel/pdv.html](https://www.accessdata.fda.gov/scripts/InteractiveNutritionFactsLabel/pdv.html). 

### Exploratory Data Analysis (EDA):  
Numerical features and their distributions were analyzed.  Distributions of each nutrient were analyzed using histograms.  It looks like there are a lot of upper outliers that skewed distributions of nutrients to the right.  It was surprising to see a lot of recipes had greater than the daily value in one serving.  Some recipes also had very high yields (number of servings) and cook times.  It would be interesting to investigate the outliers in the future.

Nutrient score distributions of recipes from different sources, recipes categorized as following some diets (such as the ketogenic diet), recipes with sulfites and recipes with FODMAP foods, and recipes for breakfast, lunch, dinner, dessert, and nibble (snack) dish types were visualized using histograms.  It was interesting to compare recipe nutrition score distribution scores between sources and other categorical features. 

The distribution of nutrient scores for all recipes was pretty normal with a bit of a right skew (likely caused by upper outliers).  

### Creating the Recommender Model:  
The clean recipes dataframe was imported into an iPython Jupyter Notebook, and the unique recipe url's were set as the index.  The column with the titles of the recipes was removed after confirming that a lot of the url's included the titles of the recipes for quick reference. All of the columns of the recipes dataframe were then numerical (integers or floats).  The data was then scaled using sklearn StandardScaler.  The dataframe was then changed to a sparse matrix using scipy's sparse module and csr_matrix function to reduce the size of the dataframe before generating the cosine similarity matrix (a computationally expensive step).  The cosine similarity matrix was then created using sklearn's metrics.pairwise.cosine_similarity module.  The cosine similarity matrix was then placed in a dataframe called recommender.  The recommender was very large (approx. 4.2 GB - at least it was smaller than the initial recommender that included 42,777 recipes and was over 10 GB in size), so it was reduced in size by changing the float64 type values to float16 type values.  The recommender was then saved to a csv file.

### Results:  
A function was created to take in a url of a recipe (which is in the recommender cosine similarity dataframe) that a consumer likes and to print the nutrition score of the recipe the user likes, the top 10 most similar recipes to the user's recipe they input, and the 10 healthiest recipes out of the 100 recipes that are most similar to the input recipe.  Details about the top 10 most nutritious recommended recipes are displayed in the output of the function as well just in case the user wants to learn more about the features of the recipe (such as the nutrition scores, diet types they qualify for, etc.).

Unfortunately the only metric I have been able to use so far is cosine similarity values and personal interpretation of whether the recommender was successful and output appropriate recommended recipes.  There are a lot of recipes that are similar to each other, and a lot of the recommended recipes make sense.  The issue is that the recommended recipes become less similar to the input recipe when the output top 100 recommendations are sorted by (biased toward) higher nutrition scores.  I'd like to survey people to see if they find the recommender useful.  I think I can gain useful metrics by surveying people, and it would be great to create a recommender that includes user input so it can become a combination of a content-based recommender and a collaborative recommender. 

I assumed that the nutrient score I created was sufficient.  I would like to find ways to optimize the nutrition score to ensure that the recommender is indeed recommending healthy recipes when it outputs the higher nutrient score biased recipe recommendations.

### Future Work:  
- I would like to clean the data more to see if I can improve the recommender.  There were a lot of cuisine types that probably could have been combined or removed.  For example, there was a cuisine type called Belgium and a cuisine type called Belgian.  Those cuisine types should have been combined.  I ran into time constraints and would like to clean up the cuisine type data in the future.
- I would also like to incorporate dish title (through some natural language processing (NLP)), ingredients (NLP likely required), number of ingredients, and meal type (NLP and cleaning likely required) features in the future.
- I would like to try a few different nutrient score algorithms.  I weighted every nutrient pretty similarly, but I wonder if it would have been beneficial to try a few different techniques to create a nutrient score. 
- It would be great to sort recommendation values by diet types people want to follow as well.  For example, if someone is ketogenic or pescatarian, it would be great to only show the person recommendations that align with his or her diet constraints.
- It would be helpful to get potential user input to gain information that could be used to add collaborative filters to the recommender so it can become a combination of a content-based recommender and a collaborative-based recommender.  I think that could increase capability of the recommender to recommend relevant recipes to users. 
- It would be great to conduct statistical hypothesis test to see whether the unbiased recommendations or the biased recommendations perform better.


### External libraries/packages used:  
- The glob module [https://docs.python.org/2/library/glob.html](https://docs.python.org/2/library/glob.html) was used to find csv files with similar names in a repository.  This was used to compile data from 150 csv files.
- An engine was used when reading data from a csv into a Pandas dataframe.  This helped to manage memory issues with large files (the following Stack Overflow post was helpful: [https://stackoverflow.com/questions/30494569/how-to-force-pandas-read-csv-to-use-float32-for-all-float-columns](https://stackoverflow.com/questions/30494569/how-to-force-pandas-read-csv-to-use-float32-for-all-float-columns)).

### Sources:  
-  https://developer.edamam.com/about/terms
- https://developer.edamam.com/edamam-docs-recipe-api
- https://www.accessdata.fda.gov/scripts/InteractiveNutritionFactsLabel/pdv.html
- https://danielmiessler.com/study/url-uri/
- https://blog.myfitnesspal.com/ask-the-dietitian-whats-the-best-carb-protein-and-fat-breakdown-for-weight-loss/
- http://www.oceanicresearch.org/education/wonders/mollusk.html
- https://www.health.harvard.edu/staying-healthy/how-much-calcium-do-you-really-need
- https://www.mayoclinic.org/healthy-lifestyle/nutrition-and-healthy-eating/in-depth/carbohydrates/art-20045705
- https://www.accessdata.fda.gov/scripts/interactivenutritionfactslabel/cholesterol.html
- https://medlineplus.gov/ency/patientinstructions/000785.htm
- https://medlineplus.gov/ency/patientinstructions/000785.htm
- https://medlineplus.gov/ency/patientinstructions/000785.htm
- https://medlineplus.gov/ency/patientinstructions/000785.htm
- https://medlineplus.gov/ency/patientinstructions/000785.htm
- Caitlin Dewey, https://www.washingtonpost.com/news/wonk/wp/2018/06/18/artificial-trans-fats-widely-linked-to-heart-disease-are-officially-banned/?utm_term=.1655f2600adf
- https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/
- https://www.mayoclinic.org/healthy-lifestyle/nutrition-and-healthy-eating/in-depth/high-fiber-foods/art-20050948
- https://www.webmd.com/diet/eat-this-fiber-chart
- https://www.medicalnewstoday.com/articles/321993.php
- https://www.mayoclinic.org/drugs-supplements-folate/art-20364625
- https://ods.od.nih.gov/factsheets/Potassium-HealthProfessional/
- https://www.mayoclinic.org/drugs-supplements/potassium-supplement-oral-route-parenteral-route/description/drg-20070753
- https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/
- https://www.fda.gov/food/nutrition-education-resources-and-materials/use-nutrition-facts-label-reduce-your-intake-sodium-your-diet
- https://www.cdc.gov/salt/pdfs/sodium_dietary_guidelines.pdf
- https://health.gov/dietaryguidelines/2015/guidelines/appendix-2/
- https://ods.od.nih.gov/factsheets/Niacin-HealthProfessional/ 
- https://www.mayoclinic.org/drugs-supplements-niacin/art-20364984
- https://medlineplus.gov/ency/article/002424.htm
- https://www.ncbi.nlm.nih.gov/books/NBK109813/
- https://www.accessdata.fda.gov/scripts/InteractiveNutritionFactsLabel/protein.html
- https://health.usnews.com/wellness/food/articles/2018-08-03/are-you-getting-too-much-protein-heres-why-it-matters
- https://ods.od.nih.gov/factsheets/Riboflavin-HealthProfessional/
- https://lpi.oregonstate.edu/mic/vitamins/riboflavin
- https://www.accessdata.fda.gov/scripts/InteractiveNutritionFactsLabel/sugars.html
- http://sugarscience.ucsf.edu/sugar-faq.html#.XNdZFutKhn4
- https://universityhealthnews.com/daily/nutrition/high-sugar-content-fruit-damaging-health-waistline/
- https://www.huffingtonpost.com.au/2017/09/14/how-much-natural-sugar-should-we-eat-a-day_a_23208377/
- https://www.medicalnewstoday.com/articles/324673.php
- https://ods.od.nih.gov/factsheets/Thiamin-HealthProfessional/
- https://www.mayoclinic.org/drugs-supplements-thiamin/art-20366430
- https://ods.od.nih.gov/factsheets/VitaminE-HealthProfessional/
- https://www.mayoclinic.org/drugs-supplements-vitamin-e/art-20364144
- https://ods.od.nih.gov/factsheets/VitaminA-HealthProfessional/
- https://www.mayoclinic.org/drugs-supplements-vitamin-a/art-20365945
- https://ods.od.nih.gov/factsheets/VitaminB12-HealthProfessional/
- https://www.mayoclinic.org/drugs-supplements-vitamin-b12/art-20363663
- https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/
- https://www.mayoclinic.org/drugs-supplements-vitamin-b6/art-20363468
- https://ods.od.nih.gov/factsheets/VitaminC-HealthProfessional/
- https://www.mayoclinic.org/healthy-lifestyle/nutrition-and-healthy-eating/expert-answers/vitamin-c/faq-20058030
- https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/
- https://www.mayoclinic.org/drugs-supplements-vitamin-d/art-20363792
- https://ods.od.nih.gov/factsheets/VitaminK-HealthProfessional/

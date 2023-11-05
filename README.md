# Dibimbing Data Engineering Batch 2 Final Project

![config](imgs/project_flow.png)

This Airflow data pipeline loads batch data source into cloud data storage, that is GCS and BigQuery, and then the raw dataset quality is checked by Soda Core. After that, the raw dataset is modeled into several dimension tables and fact tables by using dbt. Finally, the created tables are checked by Soda Core to ensure the dataset quality for visualization purposes in Metabase. This project used Astronomer CLI because it is a convenient tool set up Airflow in Docker environment.

## Table of Contents

- [Dataset](#dataset)
- [Stacks](#stack)
- [Prerequisites](#prerequisites)
- [How to Run](#how-to-run)
  - [Installation](#installation)
  - [Configuration](#configuration)
  - [Usage](#usage)
- [Code Explanation](#code-explanation)
- [Results Explanation](#results-explanation)
- [Conclusion and Future Updates](#conclusion-and-future-updates)

## Dataset
The datasets used in this project are `results.csv`, `shootouts.csv`, and `goalscorers.csv` from [kaggle](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017/data). The dataet is about 45,099 results of international football matches starting from the very first official match in 1872 up to 2023. The matches are strictly men's full internationals and the data does not include Olympic Games or matches where at least one of the teams was the nation's B-team, U-23 or a league select team.

### Dataset Information
`results.csv` includes the following columns:

- date (date): date of the match
- home_team (string): the name of the home team
- away_team (string): the name of the away team
- home_score (integer): full-time home team score including extra time, not including penalty-shootouts
- away_score (integer): full-time away team score including extra time, not including penalty-shootouts
- tournament (string): the name of the tournament 
- city (string): the name of the city/town/administrative unit where the match was played
- country (string): the name of the country where the match was played
- neutral (bool) - TRUE/FALSE column indicating whether the match was played at a neutral venue

`shootouts.csv` includes the following columns:
- date (date): date of the match
- home_team (string): the name of the home team
- away_team (string): the name of the away team
- winner (string): winner of the penalty-shootout
- first_shooter (string): the team that went first in the shootout

`goalscorers.csv` includes the following columns:
- date (date): date of the match
- home_team (string): the name of the home team
- away_team (string): the name of the away team
- team (string): name of the team scoring the goal
- scorer (string): name of the player scoring the goal
- own_goal (bool): whether the goal was an own-goal
- penalty (bool): whether the goal was a penalty

## Stacks
- [Docker](https://www.docker.com/): used to containerize and manage the dependencies of various components of the project.
- [Astro CLI](https://docs.astronomer.io/astro/cli/overview): Astro CLI is used to define and manage data pipelines in the project. Astro CLI makes it easy to set up Airflow in Docker environment. 
- [Apache Airflow](https://airflow.apache.org/): used to define, schedule, and monitor workflows of the data pipeline as directed acyclic graphs (DAGs).
- [dbt](https://www.getdbt.com/): used to transform the raw datasets and model the dimension tables and fact tables. 
- [Soda Core](https://www.soda.io/): used to check the data quality of raw datasets and created dimension tables and fact tables.
- [Metabase](https://www.metabase.com/): used to create visualizations in docker environment.

## Prerequisites

Before you begin, ensure you have met the following requirements:

- [Docker](https://www.docker.com/)
- [Astro CLI](https://docs.astronomer.io/astro/cli/overview)
- [Soda Core](https://www.soda.io/)
- Soda Core account (free trial available)
- Google Cloud account (free credits availabe)

## How to Run

### Installation

1. Clone this repository.
2. Make sure you have met the prerequisites.

### Configuration
Follow these configuration steps to be able to run the application.
1. Create a new project in Google Cloud and choose it.
2. Create a new bucket in GCS and name it as `international_football_results`.
3. Create a new service account
    
    ![csa](imgs/create_service_account.png)
    
    Name the service account as you like, and then add BigQuery Admin and Storage Admin roles.
    
    ![csa2](imgs/csa_2.png)
    
    Open the created service account, go to keys, click add key, and create JSON.
    
    ![csa3](imgs/csa_3.png)
    
    A JSON file will be downloaded. Rename the downloaded JSON file to `service_account`.json and then copy and replace the empty `service_account.json` in the folder `include/gcp/` with it.
4. Copy your Google Cloud project ID
    
    ![soda_project_id](imgs/soda_project_id.png)
    
    to change the value of `project_id` of `configuration.yml` in the folder `include/soda/`. 
    
    ![soda_config](imgs/soda_config.png)
    
    After that, in your soda account, create API Keys, copy the `SODA LIBRARY CONFIGURATION` and paste it in the same `configuration.yml`.

    ![soda lib config](imgs/soda_lib_config.png)
    
    ![soda_config_2](imgs/sc3.png)
    
    Doing this enable soda to send error alert to email and logs to your soda account.
5. Go to the folder `include/dbt/` and change the `project` value in the `profiles.yml` file with your Google Cloud project ID.
    
    ![dbt_profile](imgs/dbt_profiles.png)
    
    Do the same with database value of `sources.yml` in the folder `include/dbt/models/sources/`
    
    ![dbt_sources](imgs/dbt_sources.png)
 
### Usage
1. Make sure Docker is running and open the cloned repository folder in terminal / CMD.
2. Run the `astro dev start` in your terminal:
    ```console
    astro dev start
    ```
    
    ![astro start](imgs/astro_start.png)
3. Access Airflow UI at localhost in your browser, that is `localhost:8080` and login with `username = admin` and `password = admin`.
    
    ![airflow ui](imgs/airflow_ui.png)
4. Before triggering the DAG, add gcp connection in airflow first.
    Name the connection as `gcp`, choose `Google Cloud` connection type, and then fill the Keyfile Path with `/usr/local/airflow/include/gcp/service_account.json`. Test the connection, and save it if success.
    
    ![gcp connection](imgs/gcp_conn.png)
5. Trigger the DAG. Wait until all tasks are finished.

    ![Task Success](imgs/task_success.png)

4. Access Metabase UI at localhost in your browser, that is `localhost: 3000`.

## Code Explanation
The project directory should have the following structure:

- `main.py`: The main Python script for running the scraper and ingesting data into MySQL.
- `README.md`: The README file for your project.
- `lib/`: A subdirectory containing your project's Python modules.
    - `__init__.py`: An empty `__init__.py` file that marks the directory as a Python package.
    - `kyou_scraper.py`: Module for scraping product details from kyou.id.
    - `ingest_to_mysql.py`: Module for ingesting data into MySQL.
- `img/`: A subdirectory to store images used in the README.

Here is the help on `main.py`:

![main](img/main.png)

Here is the help on `kyou_scraper.py`
![kyou scraper](img/kyou_scraper.png)

Here is the help on `ingest_to_mysql.py`

![ingest to mysql](img/ingest_to_mysql.png)

## Results Explanation
![product details](img/product_details.png)

With the help of `kyou_scraper.py`, the program scrape products' detailed information, which are Title, Status, Price, Wishlist, Character, Series, Category, and Manufacturer. After that, the program clean the scraped data:
- Using regex, the program change the Price format from string "IDR 580,000Earn 580 Friendship Points" into an integer "5800000".
- Using regex, the program change the change the Wishlist format from string "1254 Wishlist" into an integer "1254".
- The program trim extra whitespaces in Title column.
- The program excludes the "Prototype Showcase" Status before writing to CSV and ingesting to MySQL because a products with this status have NULL Price.

![product wishlist](img/wishlist.png)

The program created the table `product_wishlist_series` based on the `product_details` table with columns Series, Character, Wislist_Total, Average_Wishlist. This table is used to see, which series is the most favorite among all products series. It turns out, Vocaloid series is the top 1 despite only have products with 1 character "Hatsune Miku". The series in top 2, top 3, and top 4 consecutively are SPY x FAMILY, Overlord, Majo no Tabitabi, and Jujutsu Kaisen. 

## Conclusion and Future Updates
Overall, the program worked as intended. However, the program still has many possibilites of improvement, for example:

- Implementing the program in Docker with Airflow, Spark, and PostgreSQL so that there is no need to manually change the database user configuration and there is no need to manually install each requirements.
- Add more details on product information and improve the data modelling for each tables.
- Improve the code quality, especially the ratings for each pylint:

![pylint wishlist](img/pylint_main.png)

![pylint wishlist](img/pylint_scraper.png)

![pylint wishlist](img/pylint_ingest.png)
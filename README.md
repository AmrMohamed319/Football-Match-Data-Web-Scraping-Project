# Football Match Data Web Scraping Project

## Overview

This project demonstrates how to use Selenium for web scraping to extract detailed football match data from the "Adam Choi - Football Over/Under Stats" website. The script automates the process of selecting specific countries, leagues, and seasons from dropdown menus, and then collects data on match dates, home teams, away teams, and scores. The data is compiled into a pandas DataFrame for easy analysis and manipulation.

## Features

- **Automated Navigation**: The script navigates to the "Adam Choi - Football Over/Under Stats" website and clicks on the "All matches" button to display all available matches.
- **Dynamic Dropdown Selection**: Users can interactively select a country, league, and season from dropdown menus.
- **Data Extraction**: Extracts match details including date, home team, away team, and score.
- **Data Storage**: Compiles the extracted data into a pandas DataFrame for further analysis.

## Requirements

- Python 3.x
- Selenium
- pandas
- Chrome WebDriver

## Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/yourusername/football-match-data-scraping.git
    cd football-match-data-scraping
    ```

2. **Install the required packages**:
    ```bash
    pip install selenium 
    ```

3. **Download and install Chrome WebDriver**:
    - [Chrome WebDriver](https://sites.google.com/a/chromium.org/chromedriver/downloads)

## Usage

1. **Run the script**:
    ```bash
    python scrape_football_data.py
    ```

2. **Follow the prompts**:
    - The script will display available options for countries, leagues, and seasons.
    - Input the corresponding number to select your desired options.

3. **View the results**:
    - The extracted data will be printed in the terminal and saved into a pandas DataFrame.

## Code Description

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import pandas as pd

# Initialize the webdriver
driver = webdriver.Chrome()
driver.get('https://www.adamchoi.co.uk/overs/detailed')

# Click on the "All matches" button
all_matches_button = WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((By.XPATH, '//*[@id="page-wrapper"]/div/home-away-selector/div/div/div/div/label[2]'))
)
all_matches_button.click()

# Function to select from dropdown
def select_from_dropdown(driver, dropdown_id, prompt):
    elements = driver.find_elements(By.XPATH, f'//select[@id="{dropdown_id}"]')
    options = [i.text.splitlines() for i in elements][0]
    
    df = pd.DataFrame(options, columns=[dropdown_id.capitalize()])
    df.index = range(1, len(df) + 1)
    print(f"Here are all {dropdown_id}s available to select from:\n", df)
    
    num = int(input(f"Please select the number of any {dropdown_id}:\n"))
    selected = df._get_value(num, col=dropdown_id.capitalize())
    print(f"You selected: {selected}")
    
    button = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.XPATH, f'//select[@id="{dropdown_id}"]/option[@label="{selected}"]'))
    )
    button.click()
    return selected

# Select a country
country_selected = select_from_dropdown(driver, "country", "country")

# Select a league
league_selected = select_from_dropdown(driver, "league", "league")

# Select a season
season_selected = select_from_dropdown(driver, "season", "season")

# Extracting all required data
def extract_column_data(xpath):
    elements = WebDriverWait(driver, 10).until(
        EC.presence_of_all_elements_located((By.XPATH, xpath))
    )
    return [element.text for element in elements]

# Define XPaths for different columns
date_xpath = '//*[@id="page-wrapper"]/div/div[4]/div/detailed-team/div/div/div[2]/div/div/div[2]/table/tbody/tr/td[1]'
home_team_xpath = '//*[@id="page-wrapper"]/div/div[4]/div/detailed-team/div/div/div[2]/div/div/div[2]/table/tbody/tr/td[2]'
away_team_xpath = '//*[@id="page-wrapper"]/div/div[4]/div/detailed-team/div/div/div[2]/div/div/div[2]/table/tbody/tr/td[4]'
score_xpath = '//*[@id="page-wrapper"]/div/div[4]/div/detailed-team/div/div/div[2]/div/div/div[2]/table/tbody/tr/td[3]'

# Extract data
all_dates_list = extract_column_data(date_xpath)
all_home_list = extract_column_data(home_team_xpath)
all_away_list = extract_column_data(away_team_xpath)
all_results_list = extract_column_data(score_xpath)

# Create DataFrame
all_data = pd.DataFrame({
    'Date': all_dates_list,
    'Home Team': all_home_list,
    'Away Team': all_away_list,
    'Score': all_results_list
})

print(all_data)

# Close the driver
driver.quit()


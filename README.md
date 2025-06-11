# Disaster Data Cleaning & Analysis with API Integration

## Project Overview

This project is a robust data engineering pipeline designed to clean, process, and significantly enrich a raw dataset detailing historical disaster events. The primary objective is to transform messy, incomplete raw data into a clean, structured, and feature-rich format. This enhanced dataset is then made suitable for various downstream applications, including in-depth statistical analysis, advanced data visualization, and the development of machine learning models for disaster prediction or impact assessment. A cornerstone of this project involves the intelligent integration with multiple external APIs to accurately impute and standardize missing geographical information, such as disaster locations, their corresponding latitudes, and longitudes.

## Key Features & Processing Steps

The `FINAL_YEAR_PROJECT.ipynb` Jupyter Notebook orchestrates a comprehensive series of data manipulation, cleaning, and feature engineering operations. Each stage is meticulously designed to refine the dataset iteratively.

### 1. Initial Setup & Data Loading

* **Library Importation:** The notebook begins by importing all necessary Python libraries. This includes `pandas` for efficient data manipulation, `requests` for making HTTP requests to external APIs, `time` for managing API call delays (to prevent rate limiting), `re` for powerful regular expression-based text cleaning, and `country_converter` (aliased as `coco`) for standardized country code and name conversions.
* **Data Ingestion:** The raw disaster dataset is loaded directly from an Excel file named `dataset.xlsx` into a `pandas.DataFrame` for immediate processing.

### 2. Extensive Data Selection & Column Management

* **Irrelevant Column Removal:** A significant number of columns are strategically dropped from the DataFrame. This step is crucial for streamlining the dataset by removing identifiers (`DisNo.`, `External IDs`), highly granular classifications (`Classification Key`, `Disaster Group`, `Subtype`, `Associated Disasters`, `Associated Disaster`), country-specific details (`Country`, `Region`), and various financial/damage-related columns (`Total Damage`, `Total Damage, Adjusted`, `Reconstruction Costs`, `Insured Damage`, etc.) that are not the focus of this analysis. The goal is to retain only the most pertinent information for subsequent steps.

### 3. Basic Missing Value Handling

* **Magnitude Imputation:** Initial missing values in the 'Magnitude' column (which could represent earthquake magnitude, storm intensity, etc.) are filled with a default value of `0`. This simple imputation ensures numerical operations can proceed without errors for these records.

### 4. Feature Engineering

* **Disaster Duration Calculation:**
    * A critical feature, 'duration' (in days), is engineered to represent the length of each disaster event. This is calculated by considering 'Start Month', 'End Month', 'Start Day', and 'End Day' columns.
    * Robust handling is implemented for potential negative duration values (indicating data errors or spanning across year boundaries) by capping them at `0`. Missing duration values are also filled with `0`.
* **Original Date Column Removal:** Once 'duration' is calculated, the granular date components ('Start Year', 'Start Month', 'Start Day', 'End Year', 'End Month', 'End Day') are dropped as their information has been consolidated.
* **Imputing Impact Metrics:**
    * Missing values in 'Total Deaths' are imputed based on the calculated 'duration' (e.g., `Total Deaths = duration * 1`, implying a simple proportional relationship).
    * Missing 'Total Affected' figures are subsequently imputed based on the 'Total Deaths' (e.g., `Total Affected = Total Deaths * 10`), providing a baseline estimate for overall impact.
* **Disaster Type Categorization (Binning):**
    * The categorical 'Disaster Type' column is transformed into numerical bins (0, 1, 2, 3, 4) for specific disaster types: 'Flood', 'Wildfire', 'Earthquake', 'Storm', and 'Drought'. This prepares the categorical data for numerical analysis.
    * The DataFrame is then filtered to retain only rows corresponding to these binned disaster types, focusing the analysis on well-defined disaster categories.

### 5. Geographical Data Enrichment (Multi-Stage API Integration)

This is a central and complex part of the pipeline, addressing missing location data through multiple external API calls.

* **Location Cleaning Function (`location_cleaner`):**
    * A custom Python function `location_cleaner` is defined using regular expressions. This function standardizes and cleans location names by:
        * Removing text within parentheses.
        * Replacing various separators ("and", "&", ",") with a consistent format.
        * Ensuring the first letter of each word in the location name is capitalized for consistency.
    * This function is applied to the 'Location' column multiple times throughout the notebook to ensure data cleanliness.
* **`country_converter` Integration:** The `ISO` column (representing country codes) is likely processed using `country_converter` to ensure standardized ISO 3166-1 alpha-3 codes, which are essential for reliable API calls.
* **API-Based Location Imputation & Validation:**
    * **Stage 1 (Geo-database API - Implied RapidAPI):** The notebook iterates through unique 'ISO' country codes where 'Location' values are missing. It attempts to find the capital city using a geo-database API (inferred from common use cases and variable names, though API key is not provided in the snippet). This helps fill in country-level missing locations.
    * **Stage 2 & 3 (API-Ninjas City API):** The notebook makes subsequent API calls to the API-Ninjas City API (suggested by variable names like `api_ninjas_api_key`) to obtain `Latitude` and `Longitude` for locations where these values are still missing. This is often done in stages to ensure maximum coverage, potentially using different API keys or fallback mechanisms if the first attempt fails.
    * **Stage 4 (LocationIQ API):** A final attempt to fill missing geographical coordinates (`Latitude`, `Longitude`) is made using the LocationIQ API (suggested by variable names like `locationiq_api_key`). A local caching mechanism is implemented to store API responses, preventing redundant calls for the same location and optimizing performance.
* **Intermediate Data Saving:** The dataset is saved to CSV files (`stage1_dataset.csv`, `stage2_dataset.csv`, `stage3_dataset.csv`, `stage4_dataset.csv`) after each significant API call stage. This provides checkpoints, allows for resume capabilities, and facilitates debugging.

### 6. Final Data Preparation & Output

* **Index Cleanup:** Unnecessary index columns (like `Unnamed: 0`), which may have been created during intermediate saving and loading operations, are dropped.
* **Missing Coordinate/Location Removal:** Any remaining rows with missing 'Latitude' or 'Location' values (after all API attempts) are removed from the dataset, ensuring a complete geographical record for all remaining entries.
* **Final Column Drop:** The 'Total Affected' column is dropped at this late stage, indicating it might not be needed for the ultimate analysis goal or was deemed less reliable after initial imputation.
* **Time Feature Extraction:** A new 'time' column is created by extracting the year component from the original 'DisNo.' column (likely a unique disaster identifier), matching based on the 'Location'. This provides a direct temporal feature.
* **Missing Time Removal:** Rows where 'time' could not be successfully extracted or remains missing are dropped.
* **Year Extraction:** The year is explicitly extracted from the 'time' column (ensuring it's in a clean numerical format).
* **CPI Column Removal:** The 'CPI' (Consumer Price Index) column is dropped, implying it's not relevant for the final analysis.
* **Final Dataset Export:** The fully processed, cleaned, and enriched dataset is saved as `final_dataset.csv`. This file represents the final output of the entire data pipeline.

## Dataset Information

* **Original Data Source:** The raw dataset used for this project is expected to be an Excel file named `dataset.xlsx`.
* **Processed Data File:** `final_dataset.csv` - This is the primary output of the notebook, containing the meticulously cleaned, feature-engineered, and geographically enriched disaster event data. It is ready for further analysis.

### `final_dataset.csv` Schema:

The final dataset includes the following key columns:

* **`Disaster Type`**: Numerical bins representing categorized disaster types (e.g., 0: Flood, 1: Wildfire, 2: Earthquake, etc.).
* **`ISO`**: Standardized 3-letter ISO country codes (e.g., 'USA', 'IND').
* **`Location`**: Cleaned and standardized names of the disaster locations (cities, regions).
* **`Magnitude`**: The intensity or scale of the disaster (e.g., earthquake magnitude, storm wind speed).
* **`Latitude`**: The geographical latitude of the disaster location.
* **`Longitude`**: The geographical longitude of the disaster location.
* **`Total Deaths`**: The estimated number of fatalities caused by the disaster.
* **`Total Affected`**: The estimated total number of people affected by the disaster.
* **`duration`**: The calculated duration of the disaster event in days.
* **`time`**: The extracted year of the disaster event.

## How to Run the Notebook

To replicate the data processing and generate the `final_dataset.csv`, follow these steps:

1.  **Clone the Repository:**
    ```bash
    git clone [https://github.com/YourUsername/YourRepoName.git](https://github.com/YourUsername/YourRepoName.git)
    cd YourRepoName
    ```
    *(Replace `YourUsername/YourRepoName` with the actual path to your repository on GitHub.)*

2.  **Set Up Python Environment:**
    It's highly recommended to use a virtual environment to manage project dependencies:
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Linux/macOS
    # venv\Scripts\activate   # On Windows
    ```

3.  **Install Required Libraries:**
    Install all the necessary Python packages. You can typically find these in a `requirements.txt` file (if provided) or install them manually:
    ```bash
    pip install pandas requests country_converter numpy re # Add any other specific libraries used in your notebook
    ```
    *(**Pro Tip:** After installing all dependencies, you can generate a `requirements.txt` file from your active environment using `pip freeze > requirements.txt` for future reference.)*

4.  **Obtain & Configure API Keys (Crucial Step):**
    This notebook relies on external APIs. You **must** obtain your own API keys from the respective providers:
    * **API-Ninjas:** For city data (Latitude/Longitude).
    * **LocationIQ:** For geocoding services (Latitude/Longitude).
    * *(Potentially RapidAPI for a geo-database API if that was the initial source)*

    **NEVER hardcode API keys directly into your notebook or commit them to GitHub.** Instead, use environment variables or a configuration file that is added to your `.gitignore`.
    * **Environment Variables (Recommended):** Set your API keys as environment variables in your system or within your Colab/Jupyter environment. For example:
        ```bash
        export API_NINJAS_KEY="your_api_ninjas_key_here"
        export LOCATION_IQ_KEY="your_location_iq_key_here"
        ```
        Then, in your notebook, access them via `import os; api_key = os.getenv("API_NINJAS_KEY")`.
    * **`.env` file (for local development):** Create a file named `.env` in your project root (and add `.env` to your `.gitignore`).
        ```
        API_NINJAS_KEY=your_api_ninjas_key_here
        LOCATION_IQ_KEY=your_location_iq_key_here
        ```
        Then use a library like `python-dotenv` to load them:
        ```python
        from dotenv import load_dotenv
        import os
        load_dotenv() # take environment variables from .env.
        api_key = os.getenv("API_NINJAS_KEY")
        ```
    * **Update your notebook's API key lines** to read from these secure sources.

5.  **Place Raw Data:**
    * Ensure your original `dataset.xlsx` file is placed in the same directory as your `FINAL_YEAR_PROJECT.ipynb` notebook. If you place it elsewhere (e.g., a `data/` folder), remember to update the file path in the notebook's data loading cell.

6.  **Execute the Notebook:**
    * Open `FINAL_YEAR_PROJECT.ipynb` using Jupyter Notebook, JupyterLab, or upload it to Google Colaboratory.
    * Run all cells sequentially, ensuring that each cell completes successfully before proceeding.

## Future Work / Possible Enhancements

This project lays a strong foundation for further analysis. Here are some potential directions for enhancement:

* **More Advanced Imputation Techniques:** Explore sophisticated methods for handling missing values, such as using `scikit-learn`'s `IterativeImputer` (MICE) or `KNNImputer`, which can leverage relationships between columns for more accurate imputation.
* **Robust Error Handling for APIs:** Implement more granular error handling for API calls, including retry mechanisms with exponential backoff, circuit breakers, and comprehensive logging to better manage API rate limits and temporary network issues.
* **Comprehensive Data Validation:** Introduce more rigorous data validation checks at various stages of the pipeline to ensure data integrity, consistency, and adherence to expected formats (e.g., latitude/longitude bounds, realistic death/affected counts).
* **In-depth Data Analysis & Visualization:** Utilize the cleaned dataset to perform detailed statistical analysis, identify trends, and create compelling data visualizations (e.g., maps of disaster occurrences, time-series plots of duration/impact, correlation matrices).
* **Machine Learning Model Development:** Develop and train machine learning models for tasks such as:
    * Predicting the total deaths or affected population for future disasters.
    * Classifying disaster types based on geographical and temporal features.
    * Identifying factors contributing to higher disaster severity.
* **Pipeline Automation:** Consider containerizing the pipeline (e.g., using Docker) or orchestrating it with tools like Apache Airflow for automated execution.
* **Interactive Dashboards:** Create interactive dashboards (e.g., using Dash, Streamlit) to allow users to explore the disaster data dynamically.

---

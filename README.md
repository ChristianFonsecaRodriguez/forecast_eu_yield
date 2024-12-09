# EU YIELD FORECAST

This project aims to make a Forecast for the month of 2022 from the EU Crop Yield data extracted from HuggingFace using Meteorology variables.

## Table of Contents

1. [Description](#description)
2. [Structure and Features](#features)
3. [Installation](#installation)
4. [Usage](#usage)
5. [Results](#results)
6. [License](#license)

---

## Description

This project is a challenge aimed at developing a forecasting model for yield prediction. To achieve this, I have undertaken the following:

1. All Crops Forecasting
  * Using a Regressor Model
  * Using a Time Series Model

2. Top Crop Forecasting
  * Using a Regressor Model
  * Using a Time Series Model

---

## Structure and Features

Below is the folder structure of the project with their respective subfolders:

```bash
forecast_eu_yield/
├── data/                   
│   ├── train/              # Training data
│   ├── test/               # Evauation data
│   └── forecast/           # Foracsting data
├── imputations/            # Pkg for imputations 
├── notebooks/              # Notebooks for Data Acquisition, PRocessing and Modelling
├── results/                # Basic Results for metrics
├── src/                    # App for Backend
└── requirements.txt        # Project dependencies
```

The `data` folder contains location information and datasets for all crops and the top crop. Meteorological data (metheorology.csv), is not allowed here, because it's size is up to 100MB, here is the [link](https://drive.google.com/file/d/1D2_be_lem5IyDGzZdFWiWP-6-j2nBFBi/view?usp=sharing) to download it.

Models are registered in a personal MLflow instance: [MLflow Access](http://ec2-54-160-110-158.compute-1.amazonaws.com:5000/) (available until December 9, 2024).


### Notebooks

* `1_Data_Acquisition_PreProcessing.ipynb`: This notebook handles the acquisition of yield data from HuggingFace, the acquisition of location data (latitude and longitude) using the Google Maps API, scraping of EU codes, and scraping of climate data using NASA's Weather API. Additionally, the data was preprocessed for ALL CROPS and TOP CROP (the top crop was selected using the Sharpe Ratio method), resulting in a dataset universe for each.

    Additionally, it was necessary to select a harvest start point to reference the months associated with climate variables. Therefore, the harvest month was selected using [this crop calendar](https://ipad.fas.usda.gov/rssiws/al/crop_calendar/europe.aspx), where September is the most common month for harvesting, and some seed data for the assessment is also available (e.g., sunflower). The meteorology data will have 8 months for each feature (January, February, ..., July, August).

    ![Crop Calendar](https://ipad.fas.usda.gov/countrysummary/images/E4/cropcalendar/europe_e4_1_calendar.png)

* `2_Models.ipynb`: This notebook involves the development of around 20 models, reviewing tests and the best ways to automate the training process (in this case using PyCaret). A greater focus was given to ALL CROPS, and this approach was replicated for the TOP CROP.

* `3_Final_Models.ipynb` : This notebook provides the training and testing of the best models identified in `2_Models.ipynb`. Models are registered in `mlflow`, and forecast results are saved in the `data/forecast` folder. Two models were used for both ALL CROP and TOP CROP datasets: an optimized regression model with LightGBM and a time-series model using Holt-Winters.

### Dataset Features

* **country**: EU country
* **code**: EU country code
* **province**: EU provinces within each country
* **year**: Crop years (2000-2022)
* **t2m**: Temperature at 2 meters for each month (e.g., `t2m_M1` avg for January)
* **rh2m**: Relative Humidity at 2 meters for each month (e.g., `rh2m_M1` avg for January)
* **ws10m**: Wind Speed at 10 meters for each month (e.g., `ws10m_M1` avg for January)
* **ps**: Surface Pressure for each month (e.g., `ps_M1` avg for January)
* **frost_days**: Frost days for each month (e.g., `frost_days_M1` sum for January)
* **snodp**: Snow depth for each month (e.g., `snodp_M1` avg for January)
* **pw**: Precipitable water for each month (e.g., `pw_M1` avg for January)

---

## Installation

Follow these steps to install the project locally.

- Install Python 3.10.11

```bash
# Clone the repository
git clone https://github.com/ChristianFonsecaRodriguez/forecast_eu_yield.git

# Navigate to the project directory
cd forecast_eu_yield

# Create a virtual environment (optional, but recommended)
python -m venv venv
source venv/bin/activate  # For Linux/MacOS
venv\Scripts\activate     # For Windows

# Install dependencies
pip install -r requirements.txt
```

## Usage

To get started, navigate to the `notebooks` folder and create a `.env` file, which will contain a Google Maps token credential (since this project uses the Google Maps API to extract the latitude and longitude of locations provided by country and province). This is only necessary if the `use_maps_api_key=True` option is enabled in the `1_Data_Adquisition_PreProcessing.ipynb` notebook.

```bash .env
API_KEY_GEOCODING=YOUR_TOKEN
```

### Steps to Run the Project
1. Data Acquisition and Preprocessing
    * Open `1_Data_Acquisition_PreProcessing.ipynb` in the notebooks folder.
    Ensure the .env file is correctly set up with your Google Maps API key if `use_maps_api_key=True`.
    * Download the meteorological data from [here](https://drive.google.com/file/d/1D2_be_lem5IyDGzZdFWiWP-6-j2nBFBi/view?usp=sharing) and move it to the `data/` folder. If you want to scrape new data, set `use_scrap_nasa_weather=True`. This process will take approximately 15–20 minutes.
    * Run all cells to acquire and preprocess the data, getting all universes.
    
2. Model Development
    * Open `2_Models.ipynb` in the notebooks folder.
    * Execution is not required, as it is designed to showcase the development process of the models. However, the results and their explanations can be reviewed.
    * I focused more on developing a universal model for all crops, and this knowledge was later extrapolated to the top crop.
    * Finally, the final output is the implementation of a segmented regressor model (using three models based on the segment) and the use of Holt-Winters as the base for time series forecasting (since there is limited annual data available for forecasting). 

3. Final Model Training and Forecasting
    - Open `3_Final_Models.ipynb` in the notebooks folder.
    - This notebook trains and tests the best models from `2_Models.ipynb`.
    - Models are registered in MLflow, and forecast results are saved in the data/forecast folder.

## Results

The results are based on the developed models, where **LightGBM** was used for regression models and **Holt-Winters** for time series models. Also, the models were developed for **All Crops** and **All Crops** (where in the Analysis using Sharpe Ratio, Top Crop was **Cereals for the production of grain including seed**).

### LighGBM Regressor

#### Model Grouping by Country

The regression models were developed by grouping countries because it was observed that, in the last three years, their yield levels varied significantly. Consequently, the countries were separated into three distinct groups. This grouping was applied to both **All Crops** and **All Crops** models.

| **Group** | **Countries**                    |
|-----------|----------------------------------|
| **G1**    | France                           |
| **G2**    | Italy, Türkiye, Poland, Spain    |
| **G3**    | All other countries              |

- **G1: France**
  - France was analyzed separately due to its unique yield patterns over the past three years.

- **G2: Italy, Türkiye, Poland, Spain**
  - These countries exhibited similar yield trends, making it effective to group them together for modeling purposes.

- **G3: All Other Countries**
  - The remaining EU countries, which did not fit into G1 or G2, were grouped together to capture their general yield behavior.

#### Model Performance

Below is a table showcasing the performance metrics (MAE in Tons and R²) for each model within the defined country groups (2022 Test):

*Note: The MAE was calculated for each granular value of country/province. After observing a decreasing MAE, the final decision was to group by country only for the predicted Yield (result delivered to the forecast folder).*

##### All Crops and Top Crop Results

| Model                        |   MAE   | R²   |
|------------------------------|---------|------|
| LightGBM (All Crops) - G1    | 1307.14 | 0.93 |
| LightGBM (All Crops) - G2    | 1974.22 | 0.81 |
| LightGBM (All Crops) - G3    |  581.10 | 0.87 |
| LightGBM (Top Crop) - G1     |  124.94 | 0.97 |
| LightGBM (Top Crop) - G2     |  307.68 | 0.84 |
| LightGBM (Top Crop) - G3     |  149.97 | 0.86 |

As an example of the provided graphs, the following is shown for the Top Crop - G3 model.

![r2image](/images/r2_example.png)

*Figure: Result of the fit based on actual versus predicted data (for R²).*

![importanceimage](/images/importance.png)

*Figure: Feature Importance. The province is the most desired feature for this model (Indicated to PyCaret that it is a categorical variable to transform accordingly).*

### Holt-Winters

#### Model Dataset

For this case, all data was grouped by country to construct the series by year. The Holt-Winters method was applied only to countries that did not have zeros in any year, as it is highly sensitive to such changes due to being a deterministic method (if the parameters are static). For this model I'm not using Weather data.

#### Model Performace

Since a model was developed for each country, the Yield data could vary based on this approach, as was also observed in the LightGBM models. Therefore, in addition to MAE, MAPE was used to evaluate the reduction of error by country. Each of this metric are in the tables below (for test).

##### All Crops Yield Results

|  country	 |    MAE    | MAPE (%) |
|------------|-----------|----------|
|austria	 |   1449.48 |    5.12  |
|belgium	 |   5252.37 |	 21.49  |
|czechia	 |   2894.85 |	 13.53  |
|finland	 |   1379.98 |	  5.28  |
|france	     | 205097.77 |	 54.27  |
|greece	     |   3287.03 |	 19.81  |
|hungary	 |  13649.36 |	 38.30  |
|luxembourg  |    124.19 |    5.53  |
|netherlands |   1858.34 |    6.92  |
|poland	     |   3252.82 |	  2.39  |
|portugal	 |    789.91 |	 13.94  |
|romania	 |   3433.18 |	  8.33  |
|slovakia	 |    438.18 |	  2.81  |

Where full metrics are:

|  Metric  |   mean   |   pond    |
|----------|----------|-----------|
| MAE      | 18685.19 | 123472.77 |
| MAPE (%) |   15.21  |    36.77  |

![hwfrall](/images/hw_fr_all.png)

*Figure: Example of Forecasting 2022 for France - All Crops*

##### Top Crop Yield Results

|  country	|   MAE	  | MAPE (%) |
|-----------|---------|----------|
|austria	|  251.54 |   4.83   |
|belgium	|  253.21 |	  9.10   |
|czechia	|  947.04 |	 11.52   |
|finland	|  804.35 |	 22.40   |
|france	    | 5636.11 |	  9.40   |
|greece	    |  666.00 |	 20.77   |
|hungary	| 2557.61 |	 28.24   |
|luxembourg |	16.12 |   9.44   |
|poland	    | 2867.19 |	  8.19   |
|portugal	|  124.15 |	 12.17   |
|romania	| 2865.80 |	 15.19   |

Where full metrics are:

|  Metric  |   mean  |   pond  |
|----------|---------|---------|
| MAE      | 1544.46 | 3690.30 |
| MAPE (%) |  13.75  |  11.88  |


![hwgrtop](/images/hw_gr_top.png)

*Figure: Example of Forecasting 2022 for Greece - Top Crop.*

## License

This project is licensed under the [MIT License](https://opensource.org/license/MIT).

## Acknowledgements

- [HuggingFace](https://huggingface.co/) for providing access to the yield data.
- [Google Maps API](https://developers.google.com/maps) for location data services.
- [NASA Weather API](https://api.nasa.gov/) for climate data.
- [PyCaret](https://pycaret.org/) for simplifying the machine learning workflow.
- [MLflow](https://mlflow.org/) for model tracking and management.

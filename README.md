# EU YIELD FORECAST

This project aims to make a Forecast for the month of 2022 from the EU Crop Yield data extracted from HuggingFace using Meteorology variables.

## Table of Contents

1. [Description](#description)
2. [Structure and Features](#features)
3. [Installation](#installation)
4. [Usage](#usage)
5. [Screenshots](#screenshots)
6. [Contributing](#contributing)
7. [License](#license)

---

## Description

This project is a challenge aimed at developing a forecasting model for yield prediction. To achieve this, I have undertaken the following:

1. All Crops Forecasting
  - Using a Regressor Model
  - Using a Time Series Model

2. Top Crop Forecasting
  - Using a Regressor Model
  - Using a Time Series Model

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

- `1_Data_Acquisition_PreProcessing.ipynb`: This notebook handles the acquisition of yield data from HuggingFace, the acquisition of location data (latitude and longitude) using the Google Maps API, scraping of EU codes, and scraping of climate data using NASA's Weather API. Additionally, the data was preprocessed for ALL CROPS and TOP CROP (the top crop was selected using the Sharpe Ratio method), resulting in a dataset universe for each.

    Additionally, it was necessary to select a harvest start point to reference the months associated with climate variables. Therefore, the harvest month was selected using [this crop calendar](https://ipad.fas.usda.gov/rssiws/al/crop_calendar/europe.aspx), where September is the most common month for harvesting, and some seed data for the assessment is also available (e.g., sunflower).

    ![Crop Calendar](https://ipad.fas.usda.gov/countrysummary/images/E4/cropcalendar/europe_e4_1_calendar.png)

- `2_Models.ipynb`: This notebook involves the development of around 20 models, reviewing tests and the best ways to automate the training process (in this case using PyCaret). A greater focus was given to ALL CROPS, and this approach was replicated for the TOP CROP.

- `3_Final_Models.ipynb` : This notebook provides the training and testing of the best models identified in `2_Models.ipynb`. Models are registered in `mlflow`, and forecast results are saved in the `data/forecast` folder. Two models were used for both ALL CROP and TOP CROP datasets: an optimized regression model with LightGBM and a time-series model using Holt-Winters.

### Dataset Features

- **country**: EU country
- **code**: EU country code
- **province**: EU provinces within each country
- **year**: Crop years (2000-2022)
- **t2m**: Temperature at 2 meters for each month (e.g., `t2m_M1` is for January)
- **rh2m**: Relative Humidity at 2 meters for each month (e.g., `rh2m_M1` is for January)
- **ws10m**: Wind Speed at 10 meters for each month (e.g., `ws10m_M1` is for January)
- **ps**: Surface Pressure for each month (e.g., `ps_M1` is for January)
- **frost_days**: Frost days for each month (e.g., `frost_days_M1` is for January)
- **snodp**: Snow depth for each month (e.g., `snodp_M1` is for January)
- **pw**: Precipitable water for each month (e.g., `pw_M1` is for January)

---

## Installation

Follow these steps to install the project locally.

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
    - Open `1_Data_Acquisition_PreProcessing.ipynb` in the notebooks folder.
    Ensure the .env file is correctly set up with your Google Maps API key if `use_maps_api_key=True`.
    - Download the meteorological data from [here](https://drive.google.com/file/d/1D2_be_lem5IyDGzZdFWiWP-6-j2nBFBi/view?usp=sharing) and move it to the `data/` folder. If you want to scrape new data, set `use_scrap_nasa_weather=True`. This process will take approximately 15–20 minutes.
    - Run all cells to acquire and preprocess the data, getting all universes.
    
2. Model Development
    - Open `2_Models.ipynb` in the notebooks folder.
    - Execution is not required, as it is designed to showcase the development process of the models. However, the results and their explanations can be reviewed.
    - I focused more on developing a universal model for all crops, and this knowledge was later extrapolated to the top crop.
    - Finally, the final output is the implementation of a segmented regressor model (using three models based on the segment) and the use of Holt-Winters as the base for time series forecasting (since there is limited annual data available for forecasting). 

3. Final Model Training and Forecasting
    - Open `3_Final_Models.ipynb` in the notebooks folder.
    - This notebook trains and tests the best models from `2_Models.ipynb`.
    - Models are registered in MLflow, and forecast results are saved in the data/forecast folder.

## Screenshots

## Contributing

## License

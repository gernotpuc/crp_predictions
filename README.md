# Pretrained CrP prediction models
Pretrained models for performing next-day CRP predictions after start of antibiotic therapy.

The Notebook **crp_forecasts** can be used for performing predictions on a specified dataset of patient specific CrP values.

To use the notebook with you own data, please read the section on **Data requirements** carefully.

## Instructions
- Extract patient cohort (see section **Cohort definition**)
- Prepare CSV-files as model **input** (see section **Data requirements** for further descriptions)
- Clone the repositpory to your local machine
- Adapt the paths to your input files in the Jupyter-Notebook **crp_forecasts** (see Step 0 in the Notebook)
- Install libraries based on  **requirements.txt**
- Choose a trained model from the list of models CRP forecasts should be based on (see Step 2 in the Notebook)
- Execute the Jupyter-Notebook **crp_forecasts**
- It will produce the following **outputs**:
  - Performance metrics as print (see Step 3 in the Notebook)
  - Dataframes with next day CRP-predictions and actual values (see Step 3 in the Notebook)
  - Plots of forecasts for specific patients (see Step 4 in the Notebook)

## Introduction
Internationally, and also in Germany, the prevalence of multi-resistant pathogens, which do not respond to antibiotics or only respond to them to a limited extent, is increasing. The main cause is the incorrect prescription and application of antibiotics, in particular the unnecessary and excessively long treatment with broad-spectrum antibiotics. 

In addition to vital parameters, inflammation levels, especially C-reactive protein (CrP), are the most important clinical parameters for sepsis. However, the response of CRP levels to antibiotic treatment is delayed due to a half-life of 19 hours. Thus, despite overall clinical improvement, an increase in CrP is often still detected, which can lead to an (unnecessary) intensification of antibiotic therapy. The reliable prediction of CrP can therefore contribute to a more restrictive antibiotic therapy. 

Using the FHIR data set with a cohort of 398 patients at the University Hospital Essen, various monocentric, global time series models were trained, which predict patient-specific CrP values.
Models were intentionally kept simple in order to keep the data requirements as low as possible. Thus, no covariates were used here and predictions are made only based on the historic CrP values and time stamps of administered antibiotics.

On an internal validation dataset with 398 time series, the following performances could be achieved for the next day CrP-prediction after start of antibiotic therapy in a 5-fold cross validation with 20 repeats:

| Model                                                             | MAE  | RMSE | MSE   | MAPE  |
|-------------------------------------------------------------------|------|------|-------|-------|
| Gradient Boosting (LightGBM)                                      | 3.35 | 4.73 | 22.40 | 35.59 |
| Neural Network (NBEATS)                                           | 3.30 | 4.80 | 23.06 | 30.58 |
| Weighted Ensemble of NNs (DeepAR, TiDE, PatchTST, DLinear, TFTM)  | 3.67 | 5.13 | 26.39 | 36.64 |
| Zero-shot Large Language Model (Chronos)                          | 3.96 | 5.65 | 32.01 | 39.13 |
| Baseline model: Average Forecast                                  | 7.70 | 10.72| 98.53 | 60.69 |
| Baseline model: Naive Forecast                                    | 4.82 | 7.18 | 26.88 | 42.47 |


### Cohort definition:<br>  

Please extract your patient cohort based on the following criteria:

- Patient >= 18 years. Selection sufficient, no birth dates required in the dataset.
- Cancer diagnosis with ICD Code C00-96, first registration after 01-01-2020. Selection sufficient, no ICD-10 codes required in the dataset.
- CRP values, recorded during stationary hospital stays. An encounter identifier, a subject identifier, time stamps of the blood collection and the respective CRP values must be provided.
- Medications: Intravenously administered antibiotics with one of the following ATC codes. An encounter identifier, a subject identifier and time stamps of the antibiotic administration must be provided. Encounter identifiers should match those in the dataset with CRP values.
ATC codes:
J01DB, J01DC , J01DD, J01DI, J01DH, J01CR50, J01CR, J01CE, J01CA, J01C, J01FA, J01G, J01M, J01FF, J01XX08, J01XA

### Data requirements:<br>  
CSV files of patient specific CRP values and administered antibiotics are used as inputs for the prediction models.

**File syntax:**

The **laboratory values** file must contain the following columns:

- encounter_id --> Unique identifier of each hospital stay; *any datatype*
- subject_reference  --> Unique identifier of each patient; *any datatype*
- category --> Type of variable; *any datatype*
- recorded_time --> Timestamp of the recording; *should be valid timestamps, please stick to the given examples if possible*
- value -->  Value of the recording; *integer or float*

**encounter_id,subject_reference,category,recorded_time,value**<br>
Encounter/dsadasf34f,Patient/d23423fdsdf,CRP,2021-09-27 08:03:00+00,1.6<br>
Encounter/dsadasf34f,Patient/d23423fdsdf,CRP,2021-09-28 12:45:00+00,2.8<br>
Encounter/34fsd2ff34,Patient/sdasd23dq14,CRP,2023-01-03 00:10:00+00,1.1


The file for the **antibiotics administered** must contain the timestamps of antibiotic administrations per hospital encounter. Please structure the file as in the following example:

**encounter_id,recorded_time**<br>
Encounter/dsadasf34f,2021-09-27 06:00:00+00<br>
Encounter/dsadasf34f,2021-09-27 12:30:00+00

### Requirements:<br>
Please use requirements.txt file for installing required packages.<br>

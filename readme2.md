
# Retail Project

## Dataset

The dataset for this project is available on Kaggle and can be accessed [here](https://www.kaggle.com/datasets/tunguz/online-retail).

### Columns Description

| Column      | Description                                                                                                             |
|-------------|-------------------------------------------------------------------------------------------------------------------------|
| InvoiceNo   | Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction.                               |
| StockCode   | Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product.                     |
| Description | Product (item) name. Nominal.                                                                                           |
| Quantity    | The quantities of each product (item) per transaction. Numeric.                                                          |
| InvoiceDate | Invice Date and time. Numeric, the day and time when each transaction was generated.                                     |
| UnitPrice   | Unit price. Numeric, Product price per unit in sterling.                                                                 |
| CustomerID  | Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer.                                 |
| Country     | Country name. Nominal, the name of the country where each customer resides.                                              |

## Pipeline

![Data Pipeline](https://github.com/Javid912/Airflow_GCP_ETL/blob/main/images/image1.png)


### Data Modeling

![Data Modeling](https://github.com/Javid912/Airflow_GCP_ETL/blob/main/images/image2.png)

## Prerequisites

- Docker
- Astro CLI
- Soda
- Google Cloud account

## Steps

**IMPORTANT!**
**Open the Dockerfile and make sure you use [quay.io/astronomer/astro-runtime:8.8.0](http://quay.io/astronomer/astro-runtime:8.8.0) in the Dockerfile (or airflow 2.6.1), If not, use that version and restart Airflow (astro dev restart with the Astro CLI)**

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/tunguz/online-retail) and store the CSV file in `include/dataset/online_retail.csv`.
2. Add `apache-airflow-providers-google==10.3.0` to `requirements.txt` and restart Airflow.
3. Create a Google Cloud Storage (GCS) bucket with a unique name `<your_name>_online_retail`.
4. Create a service account named `airflow-online-retail` and grant admin access to GCS and BigQuery. Create a JSON key for the service account and save it as `service_account.json` in `include/gcp/`.
5. Add the Google Cloud connection in Airflow with the service account key.
6. Create the DAG for loading the dataset into GCS.
7. Test the task for uploading the CSV to GCS.
8. Create an empty dataset in BigQuery.
9. Create a task for loading the CSV file into a BigQuery table.
10. Install Soda Core and create a configuration file `configuration.yml`.
11. Create a Soda Cloud account and add the API key to the configuration file.
12. Test the connection and create quality check YAML files for raw invoices.
13. Install Cosmos-DBT and set up the required files and configurations.
14. Run the DBT models to transform the data.
15. Add quality check YAML files for transformed data.
16. Add a task for running quality checks on transformed data.
17. Test the task for quality checks.

### Reports

**Note:** The reports section is yet to be completed.

```
    - [ ]  In `include/dbt/models/report`
        ```sql
        -- daily_revenue.sql
        
        -- Get daily revenue
        SELECT
            date_part('date', datetime) AS date,
            SUM(total) AS revenue
        FROM fct_invoices
        WHERE datetime BETWEEN TIMESTAMP_TRUNC(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY), DAY) AND TIMESTAMP_TRUNC(CURRENT_TIMESTAMP(), DAY)
        GROUP BY date
        ```
        
        ```sql
        -- monthly_revenue.sql
        
        -- Get monthly revenue
        SELECT
            EXTRACT(YEAR FROM datetime) AS year,
            EXTRACT(MONTH FROM datetime) AS month,
            SUM(total) AS revenue
        FROM fct_invoices
        WHERE datetime BETWEEN TIMESTAMP_TRUNC(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 12 MONTH), MONTH) AND TIMESTAMP_TRUNC(CURRENT_TIMESTAMP(), MONTH)
        GROUP BY year, month
        ```
        
        ```sql
        -- country_revenue.sql
        
        -- Get revenue by country
        SELECT
            dc.iso,
            SUM(total) AS revenue
        FROM fct_invoices fi
        INNER JOIN dim_customer dc ON fi.customer_id = dc.customer_id
        GROUP BY dc.iso
        ```
        
    - [ ]  Test the reports
        
        ```bash
        astro dev bash
        dbt run --models report
        ```
        
    **🏆 First dbt reports in place!**
    
### Visualization

- [ ]  Install Google Cloud SDK
    
    ```bash
    # Google Cloud SDK
    RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - && \
    apt-get update -y && \
    apt-get install google-cloud-sdk -y
    ```
    
- [ ]  Create a new file `include/viz/plotly.py`
    
    ```python
    import plotly.express as px
    from google.cloud import bigquery
    
    client = bigquery.Client()
    
    def plot_country_revenue():
        query = """
        SELECT
            dc.iso,
            SUM(total) AS revenue
        FROM airtube-390719.retail.fct_invoices fi
        INNER JOIN airtube-390719.retail.dim_customer dc ON fi.customer_id = dc.customer_id
        GROUP BY dc.iso
        """
        df = client.query(query).to_dataframe()
        
        fig = px.bar(df, x='iso', y='revenue', title='Revenue by Country')
        fig.show()
    ```
    
- [ ]  Test the visualization
    
    ```bash
    astro dev bash
    python include/viz/plotly.py
    ```
    
**🏆 First visualization in place!**

## Conclusion

Congratulations! seting up a data pipeline for the retail project is finished. Here's what was accomplished:

1. **Dataset Acquisition**: Obtained the dataset from Kaggle.
2. **Data Modeling**: Defined a data model and transformed the raw data into structured tables using dbt.
3. **Data Quality**: Implemented data quality checks to ensure the integrity of the data.
4. **Reports**: Created SQL queries to generate daily, monthly, and country-wise revenue reports.
5. **Visualization**: Developed a visualization to display revenue by country using Plotly.



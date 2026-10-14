[README.md](https://github.com/user-attachments/files/33410630/README.md)
# Ask A Manager Salary Survey 2021: Data Cleaning and Dashboard

**What do professionals earn, and what drives it?**

An end-to-end project: a messy, self-reported salary survey cleaned in Python (pandas) and explored in a Power BI dashboard.

![Dashboard overview](images/overview.png)

## At a glance

| | |
|---|---|
| **Question** | How do pay levels change by role, industry, experience, education, gender, race and location? |
| **Data** | Ask A Manager Salary Survey 2021, ~28,000 self-reported responses |
| **Final dataset** | 27,237 rows after cleaning (confirm with `len(df)`) |
| **Tools** | Python (pandas, pycountry, rapidfuzz, requests), Power BI, DAX |
| **Output** | Clean CSV, data dictionary, and a 6-page Power BI dashboard |

## Data

- Source: [Ask A Manager Salary Survey 2021](https://docs.google.com/spreadsheets/d/1IPS5dBSGtwYVbjsfbaMCYIWnOuRmJcbequohNxCyGVw), a public Google Sheet.
- The survey is open text for most fields, so country, state, city, industry and job title come in thousands of spellings.
- The raw data is not stored in this repository; the notebook reads it directly from the sheet.

## What I did

### 1. Cleaning (Python, pandas)

- **Countries:** standardized ~28,000 free-text answers using `pycountry`, fuzzy matching with `rapidfuzz` and a manual alias dictionary. Only unique values were processed, then mapped back to all rows. Unresolved answers (cities, non-answers) were reviewed and mapped by hand.
- **Currency to USD:** converted every salary with ECB exchange rates from the Frankfurter API, using the rates for the survey period. `AUD/NZD` was split into AUD or NZD using the respondent's country.
- **Outliers:** kept only 2021 responses (93% of the data). Salaries between 20 and 999 looked like values typed in thousands, so they were multiplied by 1,000 and flagged in `salary_rescaled`. Final range: 10,000 to 500,000 USD. Additional compensation above 2x the salary was set to null.
- **Job titles:** standardized spelling and capitalization with a replacement dictionary, then grouped ~14,000 distinct titles into **job families** (Software & IT, Education & Library, Finance & Legal, and others) using keyword rules.
- **Categorical fields:** simplified multi-select race answers, grouped small categories into `Other`, and unified Washington D.C. variants. States have `Non-US` and `Unknown` labels instead of blanks. Education was renamed to a clearer scale.
- **Ordered categories:** created `_rank` columns for age, experience and education so Power BI sorts them logically instead of alphabetically.

Every column is documented in [`data/data_dictionary.csv`](data/data_dictionary.csv).

### 2. Dashboard (Power BI)

| Page | Question it answers |
|---|---|
| Cover | What is this, where does the data come from, and what are its limits? |
| Overview | What does the typical salary look like? |
| Role & Industry | What does the market pay by role and sector? |
| Career | How do experience, education and age shape pay? |
| Equity | Are there pay gaps by gender or race? |
| Geography | Where does pay stand across US states? |

All comparisons use the **median**, which is less distorted by outliers than the average. Charts hide groups with fewer than 30 responses, and sample sizes appear in the tooltips.

Main DAX measures: median salary and total compensation, response count, share with extra compensation, gender gap, and top-paid job family and state. The measures are listed in [`powerbi/measures.md`](powerbi/measures.md).

## Key findings

- The median annual salary is about $74.5K USD, and median total compensation is about $77K.
- [Finding about experience, with numbers: for example, 0-4 years vs 21+ years]
- [Finding about job family or industry, with numbers]
- [Finding about gender gap, with the unadjusted percentage and its caveat]

*Fill these with the final figures from the dashboard.*

## Limitations

- **Self-selected sample:** respondents are readers of the Ask A Manager workplace advice blog, not a random sample of workers.
- **Skewed toward office and professional jobs:** about 9 in 10 respondents hold a college degree or higher, and women and white respondents are over-represented. Results do not describe the general workforce.
- **Nominal salaries:** converted at 2021 exchange rates and not adjusted for cost of living.
- **Job families come from keyword rules,** so some titles are misclassified (about 13% remain in `Other`).
- **Gender and race gaps are unadjusted:** they do not control for role, seniority or experience.

## Lessons learned

- **Exchange rates:** my first conversion averaged rates over five years instead of the survey period, which distorted non-USD salaries. Checking medians against known 2021 rates caught it.
- **Regional settings:** Power BI read decimal points as thousands separators and inflated salaries. Loading the CSV with the English (US) locale fixed it.
- **Rules before models:** for ~14,000 job titles, keyword rules were transparent and easy to debug, and gave a reasonable result without training a model.

## Repository structure

```
ask-a-manager-salary-survey-2021/
├── README.md
├── notebook/        Cleaning notebook (.ipynb)
├── data/            Clean CSV, data dictionary, exchange rates used
├── powerbi/         Dashboard (.pbix), theme (.json), measures.md
├── images/          Dashboard screenshots
└── docs/            Notes on cleaning decisions
```

## How to reproduce

1. Install the dependencies: `pip install pandas pycountry rapidfuzz requests`
2. Run the notebook in `notebook/` from top to bottom. It reads the public sheet and writes `ask_a_manager_clean.csv`.
3. Open the `.pbix` in Power BI Desktop and click **Refresh**. If salaries look inflated, load the CSV with the English (US) locale.

## Author

[Your name] | [LinkedIn](https://www.linkedin.com/) | [GitHub](https://github.com/)

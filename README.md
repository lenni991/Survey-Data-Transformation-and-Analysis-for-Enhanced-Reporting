# Survey Monkey Data Analysis and Transformation

This project processes and analyzes survey data from a Survey Monkey export. The primary goal is to clean, transform, and restructure the data for further analysis and reporting. The process is implemented in two main stages: first, initial data cleaning and preparation within the Excel file itself, and second, further processing and analysis using a Jupyter Notebook (`data_manipulation.ipynb`).


*   Transformed and cleaned a raw Survey Monkey dataset of 198 responses and 100 initial features, reducing it to 13 key features, improving data quality and focus for analysis.
*   Implemented data reshaping (wide-to-long format conversion) using Python's pandas, increasing the number of data points from 198 to 17,028, enabling granular analysis of individual responses.
*   Integrated respondent demographics with answer data, allowing for analysis of response patterns across different demographic groups (e.g., division, position level, generation, gender, tenure).
*   Developed a data pipeline to calculate and visualize answer frequencies, providing insights into the most common responses and potential areas of interest or concern within the survey.

## Project Workflow

### Stage 1: Excel Data Pre-processing (within "Edited Data - Survey Monkey Output.xlsx")

Before any Python code is used, the raw Survey Monkey output file (`"Edited Data - Survey Monkey Output.xlsx"`) is manually cleaned and prepared *directly within Excel*.  This involves the following steps:

1.  **Creating a Copy:** A copy of the original "Raw Data" sheet is created and renamed to "Edited_data". This preserves the original, untouched data.

2.  **Removing Unnecessary Columns:** Several columns that are not needed for the intended analysis are deleted.  These typically include columns containing personal information (PII) like names, email addresses, and potentially other irrelevant data fields. *Based on the Jupyter Notebook code, we can infer that the following columns were likely removed in this Excel step*:
    *   `Start Date`
    *   `End Date`
    *   `Email Address`
    *   `First Name`
    *   `Last Name`
    *   `Custom Data 1`

3.  **Creating a "Questions" Sheet:**
    *   The long question strings in the header row of the "Edited_data" sheet are extracted.
    *   A new sheet named "Question" is created.
    *   These extracted questions are pasted into the "Question" sheet.

4.  **Splitting Combined Questions:** In the original Survey Monkey export, some columns contain multiple questions combined (e.g., a main question and several sub-questions).  Within the **"Question" sheet** in Excel:
    *   The combined question strings are split into separate columns: one for the main question ("Raw Question") and subsequent columns for each sub-question ("Raw Subquestion 1", "Raw Subquestion 2", etc.). This is done using Excel's "Text to Columns" feature, likely using a delimiter (such as a hyphen or a specific phrase) to separate the main question from the sub-questions.
    * The extra columns are then deleted.

5.  **Shortening Question Text (in "Question" sheet):**
    *   In the "Question" sheet, concise, descriptive names are created to represent each question and sub-question. These shortened names will be used as column headers later in the Python analysis. For example, "Question 1-Response" might become "Question 1". This is done to make the column names more manageable.
    *   The shortened question texts are created in new columns (e.g., "Question", "Subquestion").

6.  **Copying Shortened Questions to "Edited_data" Sheet:** The shortened question names from the "Question" sheet are copied and *transposed* (pasted as columns) into the header row of the "Edited_data" sheet, replacing the original long question strings.

### Stage 2: Jupyter Notebook Processing (`data_manipulation.ipynb`)

The `data_manipulation.ipynb` notebook performs the following steps, building upon the pre-processed data from Stage 1:

1.  **Import Libraries:**
    *   Imports `pandas` for data manipulation.
    *   Imports `matplotlib.pyplot` for potential visualization (although visualization isn't heavily used in the provided code).
    *   Imports `os` (although its use isn't apparent in the provided code).
    *   Imports `numpy` for numerical operations.

2.  **Load Data:**
    *   Reads data from the *modified* Excel file named `"Edited Data - Survey Monkey Output.xlsx"`.
    *   Specifically loads the `"Edited_data"` sheet into a pandas DataFrame named `df`.

3.  **Initial Data Exploration:**
    *   Displays the first 10 rows of the DataFrame using `df.head(10)`.
    *   Prints information about the DataFrame, including column names, data types, and non-null counts, using `df.info()`.

4.  **Data Cleaning and Preparation (Python-based):**
    *   Creates a copy of the DataFrame named `df_modified` to preserve the original data.
    *   *Although some columns were already removed in Excel, this step might be redundant, but it's included for completeness.* Drops several columns deemed unnecessary for the analysis: `'Start Date'`, `'End Date'`, `'Email Address'`, `'First Name'`, `'Last Name'`, and `'Custom Data 1'`. The `inplace=True` argument modifies `df_modified` directly.
    *   Prints the `df_modified.info()` again to show the reduced number of columns.

5.  **Data Reshaping (Melting):**
    *   Prepares the DataFrame for reshaping using the `melt()` function. This transforms the data from a "wide" format to a "long" format.
    *   Defines a list `id_vars` containing the columns to be kept as identifiers: the first 8 columns of `df_modified`.
    *   Applies the `melt()` function:
        *   `id_vars`: Specifies the identifier columns.
        *   `var_name`: Sets the name of the new column that will hold the original column names (now unpivoted) to `'Question + SubQuestion'`.
        *   `value_name`: Sets the name of the new column that will hold the values from the unpivoted columns to `'Answer'`.
    *   The result is stored in a new DataFrame called `df_melted`.

6.  **Merging with Question Data:**
    *   Loads the `"Question"` sheet (created during the Excel pre-processing) from the same Excel file (`"Edited Data - Survey Monkey Output.xlsx"`) into a DataFrame called `questions_import`.
    *   Displays the `questions_import` DataFrame.
    *   Creates a copy of `questions_import` named `questions`.
    *   Drops a column named "Unnamed: 5" from the questions.
    *   Splits the column "Question + Subquestion" into "Question" and "Subquestion".
    *   Merges `questions` with `df_melted` using a left merge:
        *   `left_on`: Specifies the joining columns in `df_melted` (`"Question + SubQuestion"`).
        *   `right_on`: Specifies the joining columns in `questions` (`"Question + SubQuestion"`).
        *   `how='left'`: Keeps all rows from `df_melted` and adds matching data from `questions`.
    *   The merged DataFrame is stored in `dataset_merged`.
    *   Prints the number of respondents for each question.

7.  **Calculating and Merging "Same Answer" Counts:**
    *   Calculates the number of times each unique answer was given for each question and subquestion combination.
        *   Uses `groupby()` on `'Question + SubQuestion'` and `'Answer'` to group the data.
        *   Uses `['Respondent ID'].nunique()` to count the distinct number of respondents within each group. This avoids double-counting respondents who might have answered multiple subquestions within the same question.
        *   The result is a Series named `same_answer`.
    *   Converts the `same_answer` Series to a DataFrame using `to_frame()`.
    *   Renames the `'Respondent ID'` column (which now holds the counts) to `'same_answer'`.
    *   Merges `dataset_merged` with the `same_answer` DataFrame:
        *   `left_on`: `['Question + SubQuestion', 'Answer']`
        *   `right_on`: `['Question + SubQuestion', 'Answer']`
        *   `how='left'`
    *   The result is stored in `dataset_merged_three`.

8.  **Handling Missing Values:**
    *   Fills any missing values (`NaN`) in the `'same_answer'` column with 0, using `fillna(0, inplace=True)`. This indicates that no respondents gave that particular answer to that question/subquestion combination.

9.  **Renaming the remaning columns**
    *   Renames the columns for better readability.

10. **Final Output:**
    *   Creates a copy of final output.
    *   Saves the final processed DataFrame (`output`) to a new Excel file named `"final_output_survey_monkey.xlsx"`. The `index=False` argument prevents the DataFrame index from being written to the file.


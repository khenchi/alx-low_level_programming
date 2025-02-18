# alx-low_level_programming



Low_level_programming




import pytest
import pandas as pd
from your_module import add_periodic_sums  # Replace with your actual module name

@pytest.fixture
def sample_dataframe():
    """Creates a sample DataFrame with a range of dates and values."""
    data = {
        "date": pd.date_range(start="2024-01-01", periods=200, freq='D'),
        "value": [i for i in range(200)]
    }
    return pd.DataFrame(data)

@pytest.fixture
def empty_dataframe():
    """Creates an empty DataFrame."""
    return pd.DataFrame(columns=["date", "value"])

@pytest.fixture
def dataframe_with_nans():
    """Creates a DataFrame with NaN values in the value column."""
    data = {
        "date": pd.date_range(start="2024-01-01", periods=10, freq='D'),
        "value": [1, 2, None, 4, 5, None, 7, 8, 9, None]  # Some NaNs
    }
    return pd.DataFrame(data)

def test_past_20_days_sum(sample_dataframe):
    """Test if the sum over the past 20 days is correct."""
    reference_date = "2024-06-15"
    df_result = add_periodic_sums(sample_dataframe, "date", "value", reference_date)

    reference_date = pd.to_datetime(reference_date)
    expected_sum = sample_dataframe[
        (sample_dataframe["date"] >= reference_date - pd.Timedelta(days=20)) &
        (sample_dataframe["date"] <= reference_date)
    ]["value"].sum()

    assert df_result.iloc[-2]["value"] == expected_sum

def test_last_semester_sum(sample_dataframe):
    """Test if the sum over the last fixed semester is correct."""
    reference_date = "2024-06-15"
    df_result = add_periodic_sums(sample_dataframe, "date", "value", reference_date)

    year = pd.to_datetime(reference_date).year
    semester_start = pd.Timestamp(year, 1, 2)
    semester_end = pd.Timestamp(year, 6, 30)

    expected_sum = sample_dataframe[
        (sample_dataframe["date"] >= semester_start) &
        (sample_dataframe["date"] <= semester_end)
    ]["value"].sum()

    assert df_result.iloc[-1]["value"] == expected_sum

@pytest.mark.parametrize("reference_date", ["2024-01-02", "2024-07-01", "2024-12-31"])
def test_edge_case_dates(sample_dataframe, reference_date):
    """Test the logic when the reference date is at the semester boundary."""
    df_result = add_periodic_sums(sample_dataframe, "date", "value", reference_date)
    
    reference_date = pd.to_datetime(reference_date)
    
    # Compute expected past 20 days sum
    expected_20_day_sum = sample_dataframe[
        (sample_dataframe["date"] >= reference_date - pd.Timedelta(days=20)) &
        (sample_dataframe["date"] <= reference_date)
    ]["value"].sum()

    # Determine the semester range
    year = reference_date.year
    if reference_date.month >= 7:
        semester_start = pd.Timestamp(year, 7, 1)
        semester_end = pd.Timestamp(year, 12, 31)
    else:
        semester_start = pd.Timestamp(year, 1, 2)
        semester_end = pd.Timestamp(year, 6, 30)

    expected_semester_sum = sample_dataframe[
        (sample_dataframe["date"] >= semester_start) &
        (sample_dataframe["date"] <= semester_end)
    ]["value"].sum()

    assert df_result.iloc[-2]["value"] == expected_20_day_sum
    assert df_result.iloc[-1]["value"] == expected_semester_sum

def test_empty_dataframe(empty_dataframe):
    """Test behavior when an empty DataFrame is provided."""
    reference_date = "2024-12-15"
    df_result = add_periodic_sums(empty_dataframe, "date", "value", reference_date)

    assert len(df_result) == 2  # Only the two new rows should exist
    assert df_result.iloc[-2]["value"] == 0
    assert df_result.iloc[-1]["value"] == 0

def test_dataframe_with_nans(dataframe_with_nans):
    """Test handling of NaN values in the 'value' column."""
    reference_date = "2024-01-10"
    df_result = add_periodic_sums(dataframe_with_nans, "date", "value", reference_date)

    reference_date = pd.to_datetime(reference_date)
    
    # Compute expected sums ignoring NaNs
    expected_20_day_sum = dataframe_with_nans[
        (dataframe_with_nans["date"] >= reference_date - pd.Timedelta(days=20)) &
        (dataframe_with_nans["date"] <= reference_date)
    ]["value"].sum(skipna=True)

    expected_semester_sum = dataframe_with_nans[
        (dataframe_with_nans["date"] >= pd.Timestamp(2024, 1, 2)) &
        (dataframe_with_nans["date"] <= pd.Timestamp(2024, 6, 30))
    ]["value"].sum(skipna=True)

    assert df_result.iloc[-2]["value"] == expected_20_day_sum
    assert df_result.iloc[-1]["value"] == expected_semester_sum





import pytest
import pandas as pd
from your_module import add_periodic_sums  # Replace with your actual module name

@pytest.fixture
def sample_dataframe():
    """Creates a sample DataFrame for testing."""
    data = {
        "date": pd.date_range(start="2024-06-01", periods=100, freq='D'),
        "value": list(range(100))
    }
    return pd.DataFrame(data)

def test_past_20_days_sum(sample_dataframe):
    """Test if the sum over the past 20 days is correct."""
    reference_date = "2024-12-15"
    df_result = add_periodic_sums(sample_dataframe, "date", "value", reference_date)

    reference_date = pd.to_datetime(reference_date)
    expected_sum = sample_dataframe[
        (sample_dataframe["date"] >= reference_date - pd.Timedelta(days=20)) &
        (sample_dataframe["date"] <= reference_date)
    ]["value"].sum()

    assert df_result.iloc[-2]["value"] == expected_sum

def test_last_semester_sum(sample_dataframe):
    """Test if the sum over the last fixed semester is correct."""
    reference_date = "2024-12-15"
    df_result = add_periodic_sums(sample_dataframe, "date", "value", reference_date)

    year = pd.to_datetime(reference_date).year
    semester_start = pd.Timestamp(year, 7, 1)
    semester_end = pd.Timestamp(year, 12, 31)

    expected_sum = sample_dataframe[
        (sample_dataframe["date"] >= semester_start) &
        (sample_dataframe["date"] <= semester_end)
    ]["value"].sum()

    assert df_result.iloc[-1]["value"] == expected_sum

def test_dataframe_shape(sample_dataframe):
    """Test if two new rows are correctly added."""
    reference_date = "2024-12-15"
    df_result = add_periodic_sums(sample_dataframe, "date", "value", reference_date)

    assert len(df_result) == len(sample_dataframe) + 2

def test_labels_of_new_rows(sample_dataframe):
    """Test if the labels 'Past 20 Days' and 'Last Semester' are added correctly."""
    reference_date = "2024-12-15"
    df_result = add_periodic_sums(sample_dataframe, "date", "value", reference_date)

    assert df_result.iloc[-2]["date"] == "Past 20 Days"
    assert df_result.iloc[-1]["date"] == "Last Semester"

def test_empty_dataframe():
    """Test behavior when an empty DataFrame is provided."""
    df_empty = pd.DataFrame(columns=["date", "value"])
    reference_date = "2024-12-15"
    
    df_result = add_periodic_sums(df_empty, "date", "value", reference_date)

    assert len(df_result) == 2  # Only the two new rows should exist
    assert df_result.iloc[-2]["value"] == 0
    assert df_result.iloc[-1]["value"] == 0

def test_nonexistent_date_column():
    """Test behavior when the date column does not exist."""
    df_invalid = pd.DataFrame({"wrong_date": ["2024-06-01"], "value": [10]})
    reference_date = "2024-12-15"

    with pytest.raises(KeyError):
        add_periodic_sums(df_invalid, "date", "value", reference_date)

def test_nonexistent_value_column():
    """Test behavior when the value column does not exist."""
    df_invalid = pd.DataFrame({"date": ["2024-06-01"], "wrong_value": [10]})
    reference_date = "2024-12-15"

    with pytest.raises(KeyError):
        add_periodic_sums(df_invalid, "date", "value", reference_date)

def test_invalid_date_format():
    """Test behavior when the date column has invalid format."""
    df_invalid = pd.DataFrame({"date": ["invalid_date"], "value": [10]})
    reference_date = "2024-12-15"

    with pytest.raises(ValueError):
        add_periodic_sums(df_invalid, "date", "value", reference_date)





import pandas as pd

def add_periodic_sums(df, date_column, value_column, reference_date):
    """
    Adds two new rows to the DataFrame:
    - Sum over the past 20 days.
    - Sum over the last fixed semester.

    Args:
        df (pd.DataFrame): The input DataFrame.
        date_column (str): Name of the column containing dates.
        value_column (str): Name of the column containing numeric values.
        reference_date (str or pd.Timestamp): The reference date for calculations.

    Returns:
        pd.DataFrame: The DataFrame with the two new rows added.
    """
    df = df.copy()
    df[date_column] = pd.to_datetime(df[date_column])
    reference_date = pd.to_datetime(reference_date)

    def calculate_past_20_days_sum(df, reference_date):
        """Calculates the sum of the last 20 days from the reference date."""
        start_date = reference_date - pd.Timedelta(days=20)
        return df.loc[(df[date_column] >= start_date) & (df[date_column] <= reference_date), value_column].sum(skipna=True)

    def get_semester_boundaries(reference_date):
        """Returns the start and end dates of the semester based on a fixed calendar."""
        year = reference_date.year
        if reference_date.month >= 7:
            return pd.Timestamp(year, 7, 1), pd.Timestamp(year, 12, 31)
        else:
            return pd.Timestamp(year, 1, 2), pd.Timestamp(year, 6, 30)

    def calculate_semester_sum(df, semester_start, semester_end):
        """Calculates the sum of values within the last semester."""
        return df.loc[(df[date_column] >= semester_start) & (df[date_column] <= semester_end), value_column].sum(skipna=True)

    # Compute the past 20-day sum
    past_20_days_sum = calculate_past_20_days_sum(df, reference_date)

    # Compute the last semester sum
    semester_start, semester_end = get_semester_boundaries(reference_date)
    last_semester_sum = calculate_semester_sum(df, semester_start, semester_end)

    # Append new rows
    new_rows = pd.DataFrame([
        {date_column: "Past 20 Days", value_column: past_20_days_sum},
        {date_column: "Last Semester", value_column: last_semester_sum}
    ])

    return pd.concat([df, new_rows], ignore_index=True)








You can use the openpyxl library to copy a sheet from one Excel file to another while preserving formatting. Here’s a function to achieve this:

Steps:

1. Load the source workbook and the destination workbook.


2. Copy the sheet structure, including values, styles, and dimensions.


3. Save the destination workbook.



Function:

import openpyxl
from openpyxl.utils import get_column_letter
from openpyxl.styles import NamedStyle

def copy_excel_sheet(source_file, source_sheet_name, destination_file, destination_sheet_name):
    # Load the source and destination workbooks
    src_wb = openpyxl.load_workbook(source_file, data_only=False)
    dest_wb = openpyxl.load_workbook(destination_file)

    # Get the source sheet
    if source_sheet_name not in src_wb.sheetnames:
        raise ValueError(f"Sheet '{source_sheet_name}' not found in {source_file}")
    src_sheet = src_wb[source_sheet_name]

    # Create a new sheet in the destination workbook
    if destination_sheet_name in dest_wb.sheetnames:
        dest_wb.remove(dest_wb[destination_sheet_name])  # Remove if already exists
    dest_sheet = dest_wb.create_sheet(destination_sheet_name)

    # Copy cell values, styles, and dimensions
    for row in src_sheet.iter_rows():
        for cell in row:
            new_cell = dest_sheet.cell(row=cell.row, column=cell.column, value=cell.value)
            
            # Copy cell style
            if cell.has_style:
                new_cell.font = cell.font
                new_cell.border = cell.border
                new_cell.fill = cell.fill
                new_cell.number_format = cell.number_format
                new_cell.protection = cell.protection
                new_cell.alignment = cell.alignment

    # Copy column widths
    for col in src_sheet.column_dimensions:
        dest_sheet.column_dimensions[col].width = src_sheet.column_dimensions[col].width

    # Copy row heights
    for row_idx in src_sheet.row_dimensions:
        dest_sheet.row_dimensions[row_idx].height = src_sheet.row_dimensions[row_idx].height

    # Save the destination workbook
    dest_wb.save(destination_file)
    src_wb.close()
    dest_wb.close()
    print(f"Sheet '{source_sheet_name}' copied to '{destination_sheet_name}' in {destination_file}")

# Example usage:
# copy_excel_sheet("source.xlsx", "Sheet1", "destination.xlsx", "CopiedSheet")

Features:

Copies cell values and styles (font, border, fill, alignment, etc.).

Preserves column widths and row heights.

Handles cases where the destination sheet already exists.


Let me know if you need any modifications!




You can create a function to ensure that each column in your DataFrame adheres to a predefined schema. This function will:

1. Check if all values in a column match the expected type.


2. Convert or fix values where possible.


3. Optionally log or raise errors for incompatible types.



Here's a robust function to achieve this:

import pandas as pd

def enforce_schema(df: pd.DataFrame, schema: dict):
    """
    Ensures that the DataFrame columns adhere to the specified schema.
    
    Parameters:
    df (pd.DataFrame): The input DataFrame.
    schema (dict): A dictionary where keys are column names and values are the expected data types.
    
    Returns:
    pd.DataFrame: A DataFrame with corrected data types.
    """
    df_fixed = df.copy()
    
    for column, expected_type in schema.items():
        if column in df_fixed.columns:
            # Fix types based on the schema
            if expected_type == int:
                df_fixed[column] = pd.to_numeric(df_fixed[column], errors='coerce').fillna(0).astype(int)
            elif expected_type == float:
                df_fixed[column] = pd.to_numeric(df_fixed[column], errors='coerce').fillna(0.0).astype(float)
            elif expected_type == str:
                df_fixed[column] = df_fixed[column].astype(str)
            elif expected_type == bool:
                df_fixed[column] = df_fixed[column].astype(bool)
            else:
                raise ValueError(f"Unsupported type {expected_type} for column {column}")

    return df_fixed

# Example usage
data = {'column1': [1, '2', 3, 'a'], 'column2': [1.1, '2.2', 3.3, 'b'], 'column3': [True, 'False', 'yes', 0]}
df = pd.DataFrame(data)

# Define expected schema
schema = {'column1': int, 'column2': float, 'column3': bool}

# Enforce schema
df_cleaned = enforce_schema(df, schema)

# Display the cleaned DataFrame
import ace_tools as tools
tools.display_dataframe_to_user(name="Cleaned DataFrame", dataframe=df_cleaned)

Explanation:

The function takes a schema (dictionary) that defines the expected types for each column.

It checks each column and converts values to the expected type.

If conversion fails (e.g., a string where an integer is expected), it replaces invalid values with defaults (0 for int, 0.0 for float, False for bool).

Raises an error if an unsupported type is encountered.


This ensures type homogeneity and prevents type errors when processing data. Let me know if you need modifications!



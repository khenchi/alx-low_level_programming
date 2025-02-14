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

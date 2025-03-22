# data-sanitizer

Creating a Python library for data sanitization that detects and masks sensitive data can be quite involved. Below is a simplified version that masks sensitive information such as names, emails, and phone numbers from a dataset. This example uses regular expressions for detection and assumes data is in a CSV format. This is a foundation upon which more complex behaviors can be built.

```python
import re
import pandas as pd
from typing import List, Pattern, Tuple

class DataSanitizer:
    def __init__(self):
        # Patterns for detecting sensitive data
        self.patterns: List[Tuple[Pattern, str]] = [
            (re.compile(r'\b[A-Z][a-z]+\s[A-Z][a-z]+\b'), 'FULL_NAME'),   # Simple name pattern
            (re.compile(r'[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+'), 'EMAIL'),
            (re.compile(r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b'), 'PHONE_NUMBER') # US phone number pattern
        ]

    def sanitize(self, text: str) -> str:
        """
        Sanitize a given text by replacing detected patterns with mask labels.

        Parameters:
        - text (str): The text to sanitize.

        Returns:
        - str: The sanitized text.
        """
        if not text or not isinstance(text, str):
            return text

        for pattern, mask_label in self.patterns:
            text = pattern.sub(mask_label, text)
        
        return text
    
    def sanitize_dataframe(self, df: pd.DataFrame) -> pd.DataFrame:
        """
        Sanitize the sensitive data in a pandas DataFrame.

        Parameters:
        - df (pd.DataFrame): The DataFrame to sanitize.

        Returns:
        - pd.DataFrame: A sanitized DataFrame.
        """
        sanitized_df = df.copy()
        for column in df.columns:
            if df[column].dtype == 'object':  # Only process columns with string data
                sanitized_df[column] = df[column].apply(self.sanitize)
        
        return sanitized_df

def load_csv(file_path: str) -> pd.DataFrame:
    """
    Load a CSV file into a pandas DataFrame with error handling.

    Parameters:
    - file_path (str): Path to the CSV file.

    Returns:
    - pd.DataFrame: Loaded DataFrame.
    """
    try:
        df = pd.read_csv(file_path)
        print(f"Loaded {len(df)} records from {file_path}.")
        return df
    except FileNotFoundError:
        print(f"File not found: {file_path}")
        return pd.DataFrame()
    except pd.errors.EmptyDataError:
        print(f"No data found in: {file_path}")
        return pd.DataFrame()
    except pd.errors.ParserError as e:
        print(f"Error parsing the file {file_path}: {e}")
        return pd.DataFrame()
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return pd.DataFrame()

def save_to_csv(df: pd.DataFrame, file_path: str):
    """
    Save a pandas DataFrame to a CSV file with error handling.

    Parameters:
    - df (pd.DataFrame): The DataFrame to save.
    - file_path (str): Destination file path.
    """
    try:
        df.to_csv(file_path, index=False)
        print(f"Sanitized data saved to {file_path}.")
    except Exception as e:
        print(f"Failed to save data: {e}")

def main(input_file: str, output_file: str):
    df = load_csv(input_file)
    if df.empty:
        print("No data to sanitize.")
        return
    
    sanitizer = DataSanitizer()
    sanitized_df = sanitizer.sanitize_dataframe(df)
    save_to_csv(sanitized_df, output_file)

if __name__ == "__main__":
    # Replace 'input.csv' and 'output.csv' with your file paths
    main('input.csv', 'output.csv')
```

### Key Components:
- **DataSanitizer Class**: Contains regex patterns for detecting and masking sensitive data.
- **sanitize Method**: Masks data according to patterns.
- **sanitize_dataframe Method**: Applies sanitization to a DataFrame.
- **load_csv and save_to_csv Functions**: Handle loading and saving CSV files with error handling.
- **main Function**: Coordinates reading, sanitizing, and writing data.

### Error Handling:
- The program includes try-except blocks to capture and report issues like missing files, parse errors, and I/O issues. This is crucial for robust operation in real-world applications.
# dataannotation
Data Annotation sample program


import requests
import pandas as pd

def fetch_google_doc_data(url):
    # Fetch the data from the Google Doc
    response = requests.get(url)
    response.raise_for_status()
    return response.text

def parse_table(data):
    # Parse the table data using pandas
    # Attempt to read the first table, if fails, return empty DataFrame
    
    try:
        df = pd.read_html(data)[0]
        #print (df)
    except (IndexError, ValueError):
        # If no tables are found or if table parsing fails, create an empty DataFrame
        # with the expected columns
        df = pd.DataFrame(columns=['x-coordinate', 'y-coordinate', 'Character'])
        print("Warning: No table found or table parsing failed. Returning an empty grid.")

    return df

def create_grid(df):
    # Determine the dimensions of the grid
    # Handle cases where columns may be missing
    
    df.rename(columns={df.columns[0]: 'x-coordinate', 
                       df.columns[1]: 'Character', 
                       df.columns[2]: 'y-coordinate'}, inplace=True)
    
    df['x-coordinate'] = pd.to_numeric(df['x-coordinate'], errors='coerce') # Added errors='coerce'
    df['y-coordinate'] = pd.to_numeric(df['y-coordinate'], errors='coerce') # Added errors='coerce'
    
    # Replace NaN values with 0 in coordinate columns
    df['x-coordinate'] = df['x-coordinate'].fillna(0).astype(int) 
    df['y-coordinate'] = df['y-coordinate'].fillna(0).astype(int)
    
    max_x = df['x-coordinate'].max()
    max_y = df['y-coordinate'].max()

    print (max_x, max_y)

    # Convert max_x and max_y to integers if they are strings
    max_x = int(max_x)  
    max_y = int(max_y)  
    
    # Create an empty grid filled with spaces
    grid = [[' ' for _ in range(max_x + 1)] for _ in range(max_y + 1)]
    
    # Fill the grid with characters from the dataframe
    for _, row in df.iterrows():
        x = row['x-coordinate']
        y = row['y-coordinate']
        char = row['Character']
        grid[int(y)][int(x)] = char
    
    return grid

def print_grid(grid):
    # Print the grid
    for row in grid:
        print(''.join(row))

def main(url):
    data = fetch_google_doc_data(url)
    df = parse_table(data)
    grid = create_grid(df)
    print_grid(grid)


# Example usage
url = 'https://docs.google.com/document/d/e/2PACX-1vQGUck9HIFCyezsrBSnmENk5ieJuYwpt7YHYEzeNJkIb9OSDdx-ov2nRNReKQyey-cwJOoEKUhLmN9z/pub'
main(url)
